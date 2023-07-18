# Model automation - train and predict pipelines

## Introduction

текст интро

In this tutorial, you'll learn how to automatically train YOLOv8 model and infer it local images **from your python code**.

✅ [main.py](https://github.com/supervisely-ecosystem/model-automation-train-and-predict-pipeline/blob/master/src/main.py) is just 112 lines of code and includes:

- deploying and training YOLOv8 model for object detection tasks
- saving all artifacts including model weights (checkpoints), logs, charts, visualizations, etc.
- inference pre-trained model on local images and get predictions

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/model-automation-train-and-predict-pipeline) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/model-automation-train-and-predict-pipeline.git
cd model-automation-train-and-predict-pipeline
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.&#x20;

```bash
code -r .
```

**Step 4.** change ✅ workspace ID, team ID, and project ID ✅ in `local.env` file by copying the ID from the context menu of the workspace. A new project with annotated images will be created in the workspace you define. [Learn more here.](../../getting-started/environment-variables.md#team_id)

```python
TEAM_ID=435  # ⬅️ change value
WORKSPACE_ID=683  # ⬅️ change value
PROJECT_ID=24504  # ⬅️ change value
```

![Copy team, workspace and project IDs from context menu](https://github.com/supervisely/developer-portal/assets/79905215/b8e8e655-2a4d-46aa-a204-5409b1a18773)

**Step 5.** Start debugging `src/main.py`&#x20;

![Debug tutorial in Visual Studio Code](https://github.com/supervisely/developer-portal/assets/79905215/6b362bf6-ec4b-420a-a363-66c10907470c)

## Python code

### Import libraries

```python
import os
from pathlib import Path
from time import sleep

import requests
import supervisely as sly
from dotenv import load_dotenv
from ultralytics import YOLO
```

### Load environment variables

Load environment variables with credentials, team ID, project ID, and workspace ID:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
```

### Get variables

```python
GLOBAL_TIMEOUT = 1  # seconds
AGENT_ID = 230  # agent id to run training on, learn how to  https://youtu.be/aO7Zc4kTrVg
PROJECT_ID = sly.env.project_id()
TEAM_ID = sly.env.team_id()
WORKSPACE_ID = sly.env.workspace_id()
TASK_TYPE = "object detection"
DATA_DIR = "data"
TEST_DIR = os.path.join(DATA_DIR, "test")
```

### Prepare function for train model

```python
def train_model(api: sly.Api) -> Path:
    train_app_name = "supervisely-ecosystem/yolov8/train"

    module_id = api.app.get_ecosystem_module_id(train_app_name)
    module_info = api.app.get_ecosystem_module_info(module_id)
    project_name = api.project.get_info_by_id(PROJECT_ID).name

    sly.logger.info(f"Starting AutoTrain for application {module_info.name}")

    params = module_info.get_arguments(images_project=PROJECT_ID)

    session = api.app.start(
        agent_id=AGENT_ID,
        module_id=module_id,
        workspace_id=WORKSPACE_ID,
        description=f"AutoTrain session for {module_info.name}",
        task_name="AutoTrain/train",
        params=params,
        app_version=APP_VERSION,
        is_branch=BRANCH,
    )

    task_id = session.task_id
    domain = sly.env.server_address()
    token = api.task.get_info_by_id(task_id)["meta"]["sessionToken"]
    post_shutdown = f"{domain}/net/{token}/sly/shutdown"

    while not api.task.get_status(task_id) is api.task.Status.STARTED:
        sleep(GLOBAL_TIMEOUT)
    else:
        sleep(10)  # still need a time after status changed

    sly.logger.info(f"Session started: #{task_id}")

    api.task.send_request(
        task_id,
        "auto_train",
        data={
            "project_id": PROJECT_ID,
            "task_type": TASK_TYPE,
        },
        timeout=10e6,
    )

    team_files_folder = Path("/yolov8_train") / TASK_TYPE / project_name / str(task_id)
    weights = Path(team_files_folder) / "weights"
    best = None

    while best is None:
        sleep(GLOBAL_TIMEOUT)
        if api.file.dir_exists(TEAM_ID, str(weights)):
            for filename in api.file.listdir(TEAM_ID, str(weights)):
                if os.path.basename(filename).startswith("best"):
                    best = str(weights / filename)
                    sly.logger.info(f"Checkpoint founded : {best}")

    requests.post(post_shutdown)

    return team_files_folder, best
```

### Prepare function for downloading model weights from Team files

```python
def download_weight(api: sly.Api, remote_weight_path):
    # download best weight from Supervisely Team Files
    weight_name = os.path.basename(remote_weight_path)
    weight_dir = os.path.join(DATA_DIR, "weights")
    local_weight_path = os.path.join(weight_dir, weight_name)
    if sly.fs.dir_exists(weight_dir):
        sly.fs.remove_dir(weight_dir)

    api.file.download(TEAM_ID, remote_weight_path, local_weight_path)
    return local_weight_path
```

### Prepare function to get predictions

```python
def get_predictions(local_weight_path):
    # Load your model
    model = YOLO(local_weight_path)

    # Predict on an image
    results = model(image_path)
    class_names = model.names

    return results, class_names
```

### Prepare function to upload prediction to Supervisely

```python
def upload_predictions(api: sly.Api, results, class_names, image_path):
    labels = []
    obj_classes = []
    for name in class_names.values():
        obj_classes.append(sly.ObjClass(name, sly.Rectangle))
    project_meta = sly.ProjectMeta(obj_classes=obj_classes)

    # Process results list
    for result in results:
        boxes = result.boxes.cpu().numpy()  # bbox outputs
        for box in boxes:
            class_name = class_names[int(box.cls[0])]
            obj_class = project_meta.get_obj_class(class_name)
            left, top, right, bottom = box.xyxy[0].astype(int)
            bbox = sly.Rectangle(top, left, bottom, right)
            labels.append(sly.Label(bbox, obj_class))

    new_project = api.project.create(WORKSPACE_ID, "predictions", change_name_if_conflict=True)
    new_dataset = api.dataset.create(new_project.id, "dataset")
    api.project.update_meta(new_project.id, project_meta.to_json())
    image_info = api.image.upload_path(new_dataset.id, "image.jpeg", image_path)
    ann = sly.Annotation((image_info.height, image_info.width), labels=labels)
    api.annotation.upload_ann(image_info.id, ann)

    return new_project
```

### Run script

```python
if __name__ == "__main__":
    api = sly.Api()
    result_folder, best_weight = train_model(api)
    sly.logger.info("Training completed")
    sly.logger.info(
        "The weights of trained model, predictions visualization and other training artifacts can be found in the following Team Files folder:"
    )
    best_weight_local = download_weight(api, best_weight)
    results, class_names = get_predictions(best_weight)
    new_project = upload_predictions(api, results, class_names, image_path)
    sly.logger.info(f"New project created. ID: {new_project.id}, name: {new_project.name}")
```

## Explore training artefacts in Team files

Training process generates artifacts including model weights (checkpoints), logs, charts, additional visualizations of training batches, predictions on validation, precision-recall curves, confusion matrix and so on. At the last step of the training dashboard you will see the locathin and direct link where the resulting directory with training artifacts is saved.

It is automatically uploaded from the computer used for training back to the platform to Team Files. You can find it there at any time.

![Training artifacts are saved to your account to Team Files](https://github.com/supervisely/developer-portal/assets/79905215/993547b1-fcdd-4e12-ab93-93f904168f5d)
