# TensorRT Inference Guide for Cat–Dog Classifier (ONNX → TensorRT)

This tutorial shows how to convert your **ONNX model** to a **TensorRT engine**, validate correctness, and run high‑performance inference. It builds on your existing ONNX export (`models/catdog_resnet18.onnx`).

> **Target OS:** Linux (Ubuntu) with NVIDIA GPU. TensorRT is not supported on macOS; use **Docker (NGC)** or a Linux machine/VM with CUDA drivers installed.

---

## 0) Prerequisites

- NVIDIA GPU with recent driver (supports CUDA 11/12).
- CUDA Toolkit installed (matching the driver).
- TensorRT 8.6+ installed (via **NGC container** or **local install**).
- ONNX file generated earlier: `models/catdog_resnet18.onnx` (dynamic batch recommended).

**Recommended: NGC Docker container** (no local setup conflicts):
```bash
# Pull a recent PyTorch + CUDA + TensorRT container (example)
docker pull nvcr.io/nvidia/pytorch:24.10-py3

# Run with GPU access and mount your project
docker run --gpus all -it --rm   -v $(pwd):/workspace/cat-dog   -w /workspace/cat-dog   nvcr.io/nvidia/pytorch:24.10-py3   /bin/bash
```

Inside the container, TensorRT and `trtexec` are typically available. Verify:
```bash
trtexec --version
python -c "import tensorrt as trt; print('TensorRT:', trt.__version__)"
```

> If you install TensorRT locally instead of Docker, follow NVIDIA’s official instructions for your OS/CUDA version.

---

## 1) Build a TensorRT Engine from ONNX (`trtexec`)

`trtexec` is a CLI tool shipped with TensorRT. It parses ONNX and builds a `.engine` plan file.

### 1.1 FP32 (baseline)

```bash
trtexec   --onnx=models/catdog_resnet18.onnx   --saveEngine=models/catdog_resnet18_fp32.engine   --noDataTransfers   --workspace=2048
```

**Flags**
- `--workspace=2048` → builder memory workspace in MB (tune as needed).
- `--noDataTransfers` → skip host I/O during benchmarking (optional).

### 1.2 FP16 (common best trade‑off)

```bash
trtexec   --onnx=models/catdog_resnet18.onnx   --saveEngine=models/catdog_resnet18_fp16.engine   --fp16   --workspace=2048
```

> FP16 usually provides 1.5–2× throughput vs FP32 on Tensor Core GPUs with minimal accuracy drop.

### 1.3 Dynamic Shapes (if your ONNX has dynamic batch)

```bash
trtexec   --onnx=models/catdog_resnet18.onnx   --minShapes=input:1x3x224x224   --optShapes=input:8x3x224x224   --maxShapes=input:32x3x224x224   --saveEngine=models/catdog_resnet18_dynamic_fp16.engine   --fp16   --workspace=2048
```

> Choose `min/opt/max` values that reflect your production workload.

### 1.4 INT8 (advanced; requires calibration for best accuracy)

```bash
# If you already have a calibration cache (recommended for repeatable builds):
trtexec   --onnx=models/catdog_resnet18.onnx   --int8   --calib=calibration.cache   --saveEngine=models/catdog_resnet18_int8.engine   --workspace=4096
```

If you don’t have a cache, you must implement an INT8 calibrator or use `trtexec`’s calibration with a directory of representative images (see NVIDIA docs).

---

## 2) Validate the Engine

To ensure correctness, compare TensorRT outputs to PyTorch/ONNX outputs on a few samples.

Create **`validate_trt.py`**:

