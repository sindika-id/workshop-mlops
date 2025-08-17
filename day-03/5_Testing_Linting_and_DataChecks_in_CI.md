# 5. Testing, Linting, and Data Checks in CI

## 🎯 Learning Objectives
- Learn how to enforce code quality with **linting** tools.  
- Run **unit tests** for ML pipelines in CI.  
- Add **data validation checks** to prevent bad data from reaching training.  

---

## 📘 Why Add These Checks?

- **Linting** ensures consistent code style and avoids common errors.  
- **Unit tests** verify training and inference functions work as expected.  
- **Data validation** catches schema changes or corrupted data early.  

These checks prevent wasted compute time and improve reproducibility.  

---

## 🛠 Step 1: Linting with Flake8 & Black

Install tools:
```bash
pip install flake8 black
```

Run locally:
```bash
flake8 src/
black --check src/
```

Add to CI (GitHub Actions snippet):
```yaml
- name: Lint code
  run: |
    pip install flake8 black
    flake8 src/
    black --check src/
```

---

## 🛠 Step 2: Unit Tests with Pytest

Install:
```bash
pip install pytest
```

Example test `tests/test_train.py`:
```python
from train import build_model

def test_model_output_shape():
    model = build_model()
    x = model(torch.randn(1, 3, 224, 224))
    assert x.shape[1] == 2  # two classes: cat, dog
```

Run tests in CI:
```yaml
- name: Run unit tests
  run: |
    pip install pytest
    pytest tests/
```

---

## 🛠 Step 3: Data Validation with Great Expectations (Optional)

Install:
```bash
pip install great-expectations
```

Example validation script `validate_data.py`:
```python
import great_expectations as ge
import pandas as pd

df = pd.read_csv("data/train_labels.csv")
gdf = ge.from_pandas(df)

# Check columns exist
gdf.expect_column_to_exist("filename")
gdf.expect_column_values_to_not_be_null("label")
```

Run validation in CI:
```yaml
- name: Validate data
  run: |
    pip install great-expectations
    python validate_data.py
```

---

## 🛠 Step 4: Combine in CI/CD Workflow

Example `.github/workflows/ci.yml` snippet:

```yaml
jobs:
  quality:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: "3.10"

    - name: Install dependencies
      run: |
        pip install -r requirements.txt

    - name: Lint code
      run: |
        pip install flake8 black
        flake8 src/
        black --check src/

    - name: Run unit tests
      run: |
        pytest tests/

    - name: Validate data
      run: |
        python validate_data.py
```

---

## ✅ Summary
- **Linting** enforces consistent coding style.  
- **Unit tests** verify ML pipeline correctness.  
- **Data validation** ensures training data integrity.  
- These checks catch issues early, saving compute and debugging time.  
