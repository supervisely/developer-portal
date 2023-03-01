# Labeled Image

## Introduction

**`LabeledImage`** widget in Supervisely is designed to display an image with annotations and provides additional features such as the ability to toggle the visibility of individual class annotations, change fill opacity, and includes a zoom function.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/media/labeledimage)

## Function signature

```python
LabeledImage(
    annotations_opacity=0.5,
    show_opacity_slider=True,
    enable_zoom=False,
    esize_on_zoom=False,
    fill_rectangle=True,
    border_width=3,
    widget_id=None
)
```

![li_def](https://user-images.githubusercontent.com/79905215/221849681-7266b42f-636e-4068-a0d7-3825034d935b.png)

## Parameters

|      Parameters       |  Type   |                         Description                         |
| :-------------------: | :-----: | :---------------------------------------------------------: |
| `annotations_opacity` | `float` |                       Figures opacity                       |
| `show_opacity_slider` | `bool`  | Determines the presence of opacity slider on `LabeledImage` |
|     `enable_zoom`     | `bool`  |                Enable zoom on `LabeledImage`                |
|   `resize_on_zoom`    | `bool`  |                  Resize card to fit figure                  |
|   `fill_rectangle`    | `bool`  |                        Fill rectange                        |
|    `border_width`     |  `int`  |                  Border width (thickness)                   |
|      `widget_id`      |  `str`  |                      ID of the widget                       |

### annotations_opacity

Figures opacity.

**type:** `float`

**default value:** `0.5`

```python
labeled_image = LabeledImage(annotations_opacity=1)
labeled_image.set(title=image.name, image_url=image.preview_url, ann=ann)
```

![li_opacity](https://user-images.githubusercontent.com/79905215/221849926-66b25fb7-5df1-4398-aff6-7e5f2ba5d3c8.png)

### show_opacity_slider

Determines the presence of opacity slider on `LabeledImage`.

**type:** `bool`

**default value:** `true`

```python
labeled_image = LabeledImage(show_opacity_slider=False)
labeled_image.set(title=image.name, image_url=image.preview_url, ann=ann)
```

![li_opacity_slider](https://user-images.githubusercontent.com/79905215/221851814-bb98e802-9f1d-46d8-983f-c82d4c18c07a.png)

### enable_zoom

Enable zoom on `LabeledImage`.

**type:** `bool`

**default value:** `false`

```python
labeled_image = LabeledImage(enable_zoom=True)
labeled_image.set(title=image.name, image_url=image.preview_url, ann=ann)
```

![li_zoom](https://user-images.githubusercontent.com/79905215/221850680-d6ffdcf8-468f-4663-a29f-09695f45c76f.gif)

### resize_on_zoom

Resize card to fit figure.

**type:** `bool`

**default value:** `false`

### fill_rectangle

Resize card to fit figure.

**type:** `bool`

**default value:** `true`

```python
labeled_image = LabeledImage()
labeled_image.set(title=image.name, image_url=image.preview_url, ann=ann)
```

![fill_rectangle_true](https://user-images.githubusercontent.com/120389559/221583188-6bc606cd-abba-4e92-a342-dced21a093ab.gif)

```python
labeled_image = LabeledImage(fill_rectangle=False)
labeled_image.set(title=image.name, image_url=image.preview_url, ann=ann)
```

![fill_rectangle_false](https://user-images.githubusercontent.com/120389559/221583698-747dcb67-d14a-499a-b3b4-861f100ffd3d.gif)

### border_width

Resize card to fit figure.

**type:** `int`

**default value:** `3`

```python
labeled_image = LabeledImage(border_width=15)
labeled_image.set(title=image.name, image_url=image.preview_url, ann=ann)
```

![border_width](https://user-images.githubusercontent.com/120389559/221584066-de01e206-49cc-4289-b76c-c675be8b6fc5.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                               Attributes and Methods                                                | Description                     |
| :-----------------------------------------------------------------------------------------------------------------: | ------------------------------- |
|                                                        `id`                                                         | Get image id property.          |
|                                                      `loading`                                                      | Get or set `loading` property.  |
| `set(title: str, image_url: str, ann: Annotation, image_id: int, zoom_to: int, zoom_factor: float, title_url: str)` | Set item in `LabeledImage`.     |
|                                                    `clean_up()`                                                     | Clean `LabeledImage` from item. |
|                                                    `is_empty()`                                                     | Check `LabeledImage` is empty.  |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/002_labeled_image/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/002_labeled_image/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, LabeledImage
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get project ID and meta

```python
project_id = int(os.environ["modal.state.slyProjectId"])
project = api.project.get_info_by_id(project_id)
meta_json = api.project.get_meta(id=project_id)
meta = sly.ProjectMeta.from_json(data=meta_json)
```

### Get image ID and annotations

```python
image_id = int(os.environ["modal.state.slyImageId"])
image = api.image.get_info_by_id(id=image_id)
ann_json = api.annotation.download_json(image_id=image_id)
ann = sly.Annotation.from_json(data=ann_json, project_meta=meta)
```

### Initialize `LabeledImage` widget and fill it with image data

```python
labeled_image = LabeledImage()
labeled_image.set(title=image.name, image_url=image.preview_url, ann=ann)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Labeled Image",
    content=labeled_image,
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![li_app](https://user-images.githubusercontent.com/79905215/221851475-650b610b-a5c5-4e49-a32c-47d65094ac29.gif)