```python
from pathlib import Path
import numpy as np
import torch
from torch import nn
from torchvision import models, transforms
from PIL import Image

import tensorrt as trt
import pycuda.autoinit  # noqa: F401 (creates CUDA context)
import pycuda.driver as cuda

ROOT = Path(__file__).resolve().parent
MODEL_DIR = ROOT / "models"
ENGINE_PATH = MODEL_DIR / "catdog_resnet18_fp16.engine"  # adjust as needed

# ----- 1) Load PyTorch model (for reference) -----
pt_model = models.resnet18(weights=None)
pt_model.fc = nn.Linear(pt_model.fc.in_features, 2)
pt_model.load_state_dict(torch.load(MODEL_DIR / "catdog_model.pth", map_location="cpu"))
pt_model.eval()

# Preprocessing consistent with training
tf = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),
])

def preprocess(img_path):
    img = Image.open(img_path).convert("RGB")
    x = tf(img).unsqueeze(0).numpy().astype(np.float32)  # NCHW
    return x

# ----- 2) Load TensorRT engine -----
logger = trt.Logger(trt.Logger.WARNING)
with open(ENGINE_PATH, "rb") as f, trt.Runtime(logger) as runtime:
    engine = runtime.deserialize_cuda_engine(f.read())
context = engine.create_execution_context()

inp_name = engine.get_binding_name(0) if engine.binding_is_input(0) else engine.get_binding_name(1)
out_name = engine.get_binding_name(1) if not engine.binding_is_input(1) else engine.get_binding_name(0)

inp_idx = engine.get_binding_index(inp_name)
out_idx = engine.get_binding_index(out_name)

# For dynamic shapes, set binding shape
context.set_binding_shape(inp_idx, (1, 3, 224, 224))

# Allocate device buffers
x = preprocess("data/val/cat/your_image.jpg")  # replace with a real image
d_in = cuda.mem_alloc(x.nbytes)
out_shape = (1, 2)
out = np.empty(out_shape, dtype=np.float32)
d_out = cuda.mem_alloc(out.nbytes)

# Copy input
cuda.memcpy_htod(d_in, x)

bindings = [None] * engine.num_bindings
bindings[inp_idx] = int(d_in)
bindings[out_idx] = int(d_out)

# Run inference
context.execute_v2(bindings)

# Copy output back
cuda.memcpy_dtoh(out, d_out)

# Compare with PyTorch
with torch.no_grad():
    pt_out = pt_model(torch.from_numpy(x)).numpy()

print("TRT logits:", out)
print("PT  logits:", pt_out)
print("Max abs diff:", np.max(np.abs(out - pt_out)))
print("TRT pred:", out.argmax(1), "PT pred:", pt_out.argmax(1))
```

Run:
```bash
python validate_trt.py
```

Expected: small numerical differences (e.g., 1e‑4 ~ 1e‑3).

---

## 3) Minimal TensorRT Inference Script

Create **`infer_trt.py`** (no PyTorch dependency at runtime):

```python
from pathlib import Path
import numpy as np
from PIL import Image
from torchvision import transforms  # only for preprocessing

import tensorrt as trt
import pycuda.autoinit  # noqa: F401
import pycuda.driver as cuda

ROOT = Path(__file__).resolve().parent
MODEL_DIR = ROOT / "models"
ENGINE_PATH = MODEL_DIR / "catdog_resnet18_fp16.engine"  # choose your engine

tf = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),
])

def load_engine(path):
    logger = trt.Logger(trt.Logger.WARNING)
    with open(path, "rb") as f, trt.Runtime(logger) as runtime:
        return runtime.deserialize_cuda_engine(f.read())

def infer_image(engine, img_path):
    context = engine.create_execution_context()

    # Preprocess
    img = Image.open(img_path).convert("RGB")
    x = tf(img).unsqueeze(0).numpy().astype(np.float32)  # 1x3x224x224

    inp_idx = 0 if engine.binding_is_input(0) else 1
    out_idx = 1 - inp_idx

    # Dynamic shape
    context.set_binding_shape(inp_idx, x.shape)

    # Allocate
    d_in = cuda.mem_alloc(x.nbytes)
    out_shape = (x.shape[0], 2)  # logits for 2 classes
    out = np.empty(out_shape, dtype=np.float32)
    d_out = cuda.mem_alloc(out.nbytes)

    cuda.memcpy_htod(d_in, x)
    bindings = [None] * engine.num_bindings
    bindings[inp_idx] = int(d_in)
    bindings[out_idx] = int(d_out)

    context.execute_v2(bindings)
    cuda.memcpy_dtoh(out, d_out)

    return out

if __name__ == "__main__":
    engine = load_engine(ENGINE_PATH)
    logits = infer_image(engine, "data/val/dog/your_image.jpg")  # replace path
    pred = logits.argmax(1)[0]
    print("Logits:", logits)
    print("Prediction:", ["cat", "dog"][pred])
```

