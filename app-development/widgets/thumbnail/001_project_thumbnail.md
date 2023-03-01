# Project thumbnail

## Introduction

**`ProjectThumbnail`**  widget in Supervisely is a widget that allows to display a thumbnail image that represents supervisely project. It is a useful widget for applications that run from specific project, allowing users to have quick access to this project, so that when the user clicks on the thumbnail, the link will take him to this project.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/thumbnail/projectthumbnail)

## Function signature

```python
ProjectThumbnail(info=None, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/217830022-e16f1cac-83e6-4caf-aa1b-097116b68058.png)

## Parameters

| Parameters  |     Type      |                    Description                     |
| :---------: | :-----------: | :------------------------------------------------: |
|   `info`    | `ProjectInfo` | `NamedTuple`, containing information about project |
| `widget_id` |     `str`     |                  ID of the widget                  |

### info

`NamedTuple`, containing information about project.

**type:** `ProjectInfo`

**default value:** `None`

```python
project = api.project.get_info_by_id(project_id)
project_thumbnail = ProjectThumbnail(project)
```

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|  Attributes and Methods  | Description             |
| :----------------------: | ----------------------- |
| `set(info: ProjectInfo)` | Set input project data. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/thumbnail/001_project_thumbnail/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/thumbnail/001_project_thumbnail/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, ProjectThumbnail
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get Project ID and info

```python
project_id = sly.env.project_id()
project = api.project.get_info_by_id(project_id)
```

### Initialize `ProjectThumbnail` widget

```python
project_thumbnail = ProjectThumbnail(project)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
# create new cards
card = Card(
    title="Project Thumbnail",
    content=Container(widgets=[project_thumbnail]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/217830022-e16f1cac-83e6-4caf-aa1b-097116b68058.png)
