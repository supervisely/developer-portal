---
description: >-
  Step-by-step tutorial on how to integrate custom point tracking
  neural network into Supervisely platform on the example of PIPs.
---

# Integrate point tracking

## Introduction

In this tutorial you will learn how to integrate your custom point tracking model into Supervisely by creating two simple serving apps. 
First, we will construct a straightforward model that only moves the original point as an illustration. The SOTA model [PIPs](https://github.com/aharley/pips), which already has the majority of the necessary functions implemented, will be used in the second part.

## Implementation details

To integrate your model, you need to subclass **`sly.nn.inference.PointTracking`** and implement 2 methods:

* `load_on_device` method for downloading the weights and initializing the model on a specific device. Takes a `model_dir` argument, which is a directory for all model files (like configs, weights, etc). The second argument is a `device` - a torch.device like `cuda:0`, `cpu`.
* `predict`. The core implementation of model inference. It takes a list of images of `np.ndarray` type, inference settings and point to track as arguments, applies the model inference to the images and returns a list of predictions (both input point and predicted points are `sly.nn.PredictionPoint` objects).

Currently, integrating models that can track several points simultaneously is not possible due to the implementation of the `sly.nn.inference.PointTracking` class.

### Overall structure

The overall structure of the class we will implement is looking like this:

```python
class MyModel(sly.nn.inference.PointTracking):
    def load_on_device(
        self,
        model_dir: str,
        device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
    ):
        # preparing the model: model instantiating, downloading weights, loading it on the device.
        pass

    def predict(
        self,
        rgb_images: List[np.ndarray],
        settings: Dict[str, Any],
        start_object: PredictionPoint,
    ) -> List[PredictionPoint]:
        # the inference of a model here
        # ...
        return prediction
```

The superclass has a `serve()` method. For running the code on the Supervisely platform, `m.serve()` method should be executed:

```python
if sly.is_production():
    m.serve()
```

And here is the beauty comes in. The method `serve()` internally handles everything and deploys your model as a **REST API** service on the Supervisely platform. It means that other applications are able to communicate with your model and get predictions from it.

So let's implement the class.

# Simple model

## Getting started

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.**
Create [Virtual Environment](https://docs.python.org/3/library/venv.html) and install `supervisely==6.72.9` in it.

## Step-by-step implementation

**Defining imports and global variables**

```python
import numpy as np
import torch
from dotenv import load_dotenv
from pathlib import Path
from typing import Any, Dict, List, Literal
from typing_extensions import Literal

import supervisely as sly
from supervisely.nn.inference import PointTracking
from supervisely.nn.prediction_dto import PredictionPoint


load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
```

**1. load_on_device**

The following code creates the model according to config `model_settings.yaml`. Path to `.yaml` config is passed during initialization. Config in the form of a dictionary becomes available in `self.custom_inference_settings_dict` attribute. Also `load_on_device` will keep the model as a `self.model` for further use:

```python
class MyModel(PointTracking):
    def load_on_device(
        self,
        model_dir: str,
        device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
    ):
        col_shift = self.custom_inference_settings_dict.get("col_shift", 10)
        row_shift = self.custom_inference_settings_dict.get("row_shift", 7)

        self.model = lambda point, name: PredictionPoint(
            class_name=name,
            col=point.col + col_shift,
            row=point.row + row_shift,
        )
```

Our `settings.yaml` file:

```yaml
col_shift: 10
row_shift: 7
```

**2. predict**

The core method for model inference. Here we will use the defined model and make sure that predicted points are not outside of the bounds. 

```python
    def predict(
        self,
        rgb_images: List[np.ndarray],
        settings: Dict[str, Any],
        start_object: PredictionPoint,
    ) -> List[PredictionPoint]:
        name = start_object.class_name
        pred_points = []
        frame_range = len(rgb_images)
        point = strat_object

        maxh, maxw = rgbs_images[0].shape

        for _ in range(frame_range):
            # predict next point
            new_point = self.model(point, name)

            # check bounds
            new_point.col = min(new_point.col, maxh - 1)
            new_point.row = min(new_point.row, maxw - 1)
            pred_points.append(new_point)

            # next point
            point = new_point

        return pred_points
```

It **must** return exactly a list of `sly.nn.PredictionPoint` objects for compatibility with Supervisely. **Notice, that the first frame is not in the list.**

**Usage of our class**

Once the class is created, here we initialize it and get one test prediction for debugging.


```python
settings = "model_settings.yaml"

m = MyModel(model_dir="", custom_inference_settings=settings)

if sly.is_production():
    # this code block is running on Supervisely platform in production
    # just ignore it during development
    m.serve()
else:
    # for local debugging
    settings = m.custom_inference_settings_dict
    fake_frames = [None for _ in range(10)]
    start_point = PredictionPoint("name", col=0, row=0)
    pred_points = m.predict(fake_frames, settings, start_point)
```

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

✅ It will deploy the model in the Supervisely platform as a regular serving App that is able to communicate with all others apps in the platform:

![Develop and Debug](https://user-images.githubusercontent.com/31512713/223178384-cf316096-fc23-4e32-80fc-4288bad415be.png)

{% hint style="success" %}
Now you can use apps like [Apply NN to Images](https://ecosystem.supervise.ly/apps/nn-image-labeling/project-dataset), [Apply NN to videos](https://ecosystem.supervise.ly/apps/apply-nn-to-videos-project) with your deployed model.

Or get the model inference via **Python API** with the help of `sly.nn.inference.Session` class just in one line of code. See [Inference API Tutorial](../../../app-development/neural-network-integration/inference-api-tutorial.md).
{% endhint %}


## Release your code as a Supervisely App.

Once you've tested the code, it's time to release it into the platform. It can be released as an App that is shared with the all Supervisely community, or as your own private App.

Refer to [How to Release your App](../../../app-development/basics/from-script-to-supervisely-app.md) for all releasing details. For a private app check also [Private App Tutorial](../../../app-development/basics/add-private-app.md).


# PIPs tracking model

Let's now implement the class for pre-trained model. The majority of the code used to load and run the model is taken directly from the original repository.

## Getting started

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/pips) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/pips
cd pips
source .venv/bin/activate
pip3 install -r requirements.txt
```

It's feasible to run the present model on the `CPU`, thus installing `CUDA` requirements is not required.

**Step 3.** Load model weights.
```bash
./get_reference_model.sh
```

**Step 4.** Open the repository directory in Visual Studio Code.

```bash
code -r .
```

[//]: # "Debug part need to be created"

[//]: # "**Step 5.** Run debug for script `src/main.py`"

[//]: # "## Python script"

[//]: # "The integration script is simple:"

[//]: # "1. Automatically downloads NN weights to `./my_model` folder"
[//]: # "2. Loads model on the CPU or GPU device"
[//]: # "3. Runs inference on a demo image"
[//]: # "4. Visualizes predictions on top of the input image"

[//]: # "The entire integration Python script takes only 👍 **90 lines** of code (including comments) and can be found in [GitHub repository](https://github.com/supervisely-ecosystem/integrate-inst-seg-model) for this tutorial."


### Step-by-step implementation

**Defining imports and global variables**

```python
import os
import numpy as np
import saverloader
import torch
from dotenv import load_dotenv
from pathlib import Path
from typing import Any, Dict, List, Literal
from typing_extensions import Literal
from nets.pips import Pips

import sly_functions
import supervisely as sly
from supervisely.nn.inference import PointTracking
from supervisely.nn.prediction_dto import PredictionPoint


root = (Path(__file__).parent / ".." / ".." / "..").resolve().absolute()

load_dotenv(root / "local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
```

**1. load_on_device**

The following code creates the model according to config `supervisely/serve/model_settings.yaml`. Path to `.yaml` config is passed during initialization. The `saverloader.load` function provided by the creator of the original repository loads the model state dict from `model_dir`. Also `load_on_device` will keep the model as a `self.model` and the device as `self.device` for further use:

```python
class PipsTracker(PointTracking):
    def load_on_device(
        self,
        model_dir: str,
        device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
    ):
        frames_per_iter = self.custom_inference_settings_dict.get("frames_per_iter", 8)
        stride = self.custom_inference_settings_dict.get("stride", 4)

        self.model = Pips(S=frames_per_iter, stride=stride).to(torch.device(device))
        if model_dir:
            _ = saverloader.load(str(model_dir), self.model, device=device)
        self.model.eval()
        self.device = device
```

{% hint style="info" %}
Here we are downloading the model weights from local storage, but it can be also downloaded by path in Supervisely **Team Files**.
{% endhint %}

**2. predict**

The core method for model inference. Here we are preparing images and getting an inference of the model. The function `sly_functions.run_model` is borrowed from the original repository. However, there are a few changes that can be made to improve quality: preserve the aspect ratio, apply padding before resizing and make sure that predicted points are not outside of the bounds. Then we wrap model predictions into `sly.nn.PredictionPoint`.

```python
    def predict(
        self,
        rgb_images: List[np.ndarray],
        settings: Dict[str, Any],
        start_object: PredictionPoint,
    ) -> List[PredictionPoint]:
        class_name = start_object.class_name
        h_resized = settings.get("h_resized", 360)
        w_resized = settings.get("w_resized", 640)
        frames_per_iter = settings.get("frames_per_iter", 8)

        rgbs = [torch.from_numpy(rgb_img).permute(2, 0, 1) for rgb_img in rgb_images]
        rgbs = torch.stack(rgbs, dim=0).unsqueeze(0)
        point = torch.tensor([[start_object.col, start_object.row]], dtype=float)

        with torch.no_grad():
            traj = sly_functions.run_model(
                self.model,
                rgbs,
                point,
                (h_resized, w_resized),
                frames_per_iter,
                device=self.device,
            )

        pred_points = [PredictionPoint(class_name, col=p[0], row=p[1]) for p in traj[1:]]
        return pred_points
```

It **must** return exactly a list of `sly.nn.PredictionPoint` objects for compatibility with Supervisely.

**Usage of our class**

Once the class is created, here we initialize it and get one test prediction for debugging.

{% hint style="info" %}
In the code below a `custom_inference_settings` is used. It allows us to provide custom settings that could be used in `predict()` (See more in [Customized Inference Tutorial](../../../app-development/neural-network-integration/inference/customize-inference.md))
{% endhint %}

```python
settings = root / "supervisely" / "serve" / "model_settings.yaml"

if sly.is_debug_with_sly_net():
    model_dir = root / "reference_model"  # local debug
else:
    model_dir = Path("/weights")  # path in Docker

pips = PipsTracker(model_dir=str(model_dir), custom_inference_settings=str(settings))

if sly.is_production():
    # this code block is running on Supervisely platform in production
    # just ignore it during development
    pips.serve()

```

### Repository structure

The structure of [our GitHub repository](https://github.com/supervisely-ecosystem/pips) is the following:

```
.
├── LICENSE
├── README.md
├── badjadataset.py
├── chain_demo.py
├── crohddataset.py
├── demo.py
├── demo_images
│   ├── 000100.jpg
|   |   ...
│   └── extract_frames.sh
├── filter_trajs.py
├── flyingthingsdataset.py
├── get_reference_model.sh
├── local.env
├── make_occlusions.py
├── make_trajs.py
├── nets
│   ├── pips.py
│   ├── raft_core
│   │   ├── __init__.py
│   │   ├── corr.py
│   │   ├── datasets.py
│   │   ├── extractor.py
│   │   ├── raft.py
│   │   ├── update.py
│   │   └── util.py
│   └── raftnet.py
├── reference_model
│   └── model-000200000.pth
├── requirements.txt
├── saverloader.py
├── supervisely
│   ├── docker
│   │   └── Dockerfile
│   └── serve
│       ├── README.md
│       ├── config.json
│       ├── local.env
│       ├── model_settings.yaml
│       └── src
│           ├── main.py
│           └── sly_functions.py
├── test_on_badja.py
├── test_on_crohd.py
├── test_on_davis.py
├── test_on_flt.py
├── train.py
└── utils
    ├── basic.py
    ├── bremm.png
    ├── improc.py
    ├── misc.py
    ├── samp.py
    └── test.py
```

Explanation:

* `supervisely/serve/src/main.py` - main inference script
* `supervisely/serve/src/sly_functions.py` - functions to run the PIPs model based on the original repository code
* `reference_model` - directory with model weights; will be created automatically in `get_reference_model.sh`
* `demo_images` - directory with demo frames for inference
* `supervisely/serve/README.md` - readme of your application, it is the main page of an application in Ecosystem with some images, videos, and how-to-use guides
* `supervisely/serve/config.json` - configuration of the Supervisely application, which defines the name and description of the app, its context menu, icon, poster, and running settings
* `requirements.txt` - all packages needed for debugging 
* `local.env` - file with variables used for debugging
* `supervisely/serve/docker` - directory with the custom Dockerfile for this application and the script that builds it and publishes it to the docker registry


### App configuration

App configuration is stored in `config.json` file. A detailed explanation of all possible fields is covered in this [Configuration Tutorial](../../../app-development/basics/app-json-config/config.json.md). Let's check the config for our current app: 

```json
{
  "name": "PIPs object tracking",
  "type": "app",
  "version": "2.0.0",
  "poster": "https://user-images.githubusercontent.com/115161827/233959102-9c48949f-b353-4a4b-ab7d-c1da99dfd914.jpg",
  "icon_cover": true,
  "icon": "https://user-images.githubusercontent.com/115161827/233959116-9d0922c6-e6fc-4f3a-958d-430742533f3a.jpg",
  "categories": [
    "neural network",
    "videos",
    "detection & tracking",
    "serve"
  ],
  "description": "serve and use in videos annotator",
  "docker_image": "supervisely/pips:1.0.0",
  "entrypoint": "python -m uvicorn main:pips.app --app-dir ./supervisely/serve/src --host 0.0.0.0 --port 8000 --ws websockets",
  "port": 8000,
  "task_location": "application_sessions",
  "headless": true,
  "need_gpu": true,
  "restart_policy": "on_error",
  "session_tags": [
    "sly_video_tracking"
  ],
  "community_agent": false,
  "allowed_shapes": [
    "point",
    "polygon",
    "graph"
  ]
}
```

Here is the explanation for the fields:

* `type` - type of the module in Supervisely Ecosystem
* `version` - version of Supervisely App Engine. Just keep it by default
* `name` - the name of the application
* `description` - the description of the application
* `categories` - these tags are used to place the application in the correct category in Ecosystem.
* `session_tags` - these tags will be assigned to every running session of the application. They can be used by other apps to find and filter all running sessions
* `"need_gpu": true` - should be true if you want to use any `cuda` devices.
* `"community_agent": false` - this means that this app can not be run on the agents started by Supervisely team, so users have to connect their own computers and run the app only on their own agents. Only applicable in Community Edition. Enterprise customers use their private instances so they can ignore the current option
* `docker_image` - Docker container will be started from the defined Docker image, github repository will be downloaded and mounted inside the container.
* `entrypoint` - the command that starts our application in a container
* `port` - port inside the container
* `"headless": true` means that the app has no User Interface
* `allowed_shapes` - shapes can be tracked with this model. Сonversion of figures to a set of points and vice versa is implemented in the base class, so you can keep this field default.
