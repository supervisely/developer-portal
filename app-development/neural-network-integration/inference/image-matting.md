---
description: >-
  Step-by-step tutorial on how to integrate custom interactive image matting
  neural network into Supervisely platform on the example of Matte Anything.
---

# Custom interactive image matting model integration

## Introduction

In this tutorial you will learn how to integrate custom interactive image matting model into Supervisely Ecosystem. Supervisely Python SDK allows to easily integrate models for numerous image processing tasks. This tutorial takes [Matte Anything](https://github.com/hustvl/Matte-Anything/tree/main?tab=readme-ov-file) image matting model (to be more precise, it is a combination of several models united into a single pipeline) as an example and provides a complete instruction to integrate it as an application into Supervisely Ecosystem. You can find and try Matte Anything Supervisely integration [here](https://ecosystem.supervisely.com/apps/serve-matte-anything/serving_app). The code for integration can be found [here](https://github.com/supervisely-ecosystem/Serve-Matte-Anything/tree/master).

## Implementation details

To integrate your custom video object segmentation model, you need to subclass **`sly.nn.inference.PromptableSegmentation`** and implement 4 main methods:

* `initialize_custom_gui` method for building custom GUI for your model;
* `get_params_from_gui` method for getting necessary parameters for model deployment from GUI;
* `load_model` method for loading model on device (CPU / GPU);
* `serve` for serving model as REST API on Supervisely platform - it means that other applications are able to send requests to your model and get predictions from it.

### Overall structure

The overall structure of the class we will implement looks like this:

```python
import supervisely as sly
from supervisely.app.widgets import *
from fastapi import Response, Request


class MyModel(sly.nn.inference.PromptableSegmentation):
    def initialize_custom_gui(self):
        # build custom UI from supervisely widgets, put them into Container and return it
        custom_gui = Container(
            widgets=list_of_widgets,
        )
        return custom_gui

    def get_params_from_gui(self):
        # extract parameters which will be used in load_model method from GUI and return them as a dictionary
        return deploy_params

    def load_model(self):
        # initialize model architecture, load weights and put model on device
        pass

    def serve(self):
        super().serve()
        server = self._app.get_server()

        @server.post("/smart_segmentation")
        def smart_segmentation(response: Response, request: Request):
            pass


model = MyModel()
model.serve()
```

As you can see from the code snippet above, it will be necessary to add `smart_segmentation` endpoint in `serve` method - it is necessary to enable model to take requests on segmentation from Supervisely image labeling tool.

## Matte Anything interactive image matting model

Now let's implement the class specifically for Matte Anything.

### Installing necessary packages

We recommend to develop apps using VS Code [Dev Containers](https://code.visualstudio.com/docs/devcontainers/tutorial) extension - it will simplify installation of necessary packages.

Here is a Dockerfile for Serve Matte Anything app development:

```docker
FROM nvidia/cuda:11.8.0-cudnn8-devel-ubuntu20.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt update && apt install python3-pip -y
RUN apt-get install -y git

ARG USE_CUDA=0

ENV DEBIAN_FRONTEND=noninteractive
ENV AM_I_DOCKER True
ENV BUILD_WITH_CUDA "${USE_CUDA}"
ENV TORCH_CUDA_ARCH_LIST "8.9"
ENV CUDA_HOME /usr/local/cuda-11.8

RUN pip3 install networkx==2.8.8
RUN pip3 install torch==2.2.0 torchvision==0.17.0 torchaudio==2.2.0 --index-url https://download.pytorch.org/whl/cu118

RUN apt-get install ffmpeg libgeos-dev libsm6 libxext6 libexiv2-dev libxrender-dev libboost-all-dev -y

RUN git clone https://github.com/hustvl/Matte-Anything.git
RUN pip3 install git+https://github.com/facebookresearch/segment-anything.git
RUN python3 -m pip install 'git+https://github.com/facebookresearch/detectron2.git'

WORKDIR ./Matte-Anything
RUN git clone https://github.com/IDEA-Research/GroundingDINO.git
RUN python3 -m pip install --no-cache-dir wheel
RUN python3 -m pip install --no-cache-dir --no-build-isolation -e GroundingDINO

RUN pip3 install opencv-python==4.8.0.74
RUN pip3 install gradio==3.41.2
RUN pip3 install fairscale

RUN python3 -m pip install supervisely==6.73.82
RUN pip3 install urllib3==1.26.17
RUN pip3 install einops==0.8.0

RUN apt-get -y install curl
RUN apt -y install wireguard iproute2
RUN apt-get -y install wget
RUN apt-get install nano
```

### Downloading weights of models

After installing all necessary packages, it will be also necessary to download weights of models (Matte Anything uses [Segment Anything](https://github.com/facebookresearch/segment-anything), [Groundin DINO](https://github.com/IDEA-Research/GroundingDINO) and [ViTMatte](https://github.com/hustvl/ViTMatte/tree/main)) and put them into `pretrained` folder. Here are the links for [Segment Anything](https://github.com/facebookresearch/segment-anything?tab=readme-ov-file#model-checkpoints), [Grounding DINO](https://github.com/IDEA-Research/GroundingDINO?tab=readme-ov-file#luggage-checkpoints) and [ViTMatte](https://github.com/hustvl/ViTMatte/tree/main?tab=readme-ov-file#results) pretrained checkpoints. For local debug we can load model weights from local storage, but in production we recommend to save weights to a Docker image.

### Preparing json files with models data

We will need to create a json file for each model group - this data will be used to create model tables in UI and load models in code.

Here are json files for Segment Anything, Grounding DINO and ViTMatte respectively:

```json
[
    {
        "Model": "ViT-B SAM",
        "Number of parameters": "91M",
        "Size": "375 MB",
        "mIoU": "69",
        "weights": "/pretrained/sam_vit_b.pth"
    },
    {
        "Model": "ViT-L SAM",
        "Number of parameters": "308M",
        "Size": "1.25 GB",
        "mIoU": "72",
        "weights": "/pretrained/sam_vit_l.pth"
    },
    {
        "Model": "ViT-H SAM",
        "Number of parameters": "636M",
        "Size": "2.56 GB",
        "mIoU": "74",
        "weights": "/pretrained/sam_vit_h.pth"
    }
]
```

```json
[
    {
        "Model": "GroundingDINO-T",
        "backbone": "Swin-T",
        "Datasets": "O365, GoldG, Cap4M",
        "box AP on COCO": "57.2",
        "config": "./GroundingDINO/groundingdino/config/GroundingDINO_SwinT_OGC.py",
        "weights": "/pretrained/groundingdino_swint_ogc.pth"
    },
    {
        "Model": "GroundingDINO-B",
        "backbone": "Swin-B",
        "Datasets": "COCO, O365, GoldG, Cap4M, OpenImage, ODinW-35, RefCOCO",
        "box AP on COCO": "56.7",
        "config": "./GroundingDINO/groundingdino/config/GroundingDINO_SwinB_cogcoor.py",
        "weights": "/pretrained/groundingdino_swinb_cfg.pth"
    }
]
```

```json
[
    {
        "Model": "ViTMatte-S (Composition-1k)",
        "SAD": "21.46",
        "MSE": "3.3",
        "Grad": "7.24",
        "Conn": "16.21",
        "weights": "/pretrained/vitmatte_s_com.pth",
        "config": "./configs/vitmatte_s.py"
    },
    {
        "Model": "ViTMatte-S (Distinctionctions-646)",
        "SAD": "21.22",
        "MSE": "2.1",
        "Grad": "8.78",
        "Conn": "17.55",
        "weights": "/pretrained/vitmatte_s_dis.pth",
        "config": "./configs/vitmatte_s.py"
    },
    {
        "Model": "ViTMatte-B (Composition-1k)",
        "SAD": "20.33",
        "MSE": "3.0",
        "Grad": "6.74",
        "Conn": "14.78",
        "weights": "/pretrained/vitmatte_b_com.pth",
        "config": "./configs/vitmatte_b.py"
    },
    {
        "Model": "ViTMatte-B (Distinctionctions-646)",
        "SAD": "17.05",
        "MSE": "1.5",
        "Grad": "7.03",
        "Conn": "12.95",
        "weights": "/pretrained/vitmatte_b_dis.pth",
        "config": "./configs/vitmatte_b.py"
    }
]
```

We will put these files into `models_data` directory.

### Step-by-step class implementation

**Defining imports and global variables**

```python
import os
import cv2
import torch
import numpy as np
from PIL import Image
from torchvision.ops import box_convert
from torchvision.transforms import functional as F
from detectron2.config import LazyConfig, instantiate
from detectron2.checkpoint import DetectionCheckpointer
from segment_anything import sam_model_registry, SamPredictor
import groundingdino.datasets.transforms as T
from groundingdino.util.inference import (
    load_model as dino_load_model,
    predict as dino_predict,
)
import supervisely as sly
from typing import Literal
from typing import List, Any, Dict
import threading
from cachetools import LRUCache
from cacheout import Cache
from supervisely.sly_logger import logger
from supervisely.nn.inference.interactive_segmentation import functional
from supervisely.app.content import get_data_dir
from supervisely.imaging import image as sly_image
from supervisely._utils import rand_str
from fastapi import Response, Request, status
import time
import base64
from PIL import Image
from dotenv import load_dotenv
from supervisely.app.widgets import (
    RadioTable,
    Field,
    Checkbox,
    Input,
    InputNumber,
    Container,
    Empty,
)


load_dotenv("local.env")
load_dotenv("supervisely.env")
is_debug_session = bool(os.environ.get("IS_DEBUG_SESSION", False))

original_dir = os.getcwd()
```

**initialize_custom_gui**

The following code builds custom GUI from Supervisely [widgets](https://developer.supervisely.com/app-development/widgets) (we will also need `get_models` method in order to read json files with models data and preprocess extracted data):

```python
def get_models(self):
    model_types = ["Segment Anything", "Grounding DINO", "ViTMatte"]
    self.models_dict = {}
    for model_type in model_types:
        if model_type == "Segment Anything":
            model_data_path = "./models_data/segment_anything.json"
        elif model_type == "Grounding DINO":
            model_data_path = "./models_data/grounding_dino.json"
        elif model_type == "ViTMatte":
            model_data_path = "./models_data/vitmatte.json"
        model_data = sly.json.load_json_file(model_data_path)
        self.models_dict[model_type] = model_data
    return self.models_dict

def initialize_custom_gui(self):
    models_data = self.get_models()

    def remove_unnecessary_keys(data_dict):
        new_dict = data_dict.copy()
        new_dict.pop("weights", None)
        new_dict.pop("config", None)
        return new_dict

    sam_model_data = models_data["Segment Anything"]
    sam_model_data = [remove_unnecessary_keys(d) for d in sam_model_data]
    gr_dino_model_data = models_data["Grounding DINO"]
    gr_dino_model_data = [remove_unnecessary_keys(d) for d in gr_dino_model_data]
    vitmatte_model_data = models_data["ViTMatte"]
    vitmatte_model_data = [remove_unnecessary_keys(d) for d in vitmatte_model_data]
    self.sam_table = RadioTable(
        columns=list(sam_model_data[0].keys()),
        rows=[list(element.values()) for element in sam_model_data],
    )
    self.sam_table.select_row(2)
    sam_table_f = Field(
        content=self.sam_table,
        title="Pretrained Segment Anything models",
    )
    self.vitmatte_table = RadioTable(
        columns=list(vitmatte_model_data[0].keys()),
        rows=[list(element.values()) for element in vitmatte_model_data],
    )
    self.vitmatte_table.select_row(3)
    vitmatte_table_f = Field(
        content=self.vitmatte_table,
        title="Pretrained ViTMatte models",
    )
    self.erode_input = InputNumber(value=20, min=1, max=30, step=1)
    erode_input_f = Field(
        content=self.erode_input,
        title="Erode kernel size",
    )
    self.dilate_input = InputNumber(value=20, min=1, max=30, step=1)
    dilate_input_f = Field(
        content=self.dilate_input,
        title="Dilate kernel size",
    )
    erode_dilate_inputs = Container(
        widgets=[erode_input_f, dilate_input_f, Empty()],
        direction="horizontal",
        fractions=[1, 1, 2],
    )
    self.gr_dino_checkbox = Checkbox(content="use Grounding DINO", checked=False)
    gr_dino_checkbox_f = Field(
        content=self.gr_dino_checkbox,
        title="Choose whether to use Grounding DINO or not",
        description=(
            "If selected, then Grounding DINO will be used to detect transparent objects on images "
            "and correct trimap based on detected objects"
        ),
    )
    self.gr_dino_table = RadioTable(
        columns=list(gr_dino_model_data[0].keys()),
        rows=[list(element.values()) for element in gr_dino_model_data],
    )
    gr_dino_table_f = Field(
        content=self.gr_dino_table,
        title="Pretrained Grounding DINO models",
    )
    gr_dino_table_f.hide()
    self.dino_text_prompt = Input(
        "glass, lens, crystal, diamond, bubble, bulb, web, grid"
    )
    self.dino_text_prompt.hide()
    dino_text_input_f = Field(
        content=self.dino_text_prompt,
        title="Text prompt for detecting transparent objects using Grounding DINO",
    )
    dino_text_input_f.hide()
    self.dino_text_thresh_input = InputNumber(
        value=0.25, min=0.1, max=0.9, step=0.05
    )
    dino_text_thresh_input_f = Field(
        content=self.dino_text_thresh_input,
        title="Grounding DINO text confindence threshold",
    )
    self.dino_box_thresh_input = InputNumber(value=0.5, min=0.1, max=0.9, step=0.1)
    dino_box_thresh_input_f = Field(
        content=self.dino_box_thresh_input,
        title="Grounding DINO box confindence threshold",
    )
    dino_thresh_inputs = Container(
        widgets=[dino_text_thresh_input_f, dino_box_thresh_input_f, Empty()],
        direction="horizontal",
        fractions=[1, 1, 2],
    )
    dino_thresh_inputs.hide()

    @self.gr_dino_checkbox.value_changed
    def change_dino_ui(value):
        if value:
            gr_dino_table_f.show()
            self.dino_text_prompt.show()
            dino_text_input_f.show()
            dino_thresh_inputs.show()
        else:
            gr_dino_table_f.hide()
            self.dino_text_prompt.hide()
            dino_text_input_f.hide()
            dino_thresh_inputs.hide()

    custom_gui = Container(
        widgets=[
            sam_table_f,
            vitmatte_table_f,
            erode_dilate_inputs,
            gr_dino_checkbox_f,
            gr_dino_table_f,
            dino_text_input_f,
            dino_thresh_inputs,
        ],
        gap=25,
    )
    return custom_gui
```

As you can see from the code above, there are two functions inside `initialize_custom_gui` method: `remove_unnecessary_keys` and `change_dino_ui`. The first one is used in order to remove config and checkpoint paths from model tables since we do not want this data to be displayed in UI, we will need this data only in our code. The second one is used for interaction witb user: if user chooses to use Grounding DINO, then part of UI with Grounding DINO settings will appear.

**load_model**

The code below initializes Segment Anything, ViTMatte and (if necessary) Grounding DINO models, loads their checkpoints and puts them on device:

```python
def init_segment_anything(self, model_type, checkpoint_path):
    sam = sam_model_registry[model_type](checkpoint=checkpoint_path).to(self.device)
    predictor = SamPredictor(sam)
    return predictor

def init_vitmatte(self, config_path, checkpoint_path):
    cfg = LazyConfig.load(config_path)
    vitmatte = instantiate(cfg.model)
    vitmatte.to(self.device)
    vitmatte.eval()
    DetectionCheckpointer(vitmatte).load(checkpoint_path)
    return vitmatte

def load_model(
    self,
    device: Literal["cpu", "cuda", "cuda:0", "cuda:1", "cuda:2", "cuda:3"] = "cpu",
):
    os.chdir(original_dir)
    # load segment anything
    sam_row_index = self.sam_table.get_selected_row_index()
    sam_dict = self.models_dict["Segment Anything"][sam_row_index]
    sam_model = sam_dict["Model"].lower()[:5].replace("-", "_")
    sam_checkpoint_path = sam_dict["weights"]
    if is_debug_session:
        sam_checkpoint_path = "." + sam_checkpoint_path
    self.predictor = self.init_segment_anything(sam_model, sam_checkpoint_path)
    # load vitmatte
    vitmatte_row_index = self.vitmatte_table.get_selected_row_index()
    vitmatte_dict = self.models_dict["ViTMatte"][vitmatte_row_index]
    vitmatte_config_path = vitmatte_dict["config"]
    vitmatte_checkpoint_path = vitmatte_dict["weights"]
    if is_debug_session:
        vitmatte_checkpoint_path = "." + vitmatte_checkpoint_path
    self.is_vitmatte = True
    self.vitmatte = self.init_vitmatte(
        vitmatte_config_path, vitmatte_checkpoint_path
    )
    # load grounding dino if necessary
    if self.gr_dino_checkbox.is_checked():
        grounding_dino_row_index = self.gr_dino_table.get_selected_row_index()
        gr_dino_dict = self.models_dict["Grounding DINO"][grounding_dino_row_index]
        gr_dino_config_path = gr_dino_dict["config"]
        gr_dino_checkpoint_path = gr_dino_dict["weights"]
        if is_debug_session:
            gr_dino_checkpoint_path = "." + gr_dino_checkpoint_path
        self.grounding_dino = dino_load_model(
            gr_dino_config_path, gr_dino_checkpoint_path
        )
    # define list of class names
    self.class_names = ["alpha_mask"]
    # variable for storing image ids from previous inference iterations
    self.previous_image_id = None
    # dict for storing model variables to avoid unnecessary calculations
    self.cache = Cache(maxsize=100, ttl=5 * 60)
```

We will also need some additional methods for serving our model on Supervisely platform:

```python
def get_info(self):
    info = super().get_info()
    info["videos_support"] = False
    info["async_video_inference_support"] = False
    return info

def get_classes(self) -> List[str]:
    return self.class_names

@property
def model_meta(self):
    if self._model_meta is None:
        self._model_meta = sly.ProjectMeta(
            [sly.ObjClass(self.class_names[0], sly.Bitmap, [255, 0, 0])]
        )
        self._get_confidence_tag_meta()
    return self._model_meta
```

After we have initialized necessary models, we can start implementing `serve` method. But before doing it, we will have to implement some methods which we will use to process image data and generate trimaps artificially:

```python
def generate_trimap(self, mask, erode_kernel_size=10, dilate_kernel_size=10):
    erode_kernel = np.ones((erode_kernel_size, erode_kernel_size), np.uint8)
    dilate_kernel = np.ones((dilate_kernel_size, dilate_kernel_size), np.uint8)
    eroded = cv2.erode(mask, erode_kernel, iterations=5)
    dilated = cv2.dilate(mask, dilate_kernel, iterations=5)
    trimap = np.zeros_like(mask)
    trimap[dilated == 255] = 128
    trimap[eroded == 255] = 255
    return trimap

def convert_pixels(self, gray_image, boxes):
    converted_image = np.copy(gray_image)

    for box in boxes:
        x1, y1, x2, y2 = box
        x1, y1, x2, y2 = int(x1), int(y1), int(x2), int(y2)
        converted_image[y1:y2, x1:x2][converted_image[y1:y2, x1:x2] == 1] = 0.5

    return converted_image

def set_image_data(self, input_image, input_image_id):
    if input_image_id != self.previous_image_id:
        if input_image_id not in self.cache:
            self.predictor.set_image(input_image)
            self.cache.set(
                input_image_id,
                {
                    "features": self.predictor.features,
                    "input_size": self.predictor.input_size,
                    "original_size": self.predictor.original_size,
                },
            )
        else:
            cached_data = self.cache.get(input_image_id)
            self.predictor.features = cached_data["features"]
            self.predictor.input_size = cached_data["input_size"]
            self.predictor.original_size = cached_data["original_size"]
```

Image matting task usually assumes usage of trimap - specific mask which divides input image into three types of areas: foreground, background, and transition region. But manual creation of trimap can be very time-consuming. Matte Anything uses segmentation mask predicted by Segment Anything and processes it via erosion and dilation (in code it is implemented in `generate_trimap` method) to generate trimap automatically. ViTMatte uses this trimap as an input to predict alpha mask. If user has chosen to use Grounding DINO, then this model will be used for detecting transparent objects - if such objects were found on image, then trimap will be corrected based on this information - `convert_pixels` method is used in order to put such objects into transition area of trimap. `set_image_data` method is used to avoid unnecessary calculations - if given image id is in cache, then it means that predictor features for this image are already calculated and we can simply take them from cache instead of calculating them again from scratch.

**serve**

The code below implements `serve` method with `smart_segmentation` endpoint for taking requests on segmentation and sending responses with encoded mask to the platform:

```python
def serve(self):
    super().serve()
    server = self._app.get_server()

    @server.post("/smart_segmentation")
    def smart_segmentation(response: Response, request: Request):
        try:
            smtool_state = request.state.context
            api = request.state.api
            crop = smtool_state["crop"]
        except Exception as exc:
            logger.warn("Error parsing request:" + str(exc), exc_info=True)
            response.status_code = status.HTTP_400_BAD_REQUEST
            return {"message": "400: Bad request.", "success": False}

        torch.set_grad_enabled(False)
        image_np = api.image.download_np(smtool_state["image_id"])
        self.set_image_data(image_np, smtool_state["image_id"])
        dino_transform = T.Compose(
            [
                T.RandomResize([800], max_size=1333),
                T.ToTensor(),
                T.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
            ]
        )
        image_transformed, _ = dino_transform(Image.fromarray(image_np), None)
        positive_clicks, negative_clicks = (
            smtool_state["positive"],
            smtool_state["negative"],
        )
        clicks = [{**click, "is_positive": True} for click in positive_clicks]
        clicks += [{**click, "is_positive": False} for click in negative_clicks]
        point_coordinates, point_labels = [], []
        for click in clicks:
            point_coordinates.append([click["x"], click["y"]])
            if click["is_positive"]:
                point_labels.append(1)
            else:
                point_labels.append(0)
        points = torch.Tensor(point_coordinates).to(self.device).unsqueeze(1)
        labels = torch.Tensor(point_labels).to(self.device).unsqueeze(1)
        transformed_points = self.predictor.transform.apply_coords_torch(
            points, image_np.shape[:2]
        )
        point_coords = transformed_points.permute(1, 0, 2)
        point_labels = labels.permute(1, 0)
        bbox_coordinates = torch.Tensor(
            [
                crop[0]["x"],
                crop[0]["y"],
                crop[1]["x"],
                crop[1]["y"],
            ]
        ).to(self.device)
        transformed_boxes = self.predictor.transform.apply_boxes_torch(
            bbox_coordinates, image_np.shape[:2]
        )
        masks, scores, logits = self.predictor.predict_torch(
            point_coords=point_coords,
            point_labels=point_labels,
            boxes=transformed_boxes,
            multimask_output=False,
        )
        masks = masks.cpu().detach().numpy()
        mask_all = np.ones((image_np.shape[0], image_np.shape[1], 3))
        for ann in masks:
            color_mask = np.random.random((1, 3)).tolist()[0]
            for i in range(3):
                mask_all[ann[0] == True, i] = color_mask[i]

        torch.cuda.empty_cache()

        mask = masks[0][0].astype(np.uint8) * 255
        erode_kernel_size = self.erode_input.get_value()
        dilate_kernel_size = self.dilate_input.get_value()
        trimap = self.generate_trimap(
            mask, erode_kernel_size, dilate_kernel_size
        ).astype(np.float32)

        trimap[trimap == 128] = 0.5
        trimap[trimap == 255] = 1

        if self.gr_dino_checkbox.is_checked():
            tr_box_threshold = self.dino_box_thresh_input.get_value()
            tr_text_threshold = self.dino_text_thresh_input.get_value()
            tr_caption = self.dino_text_prompt.get_value()
            boxes, logits, phrases = dino_predict(
                model=self.grounding_dino,
                image=image_transformed,
                caption=tr_caption,
                box_threshold=tr_box_threshold,
                text_threshold=tr_text_threshold,
                device=self.device,
            )
            if boxes.shape[0] == 0:
                # no transparent object detected
                pass
            else:
                h, w, _ = image_np.shape
                boxes = boxes * torch.Tensor([w, h, w, h])
                xyxy = box_convert(
                    boxes=boxes, in_fmt="cxcywh", out_fmt="xyxy"
                ).numpy()
                trimap = self.convert_pixels(trimap, xyxy)

        input = {
            "image": torch.from_numpy(image_np).permute(2, 0, 1).unsqueeze(0) / 255,
            "trimap": torch.from_numpy(trimap).unsqueeze(0).unsqueeze(0),
        }

        torch.cuda.empty_cache()

        alpha = self.vitmatte(input)["phas"].flatten(0, 2)
        alpha = alpha.detach().cpu().numpy()

        torch.cuda.empty_cache()

        alpha = alpha[crop[0]["y"] : crop[1]["y"], crop[0]["x"] : crop[1]["x"], ...]
        im = F.to_pil_image(alpha)
        im.save("alpha.png")
        with open("alpha.png", "rb") as image_file:
            encoded_string = base64.b64encode(image_file.read())
        os.remove("alpha.png")
        origin = {
            "x": crop[0]["x"],
            "y": crop[0]["y"],
        }
        response = {
            "data": encoded_string,
            "origin": origin,
            "success": True,
            "error": None,
        }
        return response
```

After implementing class we will have to initialize it and execute `serve` method:

```python
m = MatteAnythingModel(model_dir="app_data", use_gui=True)
m.serve()
```

## Debug in Supervisely platform

Once the code is written, it's time to test it right in the Supervisely platform as a debugging app.

First of all it is necessary to create `.vscode` folder and `launch.json` file inside this folder. Your `launch.json` file should contain the following:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Advanced mode for Supervisely Team",
            "type": "python",
            "request": "launch",
            "module": "uvicorn",
            "args": [
                "serving_app.main:m.app",
                "--host",
                "0.0.0.0",
                "--port",
                "8000",
                "--ws",
                "websockets"
            ],
            "jinja": true,
            "justMyCode": false,
            "env": {
                "PYTHONPATH": "${workspaceFolder}:${PYTHONPATH}",
                "LOG_LEVEL": "DEBUG",
                "ENV": "production",
                "FOLDER": "./my_model",
                "DEBUG_WITH_SLY_NET": "1",
                "SLY_APP_DATA_DIR": "${workspaceFolder}/results"
            }
        }
    ]
}
```

You can read more about advanced debug mode [here](https://developer.supervisely.com/app-development/advanced/advanced-debugging).

After that: 

1. If you develop in a Docker container, you should run the container with `--cap_add=NET_ADMIN` option.

2. Install `sudo apt-get install wireguard iproute2` or `brew install wireguard-tools` for Mac.

3. Define your `TEAM_ID` in the `debug.env` file. *Actually there are other env variables that is needed, but they are already provided in `./vscode/launch.json` for you.

4. Switch the `launch.json` config to the `Advanced debug in Supervisely platform`:

![Advanced Debug in Supervisely](https://user-images.githubusercontent.com/31512713/224290229-5da93fd2-dc97-4911-abb5-66ce890485a2.png)

5. Run the code.

✅ It will deploy the model in the Supervisely platform as a REST API.

Here is how advanced debug mode launch looks like:

{% embed url="https://user-images.githubusercontent.com/91027877/257574738-6c07c37a-7b20-4e02-8fba-9f4fb5b98bef.mp4" %}

After advanced debug launch you must be able to debug your app via `Develop & Debug` app:

{% embed url="https://github.com/supervisely/developer-portal/assets/91027877/070cb2a8-022e-442b-b3ec-5ea5c95dbe3c" %}

## Release your code as a Supervisely App

### Repository structure

The structure of [our GitHub repository](https://github.com/supervisely-ecosystem/Serve-Matte-Anything/tree/master) is the following:

```
|-- GroundingDINO
|   |-- Dockerfile
|   |-- LICENSE
|   |-- README.md
|   |-- demo
|   |   |-- create_coco_dataset.py
|   |   |-- gradio_app.py
|   |   |-- image_editing_with_groundingdino_gligen.ipynb
|   |   |-- image_editing_with_groundingdino_stablediffusion.ipynb
|   |   |-- inference_on_a_image.py
|   |   `-- test_ap_on_coco.py
|   |-- docker_test.py
|   |-- environment.yaml
|   |-- groundingdino
|   |   |-- __init__.py
|   |   |-- config
|   |   |   |-- GroundingDINO_SwinB_cfg.py
|   |   |   |-- GroundingDINO_SwinT_OGC.py
|   |   |   `-- __init__.py
|   |   |-- datasets
|   |   |   |-- __init__.py
|   |   |   |-- cocogrounding_eval.py
|   |   |   `-- transforms.py
|   |   |-- models
|   |   |   |-- GroundingDINO
|   |   |   |   |-- __init__.py
|   |   |   |   |-- backbone
|   |   |   |   |   |-- __init__.py
|   |   |   |   |   |-- backbone.py
|   |   |   |   |   |-- position_encoding.py
|   |   |   |   |   `-- swin_transformer.py
|   |   |   |   |-- bertwarper.py
|   |   |   |   |-- csrc
|   |   |   |   |   |-- MsDeformAttn
|   |   |   |   |   |   |-- ms_deform_attn.h
|   |   |   |   |   |   |-- ms_deform_attn_cpu.cpp
|   |   |   |   |   |   |-- ms_deform_attn_cpu.h
|   |   |   |   |   |   |-- ms_deform_attn_cuda.cu
|   |   |   |   |   |   |-- ms_deform_attn_cuda.h
|   |   |   |   |   |   `-- ms_deform_im2col_cuda.cuh
|   |   |   |   |   |-- cuda_version.cu
|   |   |   |   |   `-- vision.cpp
|   |   |   |   |-- fuse_modules.py
|   |   |   |   |-- groundingdino.py
|   |   |   |   |-- ms_deform_attn.py
|   |   |   |   |-- transformer.py
|   |   |   |   |-- transformer_vanilla.py
|   |   |   |   `-- utils.py
|   |   |   |-- __init__.py
|   |   |   `-- registry.py
|   |   `-- util
|   |       |-- __init__.py
|   |       |-- box_ops.py
|   |       |-- get_tokenlizer.py
|   |       |-- inference.py
|   |       |-- logger.py
|   |       |-- misc.py
|   |       |-- slconfig.py
|   |       |-- slio.py
|   |       |-- time_counter.py
|   |       |-- utils.py
|   |       |-- visualizer.py
|   |       `-- vl_utils.py
|   |-- requirements.txt
|   |-- setup.py
|   `-- test.ipynb
|-- configs
|   |-- common
|   |   |-- dataloader.py
|   |   |-- model.py
|   |   |-- optimizer.py
|   |   |-- scheduler.py
|   |   `-- train.py
|   |-- vitmatte_b.py
|   `-- vitmatte_s.py
|-- docker
|   |-- Dockerfile
|   `-- publish.sh
|-- local.env
|-- media
|   |-- deployed.png
|   |-- icon.png
|   |-- matte-anything.png
|   |-- matting_example.mp4
|   |-- matting_project_settings.mp4
|   |-- poster.png
|   `-- pretrained_models.png
|-- modeling
|   |-- __init__.py
|   |-- __pycache__
|   |   |-- __init__.cpython-310.pyc
|   |   `-- __init__.cpython-38.pyc
|   |-- backbone
|   |   |-- __init__.py
|   |   |-- __pycache__
|   |   |   |-- __init__.cpython-310.pyc
|   |   |   |-- __init__.cpython-38.pyc
|   |   |   |-- backbone.cpython-310.pyc
|   |   |   |-- backbone.cpython-38.pyc
|   |   |   |-- utils.cpython-310.pyc
|   |   |   |-- utils.cpython-38.pyc
|   |   |   |-- vit.cpython-310.pyc
|   |   |   `-- vit.cpython-38.pyc
|   |   |-- backbone.py
|   |   |-- utils.py
|   |   `-- vit.py
|   |-- criterion
|   |   |-- __init__.py
|   |   |-- __pycache__
|   |   |   |-- __init__.cpython-310.pyc
|   |   |   |-- __init__.cpython-38.pyc
|   |   |   |-- matting_criterion.cpython-310.pyc
|   |   |   `-- matting_criterion.cpython-38.pyc
|   |   `-- matting_criterion.py
|   |-- decoder
|   |   |-- __init__.py
|   |   |-- __pycache__
|   |   |   |-- __init__.cpython-310.pyc
|   |   |   |-- __init__.cpython-38.pyc
|   |   |   |-- detail_capture.cpython-310.pyc
|   |   |   `-- detail_capture.cpython-38.pyc
|   |   `-- detail_capture.py
|   `-- meta_arch
|       |-- __init__.py
|       |-- __pycache__
|       |   |-- __init__.cpython-310.pyc
|       |   |-- __init__.cpython-38.pyc
|       |   |-- vitmatte.cpython-310.pyc
|       |   `-- vitmatte.cpython-38.pyc
|       `-- vitmatte.py
|-- models_data
|   |-- grounding_dino.json
|   |-- segment_anything.json
|   `-- vitmatte.json
|-- pretrained
|   |-- groundingdino_swinb_cogcoor.pth
|   |-- groundingdino_swint_ogc.pth
|   |-- sam_vit_b.pth
|   |-- sam_vit_h.pth
|   |-- sam_vit_l.pth
|   |-- vitmatte_b_com.pth
|   |-- vitmatte_b_dis.pth
|   |-- vitmatte_s_com.pth
|   `-- vitmatte_s_dis.pth
|-- serving_app
|   |-- README.md
|   |-- config.json
|   `-- main.py
`-- supervisely.env
```

Explanation:

* `serving_app/main.py` - main inference script
* `serving_app/README.md` - readme of your application, it is the main page of an application in Ecosystem with some images, videos, and how-to-use guides
* `serving_app/config.json` - configuration of the Supervisely application, which defines the name and description of the app, its context menu, icon, poster, and running settings 
* `supervisely.env` - file with variables used for debugging
* `docker/` - directory with the custom Dockerfile for this application and the script that builds it and publishes it to the docker registry

### App configuration

App configuration is stored in `config.json` file. A detailed explanation of all possible fields is covered in this [Configuration Tutorial](https://developer.supervisely.com/app-development/basics/app-json-config/config.json). Let's check the config for our current app: 

```json
{
    "name": "Serve Matte Anything",
    "type": "app",
    "version": "2.0.0",
    "description": "Deploy Matte Anything as REST API service",
    "categories": [
        "neural network",
        "images",
        "interactive segmentation",
        "image matting",
        "serve"
    ],
    "need_gpu": true,
    "gpu": "required",
    "session_tags": [
        "deployed_nn_object_segmentation"
    ],
    "community_agent": false,
    "docker_image": "supervisely/serve-matte-anything:1.0.1",
    "instance_version": "6.9.22",
    "entrypoint": "python3 -m uvicorn serving_app.main:m.app --app-dir ./serve --host 0.0.0.0 --port 8000 --ws websockets",
    "port": 8000,
    "icon": "https://github.com/supervisely-ecosystem/Serve-Matte-Anything/releases/download/v0.0.1/icon.png",
    "icon_cover": true,
    "poster": "https://github.com/supervisely-ecosystem/Serve-Matte-Anything/releases/download/v0.0.1/poster.png",
    "task_location": "application_sessions",
    "license": {
        "type": "MIT"
    }
}
```

Here is the explanation for the fields:

* `type` - type of the module in Supervisely Ecosystem
* `version` - version of Supervisely App Engine. Just keep it by default
* `name` - the name of the application
* `description` - the description of the application
* `categories` - these tags are used to place the application in the correct category in Ecosystem
* `session_tags` - these tags will be assigned to every running session of the application. They can be used by other apps to find and filter all running sessions
* `need_gpu: true` - should be true if you want to use any `cuda` devices
*  `gpu: required` - app can be runned only on GPU devices
* `community_agent: false` - this means that this app can not be run on the agents started by Supervisely team, so users have to connect their own computers and run the app only on their own agents. Only applicable in Community Edition. Enterprise customers use their private instances so they can ignore the current option
* `docker_image` - Docker container will be started from the defined Docker image, github repository will be downloaded and mounted inside the container
* `entrypoint` - the command that starts our application in a container
* `port` - port inside the container

### App release

Once you've tested the code, it's time to release it into the platform. It can be released as an App that is shared with the all Supervisely community, or as your own private App.

Refer to [How to Release your App](https://developer.supervisely.com/app-development/basics/from-script-to-supervisely-app) for all releasing details. For a private app check also [Private App Tutorial](https://developer.supervisely.com/app-development/basics/add-private-app).
