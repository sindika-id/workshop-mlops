# Cat–Dog ONNX Conversion: Step-by-Step Guide
This guide walks you through converting your **trained PyTorch model** into the **ONNX format**, validating it with **ONNX Runtime**, and preparing it for deployment (e.g., TensorRT, OpenVINO).

---

## 1) Prepare Your Environment

Add ONNX packages to your `requirements.txt`:
```txt
torch
torchvision
onnx
onnxruntime
onnxsim
```

Install:
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

## 2) Load Your Trained Model

Create **`export_onnx.py`** in the project root:

```python
import torch
from torch import nn
from torchvision import models

# 1. Define the model architecture (must match training)
model = models.resnet18(weights=None)
model.fc = nn.Linear(model.fc.in_features, 2)

# 2. Load trained weights
ckpt = torch.load("catdog_model.pth", map_location="cpu")
model.load_state_dict(ckpt)
model.eval()

print("Model loaded and ready for ONNX export.")
```

Run:
```bash
python export_onnx.py
```

---

## 3) Export to ONNX

Extend **`export_onnx.py`**:

```python
# Add after model.eval()
dummy = torch.randn(1, 3, 224, 224)

torch.onnx.export(
    model,
    dummy,
    "catdog_resnet18.onnx",
    input_names=["input"],
    output_names=["logits"],
    dynamic_axes={
        "input": {0: "batch"},
        "logits": {0: "batch"},
    },
    opset_version=17,
    do_constant_folding=True
)

print("Exported to catdog_resnet18.onnx")
```

Run:
```bash
python export_onnx.py
```

✅ This generates `catdog_resnet18.onnx`.

---

## 4) Simplify the ONNX Graph

ONNX files often contain redundant ops. Use `onnxsim`:

```bash
python -m onnxsim models/catdog_resnet18.onnx models/catdog_resnet18_simplified.onnx
```

This makes inference faster and more portable.

if failed you can use docker :

```bash
docker pull mirror.gcr.io/library/python:3.12-slim

docker run --rm -it -v "$PWD":/w -w /w mirror.gcr.io/library/python:3.12-slim bash -lc '
set -euo pipefail
apt-get update && apt-get install -y --no-install-recommends \
  cmake build-essential protobuf-compiler libprotobuf-dev git && \
  rm -rf /var/lib/apt/lists/*

python -m pip install --upgrade pip
# pin numpy<2 to avoid ABI issues with some ORT wheels
pip install --no-cache-dir "numpy<2,>=1.24.4" onnx==1.16.1 onnxruntime==1.18.0
pip install --no-cache-dir onnxsim==0.4.36

python -m onnxsim models/catdog_resnet18.onnx models/catdog_resnet18_simplified.onnx

python - <<PY
import onnx
onnx.checker.check_model("models/catdog_resnet18_simplified.onnx")
print("OK")
PY
'
```

---

## 5) Validate with ONNX Runtime

Add to **`validate_onnx.py`**:

```python
import torch
import onnxruntime as ort
import numpy as np
from torch import nn
from torchvision import models

# Load PyTorch model
model = models.resnet18(weights=None)
model.fc = nn.Linear(model.fc.in_features, 2)
model.load_state_dict(torch.load("catdog_model.pth", map_location="cpu"))
model.eval()

# Dummy input
dummy = torch.randn(1, 3, 224, 224)

# PyTorch output
with torch.no_grad():
    pt_out = model(dummy).numpy()

# ONNX Runtime output
session = ort.InferenceSession("catdog_resnet18.onnx", providers=["CPUExecutionProvider"])
onnx_out = session.run(None, {"input": dummy.numpy()})[0]

# Compare
diff = np.max(np.abs(pt_out - onnx_out))
print("Max abs diff:", diff)
```

Run:
```bash
python validate_onnx.py
```

Expected:
```
Max abs diff: ~1e-5
```

---

## 6) Project Structure After Export

```plaintext
cat-dog/
├── data/
├── models/
│   ├── catdog_model.pth
│   ├── catdog_resnet18.onnx
│   └── catdog_resnet18_simplified.onnx
├── src/
│   └── __init__.py
├── export_onnx.py
├── validate_onnx.py
├── requirements.txt
└── README.md
```

---

