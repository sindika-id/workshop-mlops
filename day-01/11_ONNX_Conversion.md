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
python -m onnxsim catdog_resnet18.onnx catdog_resnet18_simplified.onnx
```

This makes inference faster and more portable.

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

⚡ You now have a **verified ONNX model** ready for production.
