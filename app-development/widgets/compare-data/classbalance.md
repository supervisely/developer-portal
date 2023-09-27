# Class Balance

## Introduction

**`ClassBalance`** is a widget in Supervisely that allows for displaying input data classes balance on the UI. For example, you can display the distribution of tags to different classes in the project, or set your data according to the required format.
It also provides functionality for data streaming and dynamic updates, allowing the class balance to display real-time data. Additionally, users can control the widget through Python code by detecting events such as clicking on a class name or segment data.

## Function signature

```python
ClassBalance(
    segments=[],
    rows_data=[],
    slider_data={},
    max_value=None,
    max_height=350,
    rows_height=100,
    selectable=True,
    collapsable=False,
    clickable_name=False,
    clickable_segment=False,
    widget_id=None,
)
```

Example of input data we will use. If `max_value` is None, the maximum `total` form `rows_data` will be taken as `max_value`.

```python
max_value = 1000
segments = [
    {"name": "train", "key": "train", "color": "#1892f8"},
    {"name": "val", "key": "val", "color": "#25e298"},
    {"name": "test", "key": "test", "color": "#fcaf33"},
]

rows_data = [
    {
        "nameHtml": "<strong>black-pawn</strong>",
        "name": "black-pawn",
        "total": 1000,
        "disabled": False,
        "segments": {"train": 600, "val": 350, "test": 50},
    },
    {
        "nameHtml": "<strong>white-pawn</strong>",
        "name": "white-pawn",
        "total": 700,
        "disabled": False,
        "segments": {"train": 400, "val": 250, "test": 50},
    },
]

slider_data = {
    "black-pawn": [
        {
            "moreExamples": ["https://www.w3schools.com/howto/img_nature.jpg"],
            "preview": "https://www.w3schools.com/howto/img_nature.jpg",
        }
    ],
    "white-pawn": [
        {
            "moreExamples": ["https://i.imgur.com/35pUPD2.jpg"],
            "preview": "https://i.imgur.com/35pUPD2.jpg",
        }
    ],
}
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/2618fa49-c62c-4478-a036-1abd61788adf)

## Parameters

|     Parameters      |     Type     |                      Description                      |
| :-----------------: | :----------: | :---------------------------------------------------: |
|     `segments`      | `List[Dict]` |            List of segments in the widget             |
|     `rows_data`     | `List[Dict]` |            List of rows data in the widget            |
|    `slider_data`    |    `Dict`    |     Dict containing `ClassBalance` slider images      |
|     `max_value`     |    `int`     | The maximum value of the row input data "total" field |
|    `max_height`     |    `int`     |  Specifies the maximum height of the `ClassBalance`   |
|    `rows_height`    |    `int`     |    Specifies the height of the `ClassBalance` rows    |
|    `selectable`     |    `bool`    |          Determines if rows can be selected           |
|    `collapsable`    |    `bool`    |    Determines whether a rows data can be collapsed    |
|  `clickable_name`   |    `bool`    |            Allows clicking on class names             |
| `clickable_segment` |    `bool`    |           Allows clicking on class segments           |
|     `widget_id`     |    `str`     |                   ID of the widget                    |

### segments

List of segments in the widget.

**type:** `List[Dict]`

**default value:** `[]`

### rows_data

List of rows data in the widget.

**type:** `List[Dict]`

**default value:** `[]`

```python
class_balance = ClassBalance(segments=segments, rows_data=rows_data)
```

![segments_rows_data](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/9ea3bedf-aeb7-434c-92e1-3213de65a95a)

### slider_data

Dict containing images for row images sliders. It needs `collapsable=True` to be set.

**type:** `Dict`

**default value:** `{}`

```python
class_balance = ClassBalance(
    segments=segments,
    rows_data=rows_data,
    slider_data=slider_data,
    collapsable=True,
)
```

![image_slider](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/f1061942-7fef-49f8-8d1f-9cac5987a195)

### max_value

The maximum value of the row input data "total" field. If `max_value` is None, the maximum `total` form `rows_data` will be taken as `max_value`.

**type:** `int`

**default value:** `None`

```python
class_balance = ClassBalance(
    max_value=2000,
    segments=segments,
    rows_data=rows_data,
)
```

![max_value](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/ac388e85-5d13-479f-bbfe-1df0360f402c)

### max_height

Specifies the maximum height of the `ClassBalance`.

**type:** `int`

**default value:** `350`

```python
class_balance = ClassBalance(
    segments=segments,
    rows_data=rows_data,
    slider_data=slider_data,
    collapsable=True
    max_height=250,
)
```

![max_height](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/17ff31d1-a53f-4d64-aa6f-8eea90f894b5)

### rows_height

Specifies the height of the `ClassBalance` rows.

**type:** `int`

**default value:** `100`

```python
class_balance = ClassBalance(
    segments=segments,
    rows_data=rows_data,
    slider_data=slider_data,
    collapsable=True
    rows_height=200,
)
```

![row_video](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/808e4c99-70a1-4486-a070-ced2ca9cfaea)

### selectable

Determines whether a collapse button is displayed.

**type:** `bool`

**default value:** `True`

```python
class_balance = ClassBalance(
    max_value=max_value,
    segments=segments,
    rows_data=rows_data,
    slider_data=slider_data,
    selectable=False,
    collapsable=True
)
```

![selectable](https://user-images.githubusercontent.com/120389559/224936812-ba9e1bce-8cb7-4729-9dc6-c52b27da6e04.png)

### collapsable

Display the collapse button. The case `collapsable=True` has been shown above to show examples for changing the `max_height` parameter. So now an example will be shown for case `collapsable=False`.

**type:** `bool`

**default value:** `False`

```python
class_balance = ClassBalance(
    max_value=max_value,
    segments=segments,
    rows_data=rows_data,
    slider_data=slider_data,
    collapsable=False
)
```

![collapsable](https://user-images.githubusercontent.com/120389559/224938851-fdf5a506-b100-49f9-90d5-17252a6720b7.png)

### clickable_name

Allows clicking on class names.

**type:** `bool`

**default value:** `False`

```python
class_balance = ClassBalance(
    max_value=max_value,
    segments=segments,
    rows_data=rows_data,
    slider_data=slider_data,
    clickable_name=True,
)
```

### clickable_segment

Allow clicking on class segments.

**type:** `bool`

**default value:** `False`

```python
class_balance = ClassBalance(
    max_value=max_value,
    segments=segments,
    rows_data=rows_data,
    slider_data=slider_data,
    clickable_segment=True,
)
```

## Methods and attributes

|         Attributes and Methods         | Description                                                                   |
| :------------------------------------: | ----------------------------------------------------------------------------- |
|            `is_selectable`             | Returns `True` if `ClassBalance` rows is selectable, otherwise `False`        |
|            `is_collapsable`            | Returns `True` if `ClassBalance` rows is collapsable, otherwise `False`       |
|          `is_clickable_name`           | Returns `True` if `ClassBalance` rows name is clickable, otherwise `False`    |
|         `is_clickable_segment`         | Returns `True` if `ClassBalance` rows segment is clickable, otherwise `False` |
|           `get_max_value()`            | Returns the maximum value of `ClassBalance` rows                              |
|      `set_max_value(value: int)`       | Sets the maximum value of `ClassBalance` rows                                 |
|           `get_max_height()`           | Return `ClassBalance` max height.                                             |
|      `set_max_height(value: int)`      | Sets the maximum height of `ClassBalance`                                     |
|            `get_segments()`            | Return `ClassBalance` `segments`                                              |
|  `add_segments(segments: List[Dict])`  | Add new `segments` to existing in `ClassBalance`                              |
|  `set_segments(segments: List[Dict])`  | Set new `segments` in `ClassBalance`                                          |
|           `get_rows_data()`            | Return `ClassBalance` `rows_data`                                             |
| `add_rows_data(rows_data: List[Dict])` | Add new `rows_data` to now existing in `ClassBalance`                         |
| `set_rows_data(rows_data: List[Dict])` | Set new `rows_data` in `ClassBalance`                                         |
|          `get_slider_data()`           | Return `ClassBalance` `slider_data`                                           |
|  `add_slider_data(slider_data: Dict)`  | Add new `slider_data` to now existing in `ClassBalance`                       |
|  `set_slider_data(slider_data: Dict)`  | Set new `slider_data` in `ClassBalance`                                       |
|         `get_selected_rows()`          | Return list of `ClassBalance` selected rows                                   |
|                `@click`                | Decorator function to handle clicks on class names or rows segments           |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/compare-data/004\_class\_balance/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/compare%20data/004\_class\_balance/src/main.py)

### Import libraries

```python
import os
from collections import defaultdict