## 7) Benchmark Speed

bench_speed.py

```py
import time, statistics, numpy as np
import torch
from torch import nn
from torchvision import models
import onnxruntime as ort

# -------------------------
# Config
# -------------------------
ONNX_PATH = "models/catdog_resnet18.onnx"
BATCH_SIZES = [1, 8, 32]      # change as needed
WARMUP = 10
RUNS = 100                    # increase for more stable stats
NUM_THREADS = 4               # tune for your CPU; try 1, 4, or None

# Optional: reduce inter-run noise by fixing threads
if NUM_THREADS is not None:
    torch.set_num_threads(NUM_THREADS)
    torch.set_num_interop_threads(max(1, NUM_THREADS // 2))

# -------------------------
# Load PyTorch model (CPU)
# -------------------------
pt_model = models.resnet18(weights=None)
pt_model.fc = nn.Linear(pt_model.fc.in_features, 2)
pt_model.load_state_dict(torch.load("models/catdog_model.pth", map_location="cpu"))
pt_model.eval()

# -------------------------
# Prepare ONNX Runtime session
# -------------------------
so = ort.SessionOptions()
so.graph_optimization_level = ort.GraphOptimizationLevel.ORT_ENABLE_ALL
# so.intra_op_num_threads = NUM_THREADS or leave default
# so.inter_op_num_threads = max(1, (NUM_THREADS or 1)//2)
sess = ort.InferenceSession(ONNX_PATH, sess_options=so, providers=["CPUExecutionProvider"])

def bench_pytorch(batch, h=224, w=224):
    x = torch.randn(batch, 3, h, w, dtype=torch.float32)
    # warmup
    with torch.no_grad():
        for _ in range(WARMUP):
            _ = pt_model(x)
    # measure
    times = []
    with torch.no_grad():
        for _ in range(RUNS):
            t0 = time.perf_counter()
            _ = pt_model(x)
            t1 = time.perf_counter()
            times.append((t1 - t0) * 1000.0)  # ms
    return np.array(times)

def bench_onnx(batch, h=224, w=224):
    x = np.random.randn(batch, 3, h, w).astype(np.float32)
    # warmup
    for _ in range(WARMUP):
        _ = sess.run(None, {"input": x})
    # measure
    times = []
    for _ in range(RUNS):
        t0 = time.perf_counter()
        _ = sess.run(None, {"input": x})
        t1 = time.perf_counter()
        times.append((t1 - t0) * 1000.0)  # ms
    return np.array(times)

def summarize(times_ms, batch):
    mean = float(times_ms.mean())
    p50  = float(np.percentile(times_ms, 50))
    p90  = float(np.percentile(times_ms, 90))
    p99  = float(np.percentile(times_ms, 99))
    thr  = (batch / (mean / 1000.0))  # imgs/s using mean latency
    return mean, p50, p90, p99, thr

print(f"Threads: torch={torch.get_num_threads()} interop={torch.get_num_interop_threads()}")
print(f"ONNX Runtime EPs: {sess.get_providers()}")

for b in BATCH_SIZES:
    pt_times = bench_pytorch(b)
    onnx_times = bench_onnx(b)

    pt_stats = summarize(pt_times, b)
    onnx_stats = summarize(onnx_times, b)

    print(f"\nBatch {b}")
    print("PyTorch  : mean={:.2f} ms  p50={:.2f}  p90={:.2f}  p99={:.2f}  thr={:.1f} img/s"
          .format(*pt_stats))
    print("ONNX RT  : mean={:.2f} ms  p50={:.2f}  p90={:.2f}  p99={:.2f}  thr={:.1f} img/s"
          .format(*onnx_stats))
```

Batch 1
PyTorch  : mean=15.84 ms  p50=15.72  p90=16.56  p99=18.10  thr=63.1 img/s
ONNX RT  : mean=14.89 ms  p50=13.83  p90=15.33  p99=39.90  thr=67.1 img/s

Batch 8
PyTorch  : mean=142.21 ms  p50=141.11  p90=149.39  p99=159.95  thr=56.3 img/s
ONNX RT  : mean=114.16 ms  p50=113.28  p90=121.63  p99=155.30  thr=70.1 img/s


⚡ You now have a **verified ONNX model** ready for production.
