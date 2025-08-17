# 3. MLflow UI and Model Registry

## 🎯 Learning Objectives
- Navigate the MLflow Tracking UI effectively.
- Understand the purpose of the MLflow Model Registry.
- Register and manage multiple model versions for deployment.

---

## 🖥 Step 1: Exploring the MLflow UI

Once you start MLflow (`mlflow ui` or via Docker/Docker Compose), open:

👉 **http://localhost:5000**

The UI provides:

- **Experiments**: A collection of runs (each run represents a training execution).  
- **Runs**: Detailed metrics, parameters, and artifacts for each training session.  
- **Comparison**: Select multiple runs to compare metrics visually.  
- **Artifacts**: Files produced by your runs (e.g., models, plots, logs).  

📌 Example: You can compare two runs with different learning rates and visualize how accuracy changes.

---

## 🛠 Step 2: Registering Models in MLflow

MLflow provides a **Model Registry** to manage multiple versions of trained models.

### Register a model (from Python)

```python
import mlflow

# Assume you already logged a model in previous steps
result = mlflow.register_model(
    "runs:/<RUN_ID>/model",  # path from a completed run
    "CatDogClassifier"       # name of the model in the registry
)
print("Registered model:", result.name, "version:", result.version)
```

Replace `<RUN_ID>` with the run ID from your MLflow experiment.

---

### Promote a model version

Models can move across **stages**:
- **None**: Default stage after registration.  
- **Staging**: For testing before production.  
- **Production**: Approved model used in production.  
- **Archived**: Old models no longer in use.  

```python
from mlflow.tracking import MlflowClient

client = MlflowClient()
client.transition_model_version_stage(
    name="CatDogClassifier",
    version=1,
    stage="Production"
)
```

---

## 🖥 Step 3: Managing Models in the UI

In the MLflow UI, go to the **Models** tab:

- Browse registered models.  
- View all versions of a given model.  
- Assign stages (Staging, Production, Archived).  
- Track metadata (author, date, performance metrics).  

This provides **centralized governance** for ML models.

---

## 🧩 Step 4: Using a Registered Model

Load the latest **production** model in Python:

```python
import mlflow.pyfunc

model = mlflow.pyfunc.load_model("models:/CatDogClassifier/Production")
prediction = model.predict(<your_input_data>)
print("Prediction:", prediction)
```

---

## ✅ Summary
- The MLflow UI helps visualize and compare experiments.  
- The Model Registry provides version control for models.  
- You can promote models through stages (Staging → Production).  
- Registered models can be loaded programmatically for deployment.  