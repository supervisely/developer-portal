# DestinationProject

## Introduction

**`DestinationProject`** widget in Supervisely provides several options for selecting the destination project and dataset when transferring data. Users can choose between creating a new project or selecting an existing project, as well as creating a new dataset or selecting an existing dataset within the project. `DestinationProject` also includes an input field where users can enter the name of the destination project or dataset when creating a new one. This flexibility allows users to easily manage and organize their projects and datasets within the platform.

## Function signature

```python
DestinationProject(
    workspace_id=654,
    project_type="videos",
    widget_id=None,
)
```

![default](https://user-images.githubusercontent.com/79905215/225234197-95dd3c3a-18dc-4fed-a583-b46cf101217c.png)

## Parameters

|   Parameters   |      Type     |                   Description                  |
| :------------: | :-----------: | :--------------------------------------------: |
| `workspace_id` |     `int`     |                  Workspace ID                  |
| `project_type` | `ProjectType` | Determine project type available for selection |
|   `widget_id`  |     `str`     |                ID of the widget                |

### workspace\_id

Workspace ID

**type:** `int`

```python
destination = DestinationProject(workspace_id=435)
```

### project\_type

Determine project type available for selection

**type:** `ProjectType`

**default value:** `ProjectType.IMAGES`

```python
destination = DestinationProject(
    workspace_id=435,
    project_type=sly.ProjectType.IMAGES,
)
```

![project\_type](https://user-images.githubusercontent.com/79905215/225234270-efeb6a3c-45a0-4a4c-9464-e0ba8e67f2d9.png)

### widget\_id

ID of the widget

**type:** `str`

**default value:** `None`

## Methods and attributes

|    Attributes and Methods   | Description                                                               |
| :-------------------------: | ------------------------------------------------------------------------- |
| `get_selected_project_id()` | Get selected Project ID if radio input in "Add to existing project" mode. |
| `get_selected_dataset_id()` | Get selected Dataset ID if radio input in "Add to existing dataset" mode. |
|     `get_project_name()`    | Get selected Project name if radio input in "Create new project" mode.    |
|     `get_dataset_name()`    | Get selected Dataset name if radio input in "Create new dataset" mode.    |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/selection/011\_destination\_project/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/011\_destination\_project/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, DestinationProject, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get `workspace_id` from environment variables

```python
workspace_id = sly.env.workspace_id()
```

### Initialize `DestinationProject` widget

```python
destination = DestinationProject(
    workspace_id=workspace_id,
    project_type=sly.ProjectType.IMAGES,
)
```

### Create `Button`, `Text` widgets we will use in UI for demo

```python
button = Button()
text_project_id = Text()
text_project_name = Text()
text_dataset_id = Text()
text_dataset_name = Text()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python

container = Container(
    widgets=[
        button,
        text_project_id,
        text_project_name,
        text_dataset_id,
        text_dataset_name,
    ]
)

card = Card(
    "Destination Project",
    content=Container([destination, container]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from code

```python
@button.click
def get_destination():
    text_project_id.text = f"project_id: {destination.get_selected_project_id()}"
    text_project_name.text = f"project_name: {destination.get_project_name()}"
    text_dataset_id.text = f"dataset_id: {destination.get_selected_dataset_id()}"
    text_dataset_name.text = f"dataset_name: {destination.get_dataset_name()}"
```

![layout](https://user-images.githubusercontent.com/79905215/225239059-f33aa092-a74f-47eb-93d3-247aec410e57.gif)
