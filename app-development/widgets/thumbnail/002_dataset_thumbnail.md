# Dataset thumbnail

## Introduction

**`DatasetThumbnail`** widget in Supervisely is a widget that allows to display a thumbnail image that represents supervisely dataset. It is a useful widget for applications that run from specific dataset, allowing users to have quick access to this dataset, so that when the user clicks on the thumbnail, the link will take him to this dataset.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/thumbnail/datasetthumbnail)

## Function signature

```python
DatasetThumbnail(
    project_info=None,
    dataset_info=None,
    show_project_name=True,
    widget_id=None
)
```

![default](https://user-images.githubusercontent.com/120389559/217832111-9a9640fc-ee64-4164-a3ab-f2e18e47a65c.png)

## Parameters

|     Parameters      |     Type      |                    Description                     |
| :-----------------: | :-----------: | :------------------------------------------------: |
|   `project_info`    | `ProjectInfo` | `NamedTuple`, containing information about project |
|   `dataset_info`    | `DatasetInfo` | `NamedTuple`, containing information about dataset |
| `show_project_name` |    `bool`     |         Determines to display project name         |
|     `widget_id`     |     `str`     |                  ID of the widget                  |

### project_info

`NamedTuple`, containing information about project.

**type:** `ProjectInfo`

**default value:** `None`

```python
project = api.project.get_info_by_id(project_id)
dataset_thumbnail = DatasetThumbnail(project_info=project)
```

### dataset_info

`NamedTuple`, containing information about dataset.

**type:** `DatasetInfo`

**default value:** `None`

```python
dataset = api.dataset.get_info_by_id(id=dataset_id)
dataset_thumbnail = DatasetThumbnail(dataset_info=dataset)
```

### show_project_name

Determines to display project name.

**type:** `bool`

**default value:** `True`

```python
project = api.project.get_info_by_id(project_id)
dataset = api.dataset.get_info_by_id(dataset_id)
dataset_thumbnail = DatasetThumbnail(
    project_info=project,
    dataset_info=dataset,
    show_project_name=False
)
```

![show_project_name](https://user-images.githubusercontent.com/120389559/217832612-748b980d-d3af-40b4-aa51-c0e00226bf02.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                   Attributes and Methods                                    | Description             |
| :-----------------------------------------------------------------------------------------: | ----------------------- |
| `set(project_info: ProjectInfo, dataset_info: DatasetInfo, show_project_name: bool = True)` | Set input project data. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/thumbnail/002_dataset_thumbnail/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/thumbnail/002_dataset_thumbnail/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, DatasetThumbnail
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get project ID and info

```python
project_id = sly.env.project_id()
project = api.project.get_info_by_id(project_id)
```

### Get dataset ID and info

```python
dataset_id = sly.env.dataset_id()
dataset = api.dataset.get_info_by_id(id=dataset_id)
```

### Initialize `DatasetThumbnail` widget

```python
dataset_thumbnail = DatasetThumbnail(project_info=project, dataset_info=dataset)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Dataset Thumbnail",
    content=Container(widgets=[dataset_thumbnail]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```