from dotenv import load_dotenv
import supervisely as sly
from supervisely.app.widgets import Button, Card, ClassBalance, Container, Flexbox, Text
from supervisely.imaging.color import rgb2hex
from tqdm import tqdm
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Example 1. A simple example of using `ClassBalance` widget.

**Prepare series for class balance**

```python
max_value = 1000
segments = [
    {"name": "train", "key": "train", "color": "#1892f8"},
    {"name": "val", "key": "val", "color": "#25e298"},
    {"name": "test", "key": "test", "color": "#fcaf33"},
]

rows_data = [
    {
        "nameHtml": "<strong>black-pawn</strong>",
        "name": "black-pawn",
        "total": 1000,
        "disabled": False,
        "segments": {"train": 600, "val": 350, "test": 50},
    },
    {
        "nameHtml": "<strong>white-pawn</strong>",
        "name": "white-pawn",
        "total": 700,
        "disabled": False,
        "segments": {"train": 400, "val": 250, "test": 50},
    },
]

slider_data = {
    "black-pawn": [
        {
            "moreExamples": ["https://www.w3schools.com/howto/img_nature.jpg"],
            "preview": "https://www.w3schools.com/howto/img_nature.jpg",
        }
    ],
    "white-pawn": [
        {
            "moreExamples": ["https://i.imgur.com/35pUPD2.jpg"],
            "preview": "https://i.imgur.com/35pUPD2.jpg",
        }
    ],
}
```

