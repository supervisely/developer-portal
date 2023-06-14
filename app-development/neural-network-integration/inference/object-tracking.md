---
description: >-
  Step-by-step tutorial on how to integrate custom visual object tracking
  neural network into Supervisely platform on the example of MixFormer model.
---

# Integrate object tracking

## Introduction
In this tutorial, you will learn how to integrate your custom object-tracking model into Supervisely by creating two simple serving apps. 
First, we will construct a straightforward model that only moves the original bound box as an illustration. The SOTA model [MixFormer](https://github.com/MCG-NJU/MixFormer), which already has the majority of the necessary functions implemented, will be used in the second part.

## Implementation details

To integrate your model, you need to subclass **`sly.nn.inference.BBoxTracking`** and implement 3 methods:

* `load_on_device` method for downloading the weights and initializing the model on a specific device. Takes a `model_dir` argument, which is a directory for all model files (like configs, weights, etc). The second argument is the `device` - `torch.device` like `cuda:0`, `cpu`.
* `initialize` method passes the image and the bound box of the object, which the model should track during the prediction step.
* `predict`. The core implementation of model inference. It takes the frame of type `np.ndarray`, inference settings, previous (or initial) frame and bound box as arguments, applies the model inference to the current frame and returns a prediction (both input bound box and predicted are `sly.nn.PredictionBBox` objects).

Currently, integrating models that can track several objects simultaneously is not possible due to the implementation of the `sly.nn.inference.BBox` class. However, multiobject tracking is available: objects will be tracked one by one and the model will be re-initialized for each object.

### Overall structure

The overall structure of the class we will implement is looking like this:

```python
class MyModel(sly.nn.inference.BBoxTracking):
    def load_on_device(
        self,
        model_dir: str,
        device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
    ):
        # preparing the model: model instantiating, downloading weights, loading it on the device.
        pass

    def initialize(
        self, init_rgb_image: np.ndarray, target_bbox: PredictionBBox
    ) -> None:
        # initialize model with target object
        pass

    def predict(
        self,
        rgb_image: np.ndarray,
        settings: Dict[str, Any],
        prev_rgb_image: np.ndarray,
        target_bbox: PredictionBBox,
    ) -> PredictionBBox:
        # the inference of a model here
        # ...
        return prediction
```

The superclass has a `serve()` method. For running the code on the Supervisely platform, `m.serve()` method should be executed:

```python
if sly.is_production():
    m.serve()
```

And here is the beauty comes in. The method `serve()` internally handles everything and deploys your model as a **REST API** service on the Supervisely platform. It means that other applications can communicate with your model and get predictions from it.

So let's implement the class.

## Simple model

### Getting started

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.**
Create [Virtual Environment](https://docs.python.org/3/library/venv.html) and install `supervisely==6.72.32` in it.

### Step-by-step implementation

**Defining imports and global variables**

```python
import numpy as np
from dotenv import load_dotenv
from pathlib import Path
from typing import Any, Dict, List, Literal
from typing_extensions import Literal

import supervisely as sly
import supervisely.imaging.image as sly_image
from supervisely.nn.inference import BBoxTracking
from supervisely.nn.prediction_dto import PredictionBBox


load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
```

**1. load_on_device**

The following code creates the model according to config `model_settings.yaml`. Path to `.yaml` config is passed during initialization. These settings can also be given as a Python dictionary. Config in the form of a dictionary becomes available in `self.custom_inference_settings_dict` attribute. Also, `load_on_device` will keep the model as a `self.model` for further use:

```python
class MyModel(BBoxTracking):
    def load_on_device(
        self,
        model_dir: str,
        device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
    ):
        col_shift = self.custom_inference_settings_dict.get("col_shift", 10)
        row_shift = self.custom_inference_settings_dict.get("row_shift", 7)

        shift_list = lambda tlbr: [
            tlbr[0] + row_shift,
            tlbr[1] + col_shift,
            tlbr[2] + row_shift,
            tlbr[3] + col_shift,
        ]

        self.model = lambda bbox: PredictionBBox(
            class_name="",
            bbox_tlbr=shift_list(bbox.bbox_tlbr),
            score=None,
        )
```

Our `settings.yaml` file:

```yaml
col_shift: 20
row_shift: 15
```

**2. initialize and predict**

The core methods for model inference. Here we will use the defined model and make sure that the predicted bound box is not outside of the bounds.

```python
def initialize(self, init_rgb_image: np.ndarray, target_bbox: PredictionBBox) -> None:
        pass

def predict(
    self,
    rgb_image: np.ndarray,
    settings: Dict[str, Any],
    prev_rgb_image: np.ndarray,
    target_bbox: PredictionBBox,
) -> PredictionBBox:
    pred_bbox = self.model(target_bbox)
    h, w = rgb_image.shape[0], rgb_image.shape[1]
    cur = pred_bbox.bbox_tlbr
    tlbr = [
        min(max(0, cur[0]), h),
        min(max(0, cur[1]), w),
        min(max(0, cur[2]), h),
        min(max(0, cur[3]), w),
    ]
    pred_bbox.bbox_tlbr = tlbr
    return pred_bbox
```

It **must** return exactly an `sly.nn.PredictionBBox` object for compatibility with Supervisely.

**Usage of our class**

Once the class is created, here we initialize it and get one test prediction for debugging.

```python
settings = {"col_shift": 20, "row_shift": 15}
# or path to settings file
# settings = "model_settings.yaml"

m = MyModel(custom_inference_settings=settings)

if sly.is_production():
    # this code block is running on Supervisely platform in production
    # just ignore it during development
    m.serve()
else:
    images_path = Path("demo_images") / "racing"
    out = img_path / "predicted"
    out.mkdir(exist_ok=True, parents=True)

    # sort frames
    imgs_names = sorted(os.listdir(img_path))

    start, end = 80, 90
    imgs_pth = [img_path / name for name in imgs_names[start:end] if "jpg" in name]

    # top left bottom right order
    start_object = PredictionBBox("", [187, 244, 236, 365], None)

    # load frames
    images = [sly_image.read(str(pth)) for pth in imgs_pth]

    preds = []
    model.initialize(images[0], start_object)

    for image in tqdm(images[1:]):
        preds.append(model.predict(image, {}, images[0], start_object))
        start_object = preds[-1]

    model.visualize(
        preds,
        images[1:],
        out,
        thickness=5,
    )
```

Here are the output predictions of our simple model:

![Example of Simple model work](https://github-production-user-asset-6210df.s3.amazonaws.com/87002239/245512007-f6d593f8-458d-4e01-af74-ce4d279d281a.jpg)

## MixFormer tracking model

Let's now implement the class for a pre-trained model. The majority of the code used to load and run the model is taken directly from the original repository. We will also include the option to choose a model before launching the app because the authors provide two pre-trained models.

### Getting started

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone the [repository](https://github.com/supervisely-ecosystem/MixFormer) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone git@github.com:supervisely-ecosystem/MixFormer.git
cd MixFormer
source .venv/bin/activate
pip3 install torch==1.8.1+cu111 torchvision==0.9.1+cu111 -f https://download.pytorch.org/whl/torch_stable.html
pip3 install -r requirements.txt
```

**Step 3.** Load model weights.
```bash
mkdir -p save/models
wget --no-check-certificate 'https://github.com/supervisely-ecosystem/MixFormer/releases/download/v0.0.1-alpha/mixformer_vit_large_online.pth.tar' -O save/models/mixformer_vit_large_online.pth.tar
wget --no-check-certificate 'https://github.com/supervisely-ecosystem/MixFormer/releases/download/v0.0.1-alpha/mixformer_convmae_large_online.pth.tar' -O save/models/mixformer_convmae_large_online.pth.tar
```

**Step 4.** Open the repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 5.** Run debug for script `src/main.py`
## Python script

The integration script is simple:

1. Initialize model.
2. Run inference on demo images.
3. Collect predictions and save them in chronological order.

[//]: # "The entire integration Python script takes only 👍 **90 lines** of code (including comments) and can be found in [GitHub repository](https://github.com/supervisely-ecosystem/integrate-inst-seg-model) for this tutorial."


### Step-by-step implementation

**Defining imports and global variables**

```python
import os
import numpy as np
from dotenv import load_dotenv
from pathlib import Path
from typing import Any, Dict, Literal
from typing_extensions import Literal
from tqdm import tqdm

from lib.test.evaluation import create_default_local_file_ITP_test
from lib.train.admin import create_default_local_file_ITP_train

import sly_functions as F
import supervisely as sly
import supervisely.imaging.image as sly_image
from supervisely.nn.inference import BBoxTracking
from supervisely.nn.prediction_dto import PredictionBBox


# one of pre-trained model name
NAME = os.environ.get("modal.state.modelName", "mixformer_vit_online")
root = (Path(__file__).parent / ".." / ".." / "..").resolve().absolute()

load_dotenv(os.path.expanduser("~/supervisely.env"))
```

**1. load_on_device**

The following code creates the model which will keep as a `self.model`. The `SupportedModels` class is simply an `Enum` class that was built to prevent working with raw strings. The `Tracker` class is a collection of functions  supplied by the original repository's creator to create and use pre-trained models ([check implementation here](https://github.com/supervisely-ecosystem/MixFormer/blob/1a4f52db2a4ba8d8cacd18046e22d8f72be73163/serve/serve/src/sly_functions.py#LL18C9-L18C9))

```python
class MixFormer(BBoxTracking):
    def load_on_device(
        self,
        model_dir: str,
        device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
    ):
        name = F.SupportedModels.instance_by_name(NAME)
        self.model = F.Tracker(name)
```

{% hint style="info" %}
Here we are downloading the model weights from local storage, but it can be also downloaded by path in Supervisely **Team Files**.
{% endhint %}

**2. initialize and predict**

The core methods for model inference. Here we are preparing the initial bound box in initialization and wrapping model predictions into `sly.nn.PredictionBBox`.

```python
    def initialize(
        self, init_rgb_image: np.ndarray, target_bbox: PredictionBBox
    ) -> None:
        y1, x1, y2, x2 = target_bbox.bbox_tlbr
        w = abs(x2 - x1)
        h = abs(y2 - y1)
        self.model.initialize(init_rgb_image, x1, y1, w, h)

    def predict(
        self,
        rgb_image: np.ndarray,
        settings: Dict[str, Any],
        prev_rgb_image: np.ndarray,
        target_bbox: PredictionBBox,
    ) -> PredictionBBox:
        class_name = target_bbox.class_name
        x, y, w, h = self.model.track(rgb_image)
        tlbr = [int(y), int(x), int(y + h), int(x + w)]
        return PredictionBBox(class_name, tlbr, None)
```

It **must** return exactly a list of `sly.nn.PredictionBBox` objects for compatibility with Supervisely.

**Usage of our class**

Once the class is created, here we initialize it and get one test prediction for debugging.


```python
if sly.is_debug_with_sly_net() or not sly.is_production():
    # paths for local use
    create_default_local_file_ITP_test(str(root), "", str(root / "save"))
    create_default_local_file_ITP_train(str(root), "")
else:
    # paths inside docker cpntainer for production use
    create_default_local_file_ITP_test(str(root), "", "/weights")
    create_default_local_file_ITP_train(str(root), "")

mixformer = MixFormer()

if sly.is_production():
    mixformer.serve()
else:
    data_path = root / "data"
    img_path = data_path / "racing"
    out = img_path / "predicted"
    out.mkdir(exist_ok=True, parents=True)
    imgs_names = sorted(os.listdir(img_path))

    start, end = 80, 180
    imgs_pth = [img_path / name for name in imgs_names[start:end] if 'jpg' in name]

    # top left bottom right order
    start_object = PredictionBBox("", [187, 244, 236, 365], None)
    images = [sly_image.read(str(pth)) for pth in imgs_pth]

    preds = []
    mixformer.initialize(images[0], start_object)

    for image in tqdm(images[1:]):
        preds.append(mixformer.predict(image, {}, images[0], start_object))

    mixformer.visualize(
        preds,
        images[1:],
        out,
        thickness=5,
    )
```

Here are the output predictions of our MixFormer model:

![Example of MixFormer model work](https://github.com/supervisely/developer-portal/assets/87002239/bf103661-3c2c-45c9-98a7-229eb3f6d5ba)

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

3. Define your `TEAM_ID` in the `local.env` file. *Other env variables that are needed, are already provided in `./vscode/launch.json` for you.*

4. Switch the `launch.json` config to the `Advanced debug in Supervisely platform`:

![Advanced Debug in Supervisely](https://user-images.githubusercontent.com/31512713/224290229-5da93fd2-dc97-4911-abb5-66ce890485a2.png)

5. Run the code.

✅ It will deploy the model in the Supervisely platform as a regular serving App that can communicate with all other apps in the platform:

![Develop and Debug](https://user-images.githubusercontent.com/31512713/223178384-cf316096-fc23-4e32-80fc-4288bad415be.png)

{% hint style="success" %}
Now you can use apps like [Apply NN to Images](https://ecosystem.supervise.ly/apps/nn-image-labeling/project-dataset), [Apply NN to videos](https://ecosystem.supervise.ly/apps/apply-nn-to-videos-project) with your deployed model.

Or get the model inference via **Python API** with the help of `sly.nn.inference.Session` class just in one line of code. See [Inference API Tutorial](../../../app-development/neural-network-integration/inference-api-tutorial.md).
{% endhint %}


## Release your code as a Supervisely App.

Once you've tested the code, it's time to release it into the platform. It can be released as an App that is shared with the all Supervisely community, or as your own private App.

Refer to [How to Release your App](../../../app-development/basics/from-script-to-supervisely-app.md) for all releasing details. For a private app check also [Private App Tutorial](../../../app-development/basics/add-private-app.md).


### Repository structure

The structure of [our GitHub repository](https://github.com/supervisely-ecosystem/MixFormer) is the following:

```
.
├── LICENSE
├── README.md
├── app_data
│   └── models
├── data
│   └── racing
│       ├── 000001.jpg
│       ├── ...
│       └── 000260.jpg
├── experiments
│   └── # .yaml configs for models
├── external
│   └── # code for different datasets and utils connected to project
├── install_reqs.sh
├── lib
│   ├── __init__.py
│   ├── __pycache__
│   │   └── __init__.cpython-39.pyc
│   ├── config
│   │   └── # configs for different models
│   ├── models
│   │   ├── __init__.py
│   │   ├── mixformer_convmae
│   │   │   ├── __init__.py
│   │   │   ├── mixformer.py
│   │   │   └── mixformer_online.py
│   │   ├── mixformer_cvt
│   │   │   ├── __init__.py
│   │   │   ├── head.py
│   │   │   ├── mixformer.py
│   │   │   ├── mixformer_online.py
│   │   │   ├── score_decoder.py
│   │   │   └── utils.py
│   │   └── mixformer_vit
│   │       ├── __init__.py
│   │       ├── mixformer.py
│   │       ├── mixformer_online.py
│   │       └── pos_utils.py
│   ├── test
│   │   └── # scripts and utils for model testing on various datasets
│   ├── train
│   │   └── # scripts and utils for model training on various datasets
│   └── utils
│       ├── __init__.py
│       ├── box_ops.py
│       ├── classification_loss.py
│       ├── lmdb_utils.py
│       ├── lr_shed.py
│       ├── merge.py
│       ├── misc.py
│       └── tensor.py
├── save
|   └── models
|       ├── mixformer_convmae_large_online.pth.tar
│       └── mixformer_vit_large_online.pth.tar
├── requirements.txt
├── serve
│   ├── docker
│   │   └── Dockerfile
│   └── serve
│       ├── README.md
│       ├── config.json
│       ├── local.env
│       └── src
│           ├── main.py
│           ├── modal.html
│           └── sly_functions.py
└── tracking
    └── # some author scripts
```

Explanation:

* `serve/serve/src/main.py` - main inference script
* `serve/serve/src/sly_functions.py` - functions to run the MixFormer model based on the original repository code
* `serve/serve/src/modal.html` - modal window template; a simple way to control ENV variables (e.g. model type)
* `save/models` - directory with model weights
* `data/racing` - directory with demo frames for inference
* `serve/serve/README.md` - readme of your application, it is the main page of an application in Ecosystem with some images, videos, and how-to-use guides
* `serve/serve/config.json` - configuration of the Supervisely application, which defines the name and description of the app, its context menu, icon, poster, and running settings
* `requirements.txt` - all packages needed for debugging 
* `local.env` - file with variables used for debugging
* `serve/serve/docker` - directory with the custom Dockerfile for this application and the script that builds it and publishes it to the docker registry

### modal.html

The author of the original repository provides us with different models. We create a basic html file with a selector so that users may choose a model before launching the application. It is now sufficient to correctly specify the configuration, and the model name will be accessible via environment variables.

```python
NAME = os.environ.get("modal.state.modelName", "mixformer_vit_online")
```

```html
<div>
    <sly-field title="Model name"
               description="Select one of the models">
        <el-select v-model="state.modelName" placeholder="Select">
            <el-option key="vit" label="MixViT-Large" value="mixformer_vit_online"></el-option>
            <el-option key="convmae" label="MixViT-L (ConvMAE)" value="mixformer_convmae_online"></el-option>
        </el-select>
    </sly-field>
</div>
```

![Modal selector](https://github.com/supervisely/developer-portal/assets/87002239/ed5435b9-b1f7-43fa-a3c0-1eac5ad46bd6)

### App configuration

App configuration is stored in `config.json` file. A detailed explanation of all possible fields is covered in this [Configuration Tutorial](../../../app-development/basics/app-json-config/config.json.md). Let's check the config for our current app: 

```json
{
    "name": "MixFormer object tracking",
    "type": "app",
    "version": "2.0.0",
    "categories": [
      "neural network",
      "videos",
      "detection & tracking",
      "serve"
    ],
    "description": "serve and use in videos annotator",
    "docker_image": "supervisely/mixformer:1.0.2",
    "entrypoint": "python -m uvicorn main:mixformer.app --app-dir ./serve/serve/src --host 0.0.0.0 --port 8000 --ws websockets",
    "port": 8000,
    "modal_template": "serve/serve/src/modal.html",
    "modal_template_state": {
      "modelName": "mixformer_vit_online"
    },
    "task_location": "application_sessions",
    "isolate": true,
    "headless": true,
    "need_gpu": true,
    "instance_version": "6.7.40",
    "restart_policy": "on_error",
    "session_tags": [
      "sly_video_tracking"
    ],
    "community_agent": false,
    "allowed_shapes": [
      "rectangle"
    ]
  }
```

Here is the explanation for the fields:

* `type` - a type of the module in Supervisely Ecosystem
* `version` - version of Supervisely App Engine. Just keep it by default
* `name` - the name of the application
* `description` - the description of the application
* `categories` - these tags are used to place the application in the correct category in Ecosystem.
* `session_tags` - these tags will be assigned to every running session of the application. They can be used by other apps to find and filter all running sessions
* `modal_template` - path to modal window template (`modal.html` file previously described)
* `modal_template_state` - list of default values for all states
* `"need_gpu": true` - should be true if you want to use any `cuda` devices.
* `"community_agent": false` - this means that this app can not be run on the agents started by Supervisely team, so users have to connect their computers and run the app only on their agents. Only applicable in Community Edition. Enterprise customers use their private instances so they can ignore the current option
* `docker_image` - Docker container will be started from the defined Docker image, GitHub repository will be downloaded and mounted inside the container.
* `entrypoint` - the command that starts our application in a container
* `port` - port inside the container
* `"headless": true` means that the app has no User Interface
* `allowed_shapes` - shapes can be tracked with this model