Run:
```bash
python infer_trt.py
```

---

## 4) Benchmarking & Profiling

`trtexec` can benchmark directly during build or from an existing engine.

### Build + Benchmark
```bash
trtexec   --onnx=models/catdog_resnet18.onnx   --fp16   --shapes=input:8x3x224x224   --avgRuns=10 --warmUp=200   --profilingVerbosity=detailed   --saveEngine=models/catdog_bench_fp16.engine
```

### Benchmark Existing Engine
```bash
trtexec   --loadEngine=models/catdog_resnet18_fp16.engine   --shapes=input:8x3x224x224   --avgRuns=10 --warmUp=200   --profilingVerbosity=detailed
```

Key metrics printed:
- Latency (ms), Throughput (images/s)
- Layer‑wise timings (with `--dumpProfile` or detailed verbosity)

---

## 5) INT8 Calibration (Overview)

For best INT8 accuracy, calibrate on a representative dataset (a few hundred images). High‑level steps:

1. Implement an **IInt8EntropyCalibrator2** (Python or C++), providing device buffers fed with preprocessed batches from your dataset.
2. Build with `--int8` and point to a cache path; the builder will invoke your calibrator to generate `calibration.cache`.
3. Rebuild future engines using the cache (no need to recalibrate unless the network or preprocessing changes).

> Calibration code is beyond this minimal guide; use NVIDIA’s samples `sampleINT8` or Python-based calibrator examples as a starting point.

---

## 6) Common Issues & Fixes

- **Unsupported ONNX ops**: Re‑export with a different opset (13/17/19), run `onnxsim`, or upgrade TensorRT.
- **Out‑of‑memory during build**: Reduce `--workspace`, batch size, or use FP16. Ensure swap is available.
- **Dynamic shape errors**: You must set `min/opt/max` at build time and call `context.set_binding_shape(...)` before `execute_v2`.
- **Mismatch vs PyTorch**: Ensure preprocessing (resize/normalize) matches your training/inference pipeline.
- **Engine not portable**: Engines are specific to GPU architecture + TensorRT version. Rebuild per target platform or export multiple engines.

---

## 7) Suggested Project Layout

```
cat-dog/
├── data/
├── models/
│   ├── catdog_model.pth
│   ├── catdog_resnet18.onnx
│   ├── catdog_resnet18_fp32.engine
│   ├── catdog_resnet18_fp16.engine
│   └── calibration.cache        # (if using INT8)
├── export_onnx.py
├── validate_onnx.py
├── validate_trt.py
├── infer_trt.py
├── requirements.txt
└── README.md
```

---

## 8) Minimal `requirements.txt`

If you work **inside NGC container**, most of these are preinstalled; include only what you need:

```txt
numpy
Pillow
torch
torchvision
tensorrt       # if available in your environment
pycuda
```

> If `pip install tensorrt` does not work on your host, use the NGC container or NVIDIA’s local wheel repo for your CUDA/TensorRT version.

---

### You’re done
You can now build TensorRT engines from your ONNX, validate them against PyTorch outputs, and run fast inference in Python. For C++ or DeepStream deployment, port the same steps using NVIDIA’s samples as references.