**Initialize `ClassBalance`**

```python
class_balance_1 = ClassBalance(
    max_value=max_value,
    segments=segments,
    rows_data=rows_data,
    slider_data=slider_data,
    max_height=700,
    collapsable=True,
)
```

**Create card widget with first `ClassBalance` widget content**

```python
card_1 = Card(
    title="Simple example how to use ClassBalance widget",
    content=Container([class_balance_1]),
    collapsable=True,
)
card_1.collapse()
```

### Example 2. Advanced using `ClassBalance` widget.

👍 In this example, we will iterate through sample dataset images from the Supervisely platform and crop them based on object classes to set in the image slider. Finally, we will calculate and collect statistics for each class and display the class balance information.

**Create static dir**

Create a static directory to store the cropped images in local directory.
Or you can upload them to the Team Files and use the `full_storage_url` to display them in the image slider.

```python
static_dir = os.path.join(sly.app.get_data_dir(), "static")
sly.fs.mkdir(static_dir, remove_content_if_exists=True)
```

**Get environment variables**

```python
project_id = sly.env.project_id()
dataset_id = sly.env.dataset_id()
```

**Get `ProjectMeta` and `DatasetInfo` from the server**

```python
project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))
dataset = api.dataset.get_info_by_id(dataset_id)
```

**Get all image infos, annotations and download images from server**

```python
image_infos = api.image.get_list(dataset.id)
image_ids = [image_info.id for image_info in image_infos]
imame_nps = api.image.download_nps(dataset.id, image_ids)
anns_json = api.annotation.download_json_batch(dataset.id, image_ids)
anns = [sly.Annotation.from_json(json, project_meta) for json in anns_json]
```

**Calculate and collect series for the `ClassBalance` widget**

