---
description: >-
  Step-by-step tutorial of how to integrate custom object detection
  neural network into Supervisely platform on the example of detectron2.
---

# Integrate object detection

## Introduction

In this tutorial you will learn how to integrate your custom object detection model into Supervisely by creating a simple serving app. As an example, we will use [detectron2](https://github.com/facebookresearch/detectron2) repository.

## Getting started

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/integrate-obj-det-model) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/integrate-obj-det-model
cd integrate-obj-det-model
./create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 4.** Run debug for script `src/main.py`

## Python script

The integration script is simple:

1. Automatically downloads NN weights to `./my_model` folder
2. Loads model on the CPU or GPU device
3. Runs inference on a demo image
4. Visualizes predictions on top of the input image

The entire integration Python script takes only 👍 **90 lines** of code (including comments) and can be found in [GitHub repository](https://github.com/supervisely-ecosystem/integrate-obj-det-model) for this tutorial.


## Implementation details

To integrate object detection model, you need to subclass **`sly.nn.inference.ObjectDetection`** and implement 3 methods:

* `load_on_device` method for downloading the weights and initializing the model on a specific device. Takes a `model_dir` argument, that is a directory for all model files (like configs, weights, etc). The second argument is a `device` - a `torch.device` like `cuda:0`, `cpu`.
* `get_classes` method should return a list of class names (strings) that model can predict.
* `predict`. The core implementation of a model inference. It takes a path to an image and inference settings as arguments, applies the model inference to the image and returns a list of predictions (which are `sly.nn.PredictionBBox` objects).


### Overall structure

The overall structure of the class we will implement is looking like this:

```python
class MyModel(sly.nn.inference.ObjectDetection):
    def load_on_device(
        self,
        model_dir: str,
        device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
    ):
        # preparing the model: model instantiating, downloading weights, loading it on device.
        pass

    def get_classes(self) -> List[str]:
        # returns a list of supported classes, e.g. ["cat", "dog", ...]
        # ...
        return class_names

    def predict(self, image_path: str, settings: Dict[str, Any]) -> List[sly.nn.PredictionBBox]:
        # the inference of a model here
        # ...
        return prediction
```

The superclass has a `serve()` method. To run the code and deploy the model on the Supervisely platform, `m.serve()` method should be executed:

```python
if sly.is_production():
    m.serve()
else:
    # ...
```

And here is the beauty comes in. The method `serve()` internally handles everything and deploys your model as a **REST API** service on the Supervisely platform. It means that other applications are able to communicate with your model and get predictions from it.


So let's implement the class.


### Step-by-step implementation

**Defining imports and global variables**

```python
import os
from typing_extensions import Literal
from typing import List, Any, Dict
import cv2
import json
from dotenv import load_dotenv
import torch
import supervisely as sly

from detectron2 import model_zoo
from detectron2.engine import DefaultPredictor
from detectron2.config import get_cfg
from detectron2.data import MetadataCatalog


load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

device = "cuda" if torch.cuda.is_available() else "cpu"
print("Using device:", device)

weights_url = "https://dl.fbaipublicfiles.com/detectron2/COCO-Detection/faster_rcnn_R_50_FPN_3x/137849458/model_final_280758.pkl"
```

**1. load_on_device**

The following code downloads model weights and builds the model according to config in `my_model/model_info.json`. 
Also it will keep the model as a `self.predictor` and classes as `self.class_names` for further use:

```python
class MyModel(sly.nn.inference.ObjectDetection):
    def load_on_device(
        self,
        model_dir: str,
        device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
    ):
        ####### CUSTOM CODE FOR MY MODEL STARTS (e.g. DETECTRON2) #######
        weights_path = self.download(weights_url)
        model_info = sly.json.load_json_file(os.path.join(model_dir, "model_info.json"))
        architecture_name = model_info["architecture"]
        cfg = get_cfg()
        cfg.merge_from_file(model_zoo.get_config_file(architecture_name))
        cfg.MODEL.DEVICE = device  # learn more in torch.device
        cfg.MODEL.WEIGHTS = weights_path
        self.predictor = DefaultPredictor(cfg)
        self.class_names = MetadataCatalog.get(cfg.DATASETS.TRAIN[0]).get("thing_classes")
        ####### CUSTOM CODE FOR MY MODEL ENDS (e.g. DETECTRON2)  ########
        print(f"✅ Model has been successfully loaded on {device.upper()} device")
```

{% hint style="info" %}
Here we are downloading the model weights by **url**, but it can be also downloaded by path in Supervisely **Team Files**. You can even pass a path to folder with the model, then an entire folder will be downloaded.
{% endhint %}

**2. get_classes**

Simply returns previously saved **class_names**:

```python
    def get_classes(self) -> List[str]:
        return self.class_names  # e.g. ["cat", "dog", ...]
```

**3. predict**

The core method for model inference. Here we are reading an image and getting an inference of the model. The code here is usually borrowed from the framework or the model you use, that is **detectron2** in our case. Then we wrap the model prediction into a `sly.nn.PredictionBBox` class and do some post-processing steps.

```python
    def predict(
        self, image_path: str, settings: Dict[str, Any]
    ) -> List[sly.nn.PredictionBBox]:
        confidence_threshold = settings.get("confidence_threshold", 0.5)
        image = cv2.imread(image_path)  # BGR

        ####### CUSTOM CODE FOR MY MODEL STARTS (e.g. DETECTRON2) #######
        outputs = self.predictor(image)  # get predictions from Detectron2 model
        pred_classes = outputs["instances"].pred_classes.detach().cpu().numpy()
        pred_class_names = [self.class_names[pred_class] for pred_class in pred_classes]
        pred_scores = outputs["instances"].scores.detach().cpu().numpy().tolist()
        pred_bboxes = outputs["instances"].pred_boxes.tensor.detach().cpu().numpy()
        ####### CUSTOM CODE FOR MY MODEL ENDS (e.g. DETECTRON2)  ########

        results = []
        for score, class_name, bbox in zip(pred_scores, pred_class_names, pred_bboxes):
            # filter predictions by confidence
            if score >= confidence_threshold:
                bbox = [bbox[1], bbox[0], bbox[3], bbox[2]]
                results.append(sly.nn.PredictionBBox(class_name, bbox, score))
        return results
```

It **must** return exactly the list of `sly.nn.PredictionBBox` objects for compatibility with Supervisely. Your code should just wrap the model predictions: `sly.nn.PredictionBBox(class_name, bbox, score)`, where the `class_name` is a `str`, `bbox` is a list of 4 int coordinates `[top, left, bottom, right]` and the `score` is a model `confidence_score`.

**Usage of our class**

Once the class is created, here we initialize it and get one test prediction for debugging.

{% hint style="info" %}
In the code below a `custom_inference_settings` is used. It allows us to provide a custom settings that could be used in `predict()` (See more in [Customized Inference Tutorial](https://developer.supervise.ly/app-development/neural-network-integration/inference/customize-inference))
{% endhint %}

```python
model_dir = "my_model"  # model weights will be downloaded into this dir
settings = {"confidence_threshold": 0.7}

m = MyModel(model_dir=model_dir, custom_inference_settings=settings)
m.load_on_device(model_dir=model_dir, device=device)

if sly.is_production():
    # this code block is running on Supervisely platform in production
    # just ignore it during development
    m.serve()
else:
    # for local development and debugging
    image_path = "./demo_data/image_01.jpg"
    results = m.predict(image_path, settings)
    vis_path = "./demo_data/image_01_prediction.jpg"
    m.visualize(results, image_path, vis_path)
    print(f"predictions and visualization have been saved: {vis_path}")
```

Here are the input image and output predictions:

| Input                                                                                                      | Output                                                                                                     |
| ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| ![](https://user-images.githubusercontent.com/12828725/195988529-a31f2b97-43a8-4c16-82a4-9d2f85b27828.jpg) | ![](https://user-images.githubusercontent.com/12828725/195988525-9fdd56d5-f0da-4b2c-9226-2a1b1bce49ae.jpg) |


---


## Run and debug

The beauty of this class is that you can easily debug your code locally in your favorite IDE.

{% hint style="info" %}
For now, we recommend using **Visual Studio Code** IDE, because our repositories have prepared settings for convenient debugging in VSCode. It is the easiest way to start.
{% endhint %}


### Local debug

You can run the code locally for debugging. For **Visual Studio Code** we've created a `launch.json` config file that can be selected:

![Local debug](https://user-images.githubusercontent.com/31512713/223177253-e4475c1f-6909-43d5-99bd-d1f6310c7f48.png)

### Debug in Supervisely platform

Once the code seems working locally, it's time to test the code right in the Supervisely platform as a debugging app. For that: 

1. If you develop in a Docker container, you should run the container with `--cap_add=NET_ADMIN` option.

2. Install `sudo apt-get install wireguard iproute2`.

3. Define your `TEAM_ID` in the `local.env` file. *Actually there are other env variables that is needed, but they are already provided in `./vscode/launch.json` for you.*

4. Switch the `launch.json` config to the `Advanced debug in Supervisely platform`:

![Advanced Debug in Supervisely](https://user-images.githubusercontent.com/31512713/224290229-5da93fd2-dc97-4911-abb5-66ce890485a2.png)

5. Run the code.

✅ It will deploy the model in the Supervisely platform as a regular serving App that is able to communicate with all others app in the platform:

![Develop and Debug](https://user-images.githubusercontent.com/31512713/223178384-cf316096-fc23-4e32-80fc-4288bad415be.png)

{% hint style="success" %}
Now you can use apps like [Apply NN to Images](https://ecosystem.supervise.ly/apps/nn-image-labeling/project-dataset), [Apply NN to videos](https://ecosystem.supervise.ly/apps/apply-nn-to-videos-project) with your deployed model.

Or get the model inference via **Python API** with the help of `sly.nn.inference.Session` class just in one line of code. See [Inference API Tutorial](https://developer.supervise.ly/app-development/neural-network-integration/inference-api-tutorial).
{% endhint %}


## Release your code as a Supervisely App.

Once you've tested the code, it's time to release it into the platform. It can be released as an App that shared with the all Supervisely community, or as your own private App.

Refer to [How to Release your App](https://developer.supervise.ly/app-development/basics/from-script-to-supervisely-app) for all releasing details. For a private app check also [Private App Tutorial](https://developer.supervise.ly/app-development/basics/add-private-app).

In this tutorial we'll quickly observe the key concepts of our app.


### Repository structure

The structure of [our GitHub repository](https://github.com/supervisely-ecosystem/integrate-obj-det-model) is the following:

```
.
├── README.md
├── config.json
├── create_venv.sh
├── requirements.txt
├── demo_data
│   ├── image_01.jpg
│   └── image_01_prediction.jpg
├── docker
│   ├── Dockerfile
│   └── publish.sh
├── local.env
├── my_model
│   └── model_info.json
└── src
    └── main.py
```

Explanation:

* `src/main.py` - main inference script
* `my_model` - directory with model weights and additional config files
* `demo_data` - directory with demo image for inference
* `README.md` - readme of your application, it is the main page of an application in Ecosystem with some images, videos, and how-to-use guides
* `config.json` - configuration of the Supervisely application, which defines the name and description of the app, its context menu, icon, poster, and running settings
* `create_venv.sh` - creates a virtual environment, installs detectron2 and requirements.
* `requirements.txt` - all needed packages
* `local.env` - file with env variables used for debugging
* `docker` - directory with the custom Dockerfile for this application and the script that builds it and publishes it to the docker registry


### App configuration

App configuration is stored in `config.json` file. A detailed explanation of all possible fields is covered in this [Configuration Tutorial](https://developer.supervise.ly/app-development/basics/app-json-config). Let's check the config for our current app: 

```json
{
  "type": "app",
  "version": "2.0.0",
  "name": "Serve custom object detection model",
  "description": "Demo app of integrating your custom object detection model",
  "categories": [
    "neural network",
    "images",
    "videos",
    "object detection",
    "detection & tracking",
    "serve",
    "development"
  ],
  "session_tags": ["deployed_nn"],
  "need_gpu": true,
  "community_agent": false,
  "docker_image": "supervisely/detectron2-demo:1.0.3",
  "entrypoint": "python -m uvicorn src.main:m.app --host 0.0.0.0 --port 8000",
  "port": 8000,
  "headless": true
}
```

Here is an explanation for the fields:

* `type` - type of the module in Supervisely Ecosystem
* `version` - version of Supervisely App Engine. Just keep it by default
* `name` - the name of the application
* `description` - the description of the application
* `categories` - these tags are used to place the application in the correct category in Ecosystem.
* `session_tags` - these tags will be assigned to every running session of the application. They can be used by other apps to find and filter all running sessions
* `"need_gpu": true` - should be true if you want to use any `cuda` devices.
* `"community_agent": false` - this means that this app can not be run on the agents started by Supervisely team, so users have to connect their own computers and run the app only on their own agents. Only applicable in Community Edition. Enterprise customers use their private instances so they can ignore current option
* `docker_image` - Docker container will be started from the defined Docker image, github repository will be downloaded and mounted inside the container.
* `entrypoint` - the command that starts our application in a container
* `port` - port inside the container
* `"headless": true` means that the app has no User Interface
