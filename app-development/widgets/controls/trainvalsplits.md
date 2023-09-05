# Train Val Splits

## Introduction

**`TrainValSplits`** widget in Supervisely is a tool that helps with the creation of training and validation datasets. The widget allows for easy splitting of the original dataset into training and validation sets based on customizable parameters such as percentage split or based on datasets or specific tag. `TrainValSplits` helps improve the performance of machine learning models by ensuring that they are trained on diverse and representative data.


## Function signature

```python
TrainValSplits(
    project_id=None,
    project_fs=None,
    random_splits=True,
    tags_splits=True,
    datasets_splits=True,
    widget_id=None,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/120389559/222371242-96bc091e-e3e4-4135-a578-727026453842.gif" alt="default" />
</p>

## Parameters

|    Parameters     |  Type  |                   Description                   |
| :---------------: | :----: | :---------------------------------------------: |
|   `project_id`    | `int`  |               Input `Project` ID                |
|   `project_fs`    | `str`  |      Path to input `Project` on local host      |
|  `random_splits`  | `bool` | Shuffle data and split with defined probability |
|   `tags_splits`   | `bool` |  Images should have assigned train or val tag   |
| `datasets_splits` | `bool` | Select one or several datasets for every split  |
|    `widget_id`    | `str`  |                ID of the widget                 |

### project_id

Determine input `Project` ID.

**type:** `int`

**default value:** `None`

```python
splits = TrainValSplits(project_id=project_id)
```

### project_fs

Determine path to input `Project` on local host.

**type:** `str`

**default value:** `None`

```python
project_dir = os.path.join(sly.app.get_data_dir(), project_info.name)
sly.Project.download(api, project_id, project_dir)
project_fs = sly.Project(project_dir, sly.OpenMode.READ)

splits = TrainValSplits(project_fs=project_fs)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/120389559/222371242-96bc091e-e3e4-4135-a578-727026453842.gif" alt="default" />
</p>

### random_splits

Shuffle data and split with defined probability.

**type:** `bool`

**default value:** `true`

```python
splits = TrainValSplits(
    project_id=project_id,
    random_splits=False,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/120389559/222375099-fa7f0aa2-51f0-4800-9c04-5d8a64de9f73.gif" alt="random_splits" />
</p>

### tags_splits

Images should have assigned train or val tag.

**type:** `bool`

**default value:** `true`

```python
splits = TrainValSplits(
    project_id=project_id,
    tags_splits=False,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/120389559/222375745-ec6dc5ee-a438-427e-b4ab-b55f8c2f185b.gif" alt="tags_splits" />
</p>

### datasets_splits

Select one or several datasets for every split.

**type:** `bool`

**default value:** `true`

```python
splits = TrainValSplits(
    project_id=project_id,
    datasets_splits=False,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/120389559/222376683-28eb633f-1ad7-4baa-a6c5-bbd32eb7d738.gif" alt="datasets_splits" />
</p>

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                    |
| :--------------------: | ------------------------------ |
|     `get_splits()`     | Return result train/val split. |
|      `disable()`       | Disable widget.                |
|       `enable()`       | Enable widget.                 |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/controls/006_train_val_splits/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/controls/006_train_val_splits/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, TrainValSplits, Button, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare project_id, download project and create project_fs

```python
project_id = sly.env.project_id()
project_info = api.project.get_info_by_id(project_id)
project_dir = os.path.join(sly.app.get_data_dir(), project_info.name)
sly.fs.remove_dir(project_dir)
sly.Project.download(api, project_id, project_dir)
project_fs = sly.Project(project_dir, sly.OpenMode.READ)
```

### Initialize `TrainValSplits` widget

```python
splits = TrainValSplits(project_fs=project_fs)
```

### Initialize `Button` and `Text` widget we will use

```python
button = Button("Get splits")
text = Text("")
text.hide()

```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Train Val Splits",
    content=Container([splits, button, text], gap=5),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add function to control widgets from python code

```python
@button.click
def get_splits():
    train_set, val_set = splits.get_splits()
    text.text = f"Train split: {len(train_set)} images, Val split: {len(val_set)} images"
    text.show()
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/120389559/222379597-f31db481-297d-4e29-a086-5ba9313208ed.gif" alt="layout" />
</p>