```python
PADDINGS = {"top": "20px", "left": "20px", "bottom": "20px", "right": "20px"}

progress = tqdm(desc=f"Processing datasets...", total=dataset.items_count)

# prepare data for ClassBalance widget
crop_id = 0
slider_data = defaultdict(list)
new_slider_data = defaultdict(list)
objclass_stats = defaultdict(lambda: defaultdict(lambda: 0))
# collect cropped images for image sliders
for image_info, img, ann in zip(image_infos, imame_nps, anns):
    for idx, objclass in enumerate(project_meta.obj_classes):
        # crop current image to separated images which contain current class instance
        crops = sly.aug.instance_crop(img, ann, objclass.name, False, PADDINGS)
        objclass_stats[objclass.name]["total"] += len(crops)

        for crop_img, crop_ann in crops:
            # draw annotations on image and upload result crop image to Team Files
            crop_ann.draw_pretty(crop_img)
            path = os.path.join(static_dir, f"{crop_id}-{image_info.name}")
            static_path = os.path.join("static", os.path.basename(path))
            crop_id += 1
            sly.image.write(path, crop_img)

            # collect cropped image paths for ClassBalance widget
            if idx == 0:
                slider_data[objclass.name].append({"preview": static_path})
            else:
                new_slider_data[objclass.name].append({"preview": static_path})

    # count number of objects with each tag
    for label in ann.labels:
        for tag in label.tags:
            objclass_stats[label.obj_class.name][tag.name] += 1

    progress.update(1)

# prepare rows data
rows_data = []
new_rows_data = []

tag_metas = project_meta.tag_metas
for idx, obj_class in enumerate(project_meta.obj_classes):
    data = {}
    data["name"] = obj_class.name
    data["nameHtml"] = f"<strong>{obj_class.name}</strong>"
    data["total"] = objclass_stats[obj_class.name]["total"]
    data["disabled"] = False
    data["segments"] = {}
    for tag_meta in tag_metas:
        data["segments"][tag_meta.name] = objclass_stats[obj_class.name][tag_meta.name]
    if idx == 0:
        rows_data.append(data)
    else:
        new_rows_data.append(data)

segments = [{"name": tm.name, "key": tm.name, "color": rgb2hex(tm.color)} for tm in tag_metas]
```

**Initialize `ClassBalance`**

```python
class_balance_2 = ClassBalance(
    max_value=max([stat["total"] for stat in objclass_stats.values()]),
    segments=segments[:-1],
    rows_data=rows_data,
    slider_data=slider_data,
    max_height=700,
    collapsable=True,
    clickable_name=True,
    clickable_segment=True,
)
```

**Create additional widgets**

```python
add_segment_btn = Button("Add segment")
add_row_btn = Button("Add row data")
add_slider_data_btn = Button("Add slider data")

buttons = Flexbox([add_segment_btn, add_row_btn, add_slider_data_btn], "horizontal")
text = Text()

card_2 = Card(
    title="Advanced example how to use ClassBalance widget",
    content=Container([class_balance_2, buttons, text]),
    collapsable=True,
)
card_2.collapse()
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter.

```python
layout = Container(widgets=[card_1, card_2])
```

### Create app using layout

Create an app object with the layout and static_dir parameters.

```python
app = sly.Application(layout=layout, static_dir=static_dir)
```

**Add functions to control widgets from python code**

```python
@class_balance_2.click
def show_item(res):
    if res.get("segmentValue") is not None and res.get("segmentName") is not None:
        info = (
            f"Class {res['name']} contain {res['segmentValue']} tags with name {res['segmentName']}"
        )
        if res["segmentName"] == "val":
            status = "success"
        elif res["segmentName"] == "test":
            status = "warning"
        elif res["segmentName"] == "trash":
            status = "error"
        else:
            status = "info"
    else:
        info = f"Class {res['name']}"
        status = "text"

    text.set(text=info, status=status)


@add_segment_btn.click
def add_segment():
    class_balance_2.add_segments(segments[-1:])


@add_row_btn.click
def add_row():
    class_balance_2.add_rows_data(new_rows_data)


@add_slider_data_btn.click
def add_slider_data():
    class_balance_2.add_slider_data(new_slider_data)
```

![layout](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/73e9a5b7-e06f-40a7-b5e1-9c5fed436161)
