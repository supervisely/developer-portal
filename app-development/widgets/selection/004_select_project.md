# Select Project

## Introduction

This Supervisely widget allows you to create a dropdown menu that lets users select a project from a list of projects. It is a useful widget for applications that require users to select a project to work on or view.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/selectproject)

## Function signature

```python
SelectProject(
    default_id=None,
    workspace_id=None,
    compact=False,
    allowed_types=[],
    show_label=True,
    size=None,
    widget_id=None,
)
```

![default](https://user-images.githubusercontent.com/120389559/217844120-65f36676-0e42-4a75-a74b-3ae2f8d18167.png)

## Parameters

|   Parameters    |                   Type                    |                       Description                       |
| :-------------: | :---------------------------------------: | :-----------------------------------------------------: |
|  `default_id`   |                   `int`                   |                      `Project` ID                       |
| `workspace_id`  |                   `int`                   |                     `Workspace` ID                      |
|    `compact`    |                  `bool`                   |            Check `Workspace` ID is not None             |
| `allowed_types` |            `List[ProjectType]`            | List of project types witch will be available to select |
|  `show_label`   |                  `bool`                   |                       Show label                        |
|     `size`      | `Literal["large", "small", "mini", None]` |            Selector size (large/small/mini)             |
|   `widget_id`   |                   `str`                   |                    Id of the widget                     |

### default_id

Determine `Project` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_project = SelectProject(default_id=project_id)
```

### workspace_id

Determine `Workspace` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_project = SelectProject(workspace_id=workspace_id)
```

### compact

Show only `Project` select.

**type:** `bool`

**default value:** `false`

```python
select_project = SelectProject(default_id=project_id, workspace_id=workspace_id, compact=True)
```

![compact](https://user-images.githubusercontent.com/120389559/217844837-4b142a81-b456-4f1d-a57f-500e7f38adb8.png)

### allowed_types

List of project types witch will be available to select. Possible project types: `images`, `videos`, `volumes`, `point_clouds`, `point_cloud_episodes`.

**type:** `List[ProjectType]`

**default value:** `[]`

### show_label

Determine show text `Project` on widget or not, work only if `compact` is True.

**type:** `bool`

**default value:** `True`

```python
select_project = SelectProject(
    default_id=project_id, workspace_id=workspace_id, compact=True, show_label=False
)
```

![show_label](https://user-images.githubusercontent.com/120389559/217845166-41f15aae-febf-4c27-9084-ba88e9e5550a.png)

### size

Size of input.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
select_project = SelectProject(
    default_id=project_id, workspace_id=workspace_id, compact=True, show_label=False
)
select_mini = SelectProject(
    default_id=project_id, workspace_id=workspace_id, compact=True, show_label=False, size="mini"
)
select_small = SelectProject(
    default_id=project_id, workspace_id=workspace_id, compact=True, show_label=False, size="small"
)
select_large = SelectProject(
    default_id=project_id, workspace_id=workspace_id, compact=True, show_label=False, size="large"
)
card = Card(
    title="Select Project",
    content=Container(widgets=[select_project, select_mini, select_small, select_large]),
)
```

![size](https://user-images.githubusercontent.com/120389559/218711520-86b459e0-4dcb-45e5-aa76-f68a1638f718.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                 |
| :--------------------: | --------------------------- |
|  `get_selected_id()`   | Return selected project id. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/010_select_project/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/010_select_project/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, SelectProject
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare `project_id` and `workspace_id`

```python
workspace_id = int(os.environ["modal.state.slyWorkspaceId"])
project_id = int(os.environ["modal.state.slyProjectId"])
```

### Initialize `SelectProject` widget

```python
select_project = SelectProject(
    default_id=project_id,
    workspace_id=workspace_id,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select Project",
    content=Container(widgets=[select_project]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![default](https://user-images.githubusercontent.com/120389559/217844120-65f36676-0e42-4a75-a74b-3ae2f8d18167.png)
