---
description: >-
  Step-by-step tutorial explains how to integrate custom instance segmentation
  neural network into Supervisely platform
---

# Instance segmentation

## Introduction

This tutorial will teach you how to integrate your custom instance segmentation model into Supervisely by creating a simple serving app.&#x20;

✅ **Integration process is simple** - the only thing you need is to implement how to apply your model to an image. Supervisely SDK will handle the rest automatically.

{% hint style="info" %}
If you are using popular machine learning frameworks, you can skip integration and start using already existing apps in [Supervisely Ecosystem](https://ecosystem.supervise.ly/). Most popular neural network frameworks are already integrated into Supervisely. Users can train these models on their data and test them (inference) right in the platform in a few clicks.

We highly recommend to explore apps in [Supervisely Ecosystem](https://ecosystem.supervise.ly/), here are several examples of ready-to-use frameworks for instance segmentation:

* MMDetection [![GitHub Org's stars](https://camo.githubusercontent.com/bf25a249878d6417d7ab913069e1868e6e1c56baa2ec4f6dd4c5806e6d9c578f/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f6f70656e2d6d6d6c61622f6d6d646574656374696f6e3f7374796c653d736f6369616c)](https://camo.githubusercontent.com/bf25a249878d6417d7ab913069e1868e6e1c56baa2ec4f6dd4c5806e6d9c578f/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f6f70656e2d6d6d6c61622f6d6d646574656374696f6e3f7374796c653d736f6369616c) - apps for [training](https://ecosystem.supervise.ly/apps/mmdetection/train) and [inference](https://ecosystem.supervise.ly/apps/mmdetection/serve)
* Detectron2 [![GitHub Org's stars](https://camo.githubusercontent.com/709465743709c522feb07a94a3a9598a3585cc3e2b54324cb4f7bdce107a6506/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f66616365626f6f6b72657365617263682f646574656374726f6e323f7374796c653d736f6369616c)](https://camo.githubusercontent.com/709465743709c522feb07a94a3a9598a3585cc3e2b54324cb4f7bdce107a6506/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f66616365626f6f6b72657365617263682f646574656374726f6e323f7374796c653d736f6369616c) - apps for [training](https://ecosystem.supervise.ly/apps/detectron2/supervisely/train) and [inference](https://ecosystem.supervise.ly/apps/detectron2/supervisely/instance\_segmentation/serve)

If your favorite NN framework is not in our Ecosystem yet, you can send us a feature request in [Supervisely Ideas Exchange](https://ideas.supervise.ly/).
{% endhint %}

## Benefits

Once you implement a serving application for your NN architecture, you can do a lot of things, like inference on your data for pre-labeling to speed up annotation, perform active learning, analyze and debug your model with various data science tools, combine models into pipelines and many more.&#x20;

Find more use cases and video tutorials on [our youtube channel](https://www.youtube.com/c/Supervisely).

{% hint style="success" %}
Generally speaking, your model will be compatible with the entire ecosystem of applications in Supervisely.
{% endhint %}

&#x20;Here are the examples of apps you might be interested to use with your model:

* [`Apply NN to Images Project` app](https://ecosystem.supervise.ly/apps/nn-image-labeling/project-dataset) - apply NN to your images and save predictions&#x20;
* [`NN Image Labeling` app](https://ecosystem.supervise.ly/apps/nn-image-labeling/annotation-tool) - use NN right in labeling interface
* [`Apply Detection and Classification Models to Images Project` app](https://ecosystem.supervise.ly/apps/apply-det-and-cls-models-to-project) - combine models into pipelines
* [`Apply NN to Videos Project` app](https://ecosystem.supervise.ly/apps/apply-nn-to-videos-project) - predict and track objects on videos
* Analyze model performance metrics ([app1](https://ecosystem.supervise.ly/apps/review\_object\_detection\_metrics/supervisely), [app2](https://ecosystem.supervise.ly/apps/semantic-segmentation-metrics-dashboard))
* and so on ...

## How to debug this tutorial

**Step 1.** Prepare  `~/supervisely.env` file with credentials. [Learn more here.](../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/integrate-inst-seg-model) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/integrate-inst-seg-model
cd integrate-inst-seg-model
./create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.&#x20;

```bash
code -r .
```

**Step 4.** Start debugging `src/main.py`

## Python script

The integration script is really simple:

1. Automatically downloads NN weights to `./my_model` folder
2. Loads model on the CPU or GPU device
3. Runs inference on a demo image
4. Visualizes predictions on top of the input image

The entire integration Python script takes only 👍 **87 lines** of code (including comments):

```python
import os
from typing_extensions import Literal
from typing import List
import cv2
import json
from dotenv import load_dotenv
import torch
import supervisely as sly

from detectron2 import model_zoo
from detectron2.engine import DefaultPredictor
from detectron2.config import get_cfg
from detectron2.data import MetadataCatalog

from src.demo_data import prepare_weights

load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
prepare_weights()  # prepare demo data automatically for convenient debug

# code for detectron2 inference copied from official COLAB tutorial (inference section):
# https://colab.research.google.com/drive/16jcaJoc6bCFAQ96jDe2HwtXj7BMD_-m5
# https://detectron2.readthedocs.io/en/latest/tutorials/getting_started.html


class MyModel(sly.nn.inference.InstanceSegmentation):
    def load_on_device(
        self,
        device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
    ):
        ####### CUSTOM CODE FOR MY MODEL STARTS (e.g. DETECTRON2) #######
        with open(os.path.join(self.model_dir, "model_info.json"), "r") as myfile:
            architecture = json.loads(myfile.read())["architecture"]
        cfg = get_cfg()
        cfg.merge_from_file(model_zoo.get_config_file(architecture))
        cfg.MODEL.DEVICE = device  # learn more in torch.device
        cfg.MODEL.WEIGHTS = os.path.join(self.model_dir, "weights.pkl")
        self.predictor = DefaultPredictor(cfg)
        self.class_names = MetadataCatalog.get(cfg.DATASETS.TRAIN[0]).get("thing_classes")
        ####### CUSTOM CODE FOR MY MODEL ENDS (e.g. DETECTRON2)  ########
        print(f"✅ Model has been successfully loaded on {device.upper()} device")

    def get_classes(self) -> List[str]:
        return self.class_names  # e.g. ["cat", "dog", ...]

    def predict(
        self, image_path: str, confidence_threshold: float = 0.8
    ) -> List[sly.nn.PredictionMask]:
        image = cv2.imread(image_path)  # BGR

        ####### CUSTOM CODE FOR MY MODEL STARTS (e.g. DETECTRON2) #######
        outputs = self.predictor(image)  # get predictions from Detectron2 model
        pred_classes = outputs["instances"].pred_classes.detach().numpy()
        pred_class_names = [self.class_names[pred_class] for pred_class in pred_classes]
        pred_scores = outputs["instances"].scores.detach().numpy().tolist()
        pred_masks = outputs["instances"].pred_masks.detach().numpy()
        ####### CUSTOM CODE FOR MY MODEL ENDS (e.g. DETECTRON2)  ########

        results = []
        for score, class_name, mask in zip(pred_scores, pred_class_names, pred_masks):
            # filter predictions by confidence
            if score >= confidence_threshold:
                results.append(sly.nn.PredictionMask(class_name, mask, score))
        return results


model_dir = sly.env.folder()
print("Model directory:", model_dir)

device = "cuda" if torch.cuda.is_available() else "cpu"
print("Using device:", device)

m = MyModel(model_dir)
m.load_on_device(device)

if sly.is_production():
    # this code block is running on Supervisely platform in production
    # just ignore it during development
    m.serve()
else:
    # for local development and debugging
    image_path = "./demo_data/image_01.jpg"
    confidence_threshold = 0.7
    results = m.predict(image_path, confidence_threshold)
    vis_path = "./demo_data/image_01_prediction.jpg"
    m.visualize(results, image_path, vis_path)
    print(f"predictions and visualization have been saved: {vis_path}")

```

Here are the input image and output predictions:

| Input                                                                                                      | Output                                                                                                     |
| ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| ![](https://user-images.githubusercontent.com/12828725/195988529-a31f2b97-43a8-4c16-82a4-9d2f85b27828.jpg) | ![](https://user-images.githubusercontent.com/12828725/195988525-9fdd56d5-f0da-4b2c-9226-2a1b1bce49ae.jpg) |

## Implementation details

To integrate your model, you need to subclass **`sly.nn.inference.InstanceSegmentation`** and implement 3 methods:

* `load_on_device` - that takes a device name as input and loads weights to the device from the model directory (`self.model_dir`)
* `get_classes` - returns the list of class names (strings) that model predicts
* `predict` - takes the path to an image and confidence threshold as arguments, applies the model to the input image, and returns the list of predictions (`sly.nn.PredictionMask`)

The beauty of this method is that you can easily debug your implementation locally in your favorite IDE. Once the code is ready, when you run it on the Supervisely platform the following lines will be executed.&#x20;

```python
if sly.is_production():
    m.serve()
else:
    # ...
```

The method `m.serve()` handles everything and deploys your model as REST API service on the Supervisely platform. It means that other applications are able to communicate with your model and get predictions from it.

## Create serving app from python script

### Repository structure

The structure of [our GitHub repository](https://github.com/supervisely-ecosystem/integrate-inst-seg-model) is the following:

```
.
├── README.md
├── config.json
├── create_venv.sh
├── dev_requirements.txt
├── demo_data
│   ├── image_01.jpg
│   └── image_01_prediction.jpg
├── docker
│   ├── Dockerfile
│   └── publish.sh
├── local.env
├── my_model
│   ├── init.sh
│   └── model_info.json
└── src
    ├── demo_data.py
    └── main.py
```

Explanation:

* `src/main.py` - main inference script
* `src/demo_data.py` - auxiliary functionality that downloads NN weights for demo model only during debugging&#x20;
* `my_model` - directory with model weights and additional config files
* `demo_data` - directory with demo image for inference
* `README.md` - readme of your application, it is the main page of an application in Ecosystem with some images, videos, and how-to-use guides
* `config.json` - configuration of the Supervisely application, which defines the name and description of the app, its context menu, icon, poster, and running settings
* `create_venv.sh` - creates a virtual environment, installs detectron2 framework, includes the support of Apple CPUs (m1 / m2 ...) &#x20;
* `dev_requirements.txt` - all packages needed for debugging &#x20;
* `local.env` - file with variables used for debugging
* `docker` - directory with the custom Dockerfile for this application and the script that builds it and publishes it to the docker registry

### App configuration

App configuration is stored in `config.json` file. A detailed explanation of all possible fields in app configuration will be covered in other tutorials. Let's check the config for our current app: &#x20;

```json
{
  "type": "app",
  "version": "2.0.0",
  "name": "Serve custom instance segmentation model",
  "description": "Learn how to integrate your custom model in readme",
  "categories": [
    "neural network",
    "images",
    "videos",
    "instance segmentation",
    "segmentation & tracking",
    "serve",
    "development"
  ],
  "icon": "some url",
  "poster": "some url",
  "context_menu": {
    "target": ["files_folder"]
  },
  "session_tags": ["deployed_nn"],
  "community_agent": false,
  "docker_image": "supervisely/detectron2-demo:1.0.2",
  "entrypoint": "python -m uvicorn src.main:m.app --host 0.0.0.0 --port 8000",
  "port": 8000,
  "headless": true
}
```

Here is the explanation for all fileds:

* `type` - type of the module in Supervisely Ecosystem
* `version` - version of Supervisely App Engine. Just  keep it by default
* `name` - the name of the application
* `description` - the description of the application
* `categories` - these tags are used to place the application in the correct category in Ecosystem.
* `icon`, `poster` - image URLs for icon and poster&#x20;
* `context_menu` - defines that this app have to be run from the context menu of a directory in Team Files
* `session_tags` - these tags will be assigned to every running session of the application. They can be used by other apps to find and filter all running sessions&#x20;
* `"community_agent": false` - this means that this app can not be run on the agents started by Supervisely team, so users have to connect their own computers and run the app only on their own agents. Only applicable in Community Edition. Enterprise customers use their private instances so they can ignore current option
* `docker_image` - Docker container will be started from the defined Docker image, github repository will be downloaded and mounted inside the container.
* `entrypoint` - the command that starts our application in a container
* `port` -  port inside the container
* `"headless": true` means that the app has no User Interface

### How to add your private app

Supervisely supports both private and public apps.&#x20;

🔒 **Private apps** are those that are available only on private Supervisely Instances (Enterprise Edition).

🌎 **Public apps** are available on all private Supervisely Instances and in Community Edition. The guidelines for adding public apps will be covered in other tutorials.&#x20;

Since Supervisely app is just a git repository, we support public and private repos from the most popular hosting platforms in the world - **GitHub** and **GitLab**. You just need to generate and provide  access token to your repo. Learn more in [the documentation](https://docs.supervise.ly/enterprise-edition/advanced-tuning/private-apps).

Go to `Ecosystem` -> `Private Apps` -> `Add private app`.&#x20;

![Add private app](https://user-images.githubusercontent.com/12828725/182870411-6632dde4-93ed-481c-a8c2-79718b0f5a7d.gif)

## Run your app from the context menu

### Prepare model directory

We need to drag-and-drop local directory with our model weights to the Team Files:

**To be done**

### Run app

Now, we have the app added to Supervisely. Let's run it:

**To be done**



