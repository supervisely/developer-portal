# Train and predict automation model pipeline

## Introduction

Welcome to the Model Automation Training and Prediction tutorial!

In this guide, you'll learn how to automatically train a computer vision model and use it to make predictions on local images directly from your Python code.

This tutorial provides you with the necessary steps to achieve the following:

- Automatically run training with given or default parameters.
- Download pre-trained model weights from Team files where all generated artifacts will be saved.
- Perform inference with a pre-trained model on local images to obtain object detection predictions.
- Upload annotated images to Supervisely

![Automated pipeline schema](https://github.com/supervisely/developer-portal/assets/79905215/29678b69-a109-494d-8ef2-63c3f2434905)

💻 We wll use a 196 lines of Python code in [main.py](https://github.com/supervisely-ecosystem/model-automation-train-and-predict-pipeline/blob/master/src/main.py) to demonstrate the entire process.

{% hint style="info" %}
In this demo script we will use to automate the process of training the *YOLOv8* model, but this workflow is applicable to other models as well.
{% endhint %}

Before we dive into the tutorial, let's learn how to debug it.

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

**Step 4.** change ✅ workspace ID, team ID, and project ID ✅ in `local.env` file by copying the ID from the context menu. A new project with annotated images will be created in the workspace you define. [Learn more here.](../../getting-started/environment-variables.md#team_id)

```python
TEAM_ID=481        # ⬅️ change value
WORKSPACE_ID=885   # ⬅️ change value
PROJECT_ID=24640   # ⬅️ change value
DATASET_ID=70781   # ⬅️ change value

SLY_APP_DATA_DIR="data_dir"
```

![Copy team, workspace and project IDs from context menu](https://github.com/supervisely/developer-portal/assets/79905215/ec948aa2-a619-42cc-aed9-61f8ada6a388)

**Step 5.** Start debugging `src/main.py`&#x20;

Go to `Run and Debug` section (`Ctrl`+`Shift`+`D`). Press `green triangle` or `F5` to start debugging.

![Debug tutorial in Visual Studio Code](https://github.com/supervisely/developer-portal/assets/79905215/6b362bf6-ec4b-420a-a363-66c10907470c)

{% hint style="success" %}
Suppervisely allows you to connect your own computers with GPU to the platform and use them for model training, inference and evaluation ✨ for FREE. It is as simple as running a single command in the terminal on your machine.

🔗 Watch the short [video](https://youtu.be/aO7Zc4kTrVg) to learn how to connect your machine.
{% endhint %}

## Prepare labeled data

You can use on of our [demo projects](https://dev.supervisely.com/ecosystem/projects).

![Get demo project](https://github.com/supervisely/developer-portal/assets/79905215/a8884896-cbda-4feb-b2e1-d50081d5dcab)

If you already have the **labeled data** — just upload it into Supervisely platform using one of the [70+ import Supervisely Apps](https://ecosystem.supervisely.com/import) from our Ecosystem. You will find there the imports for all popular data formats in computer vision.

## Python code

### Import libraries

```python
import os
from pathlib import Path
from time import sleep

import cv2
import requests
import supervisely as sly
import torch
from dotenv import load_dotenv
from ultralytics import YOLO
```

### Load environment variables

Load environment variables with credentials, team ID, project ID, and workspace ID.
Init api for communicating with Supervisely Instance.

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Define variables

```python
GLOBAL_TIMEOUT = 1  # seconds
AGENT_ID = 230  # agent id to run training on
APP_NAME = "supervisely-ecosystem/yolov8/train"
PROJECT_ID = sly.env.project_id()
DATASET_ID = sly.env.dataset_id()
TEAM_ID = sly.env.team_id()
WORKSPACE_ID = sly.env.workspace_id()
DATA_DIR = sly.app.get_data_dir()
task_type = "object detection"  # you can choose "instance segmentation" or "pose estimation"
image_path = os.path.join("data_dir/test/image3.png") # ⬅️ change value to your image path
```

Set the path to the image you want to predict on

### Train model

```python
module_id = api.app.get_ecosystem_module_id(APP_NAME)
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
    app_version="auto-train",
    is_branch=True,
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
```

{% hint style="info" %}
📗 By changing `data` field you can customize training parameters such as: **project id** and **dataset ids** to train on, **train mode** (finetune or scratch), **number of epochs**, **patience**, **batch size**, **input image size**, **optimizer**, **number of workers**, **learning rate**, **momentum**, **weight decay**, **warmup epochs**, **warmup momentum**, **warmup bias lr**, **augmentation parameters**, and many others.
{% endhint %}

```python
# 📗 You can set any parameters you want to customize training in the data field
api.task.send_request(
    task_id,
    "auto_train",
    data={
        "project_id": PROJECT_ID,
        # "dataset_ids": [DATASET_ID], # optional (specify if you want to train on specific datasets)
        "task_type": task_type,
        "train_mode": "finetune",  # finetune / scratch
        "n_epochs": 100,
        "patience": 50,
        "batch_size": 16,
        "input_image_size": 640,
        "optimizer": "AdamW",  # AdamW, Adam, SGD, RMSProp
        "n_workers": 8,
        "lr0": 0.01,
        "lrf": 0.01,
        "momentum": 0.937,
        "weight_decay": 0.0005,
        "warmup_epochs": 3.0,
        "warmup_momentum": 0.8,
        "warmup_bias_lr": 0.1,
        "amp": "true",
        "hsv_h": 0.015,
        "hsv_s": 0.7,
        "hsv_v": 0.4,
        "degrees": 0.0,
        "translate": 0.1,
        "scale": 0.5,
        "shear": 0.0,
        "perspective": 0.0,
        "flipud": 0.0,
        "fliplr": 0.5,
        "mosaic": 0.0,
        "mixup": 0.0,
        "copy_paste": 0.0,
    },  # 📗 train paramaters
    timeout=10e6,
)

team_files_folder = Path("/yolov8_train") / task_type / project_name / str(task_id)
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

sly.logger.info("Training completed")
sly.logger.info(
    "The weights of trained model, predictions visualization and other training artifacts can be found in the following Team Files folder:"
)
```

## Explore training artefacts in Team files

Training process generates artifacts including model weights (checkpoints), logs, charts, additional visualizations of training batches, predictions on validation, precision-recall curves, confusion matrix and so on. At the last step of the training dashboard you will see the location and direct link where the resulting directory with training artifacts is saved.

It is automatically uploaded from the computer used for training back to the platform to Team Files. You can find it there at any time.

![Training artifacts are saved to your account to Team Files](https://github.com/supervisely/developer-portal/assets/79905215/993547b1-fcdd-4e12-ab93-93f904168f5d)

### Download model weights from Team files

```python
weight_name = os.path.basename(best)
weight_dir = os.path.join(DATA_DIR, "weights")
local_weight_path = os.path.join(weight_dir, weight_name)
if sly.fs.dir_exists(weight_dir):
    sly.fs.remove_dir(weight_dir)

api.file.download(TEAM_ID, best, local_weight_path)

sly.logger.info(f"Model weight downloaded to {local_weight_path}")
```

### Get model predictions and visualize

```python
# Load your model
model = YOLO(local_weight_path)

# define device
device = torch.device("cuda:0") if torch.cuda.is_available() else "cpu"

# load image
input_image = sly.image.read(image_path)
input_image = input_image[:, :, ::-1]

# Predict on an image
results = model(
    source=input_image,
    conf=0.25,
    iou=0.7,
    half=False,
    device=device,
    max_det=300,
    agnostic_nms=False,
)

# visualize predictions
predictions_plotted = predictions[0].plot()
cv2.imwrite(os.path.join(DATA_DIR, "predictions.jpg"), predictions_plotted)
```

![tomato](https://github.com/supervisely/developer-portal/assets/79905215/4c5bc4a3-e68f-476e-aceb-283e45577b5d)

### Upload prediction in Supervisely format

```python
# Get class names dictionary
class_names = model.names

# Create list of the sly.ObjClass objects
obj_classes = []
for name in class_names.values():
    obj_classes.append(sly.ObjClass(name, sly.Rectangle))
project_meta = sly.ProjectMeta(obj_classes=obj_classes)

# Process results list
labels = []
for result in results:
    boxes = result.boxes.cpu().numpy()  # bbox outputs
    for box in boxes:
        class_name = class_names[int(box.cls[0])]
        obj_class = project_meta.get_obj_class(class_name)
        left, top, right, bottom = box.xyxy[0].astype(int)
        bbox = sly.Rectangle(top, left, bottom, right)
        labels.append(sly.Label(bbox, obj_class))

# Create project, dataset and update project meta
project = api.project.create(WORKSPACE_ID, "predictions", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, "dataset")
api.project.update_meta(project.id, project_meta.to_json())

# Upload the image to Supervisely
image_info = api.image.upload_path(dataset.id, "image.jpeg", image_path)

# Create an annotation for the image and upload it
ann = sly.Annotation((image_info.height, image_info.width), labels=labels)
api.annotation.upload_ann(image_info.id, ann)

sly.logger.info(f"New project created. ID: {project.id}, name: {project.name}")
```

Explore result project with model predictions in Supervisely.

![results](https://github.com/supervisely/developer-portal/assets/79905215/51428454-0897-47e0-a2f6-4113c28f7231)

{% hint style="info" %}
In this tutorial we learned how to train a model using automatically train and perform inference on local image for **object detection task**.
You can also use this code for other tasks: **instance segmentation** and **pose estimation**.
Just change the `task_type` parameter in the `data` field of the request and update label creation code in the last part of the tutorial.
{% endhint %}
