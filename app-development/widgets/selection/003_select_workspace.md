# Select Workspace

## Introduction

This widget is a select `Workspace` input, clicking on it can be processed from python code. In this tutorial you will learn how to use `SelectWorkspace` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/selectworkspace)

## Function signature

```python
SelectWorkspace(default_id=None, team_id=None, compact=False, show_label=True, size=None, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218714865-879144d5-9567-4560-a49b-e3c8ee0154b7.png)

## Parameters

|  Parameters  |                   Type                    |           Description            |
| :----------: | :---------------------------------------: | :------------------------------: |
| `default_id` |                   `int`                   |          `Workspace` ID          |
|  `team_id`   |                   `int`                   |            `Team` ID             |
|  `compact`   |                  `bool`                   |    Show only workspace select    |
| `show_label` |                  `bool`                   |            Show label            |
|    `size`    | `Literal["large", "small", "mini", None]` | Selector size (large/small/mini) |
| `widget_id`  |                   `str`                   |         Id of the widget         |

### default_id

Determine `Workspace` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_workspace = SelectWorkspace(default_id=workspace_id)
```

![default_id](https://user-images.githubusercontent.com/120389559/218031925-1f70bb32-5a44-4ee2-9c9b-813fa88ac8a7.png)

### team_id

Determine `Team` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_workspace = SelectWorkspace(team_id=team_id)
```

### compact

Show only `Workspace` select.

**type:** `bool`

**default value:** `false`

```python
select_workspace = SelectWorkspace(default_id=workspace_id, compact=True)
```

![compact](https://user-images.githubusercontent.com/120389559/221557626-042f1911-7065-48d4-a830-c003f0baa4d2.png)

### show_label

Determine show text `Workspace` on widget or not, work only if `compact` is True.

**type:** `bool`

**default value:** `True`

```python
select_workspace = SelectWorkspace(
    default_id=workspace_id, team_id=team_id, compact=True, show_label=False
)
```

![show_label](https://user-images.githubusercontent.com/120389559/221558086-abb3aaa3-8f2c-46f5-8ec4-3f66125aed9e.png)

### size

Size of input.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
select_workspace = SelectWorkspace(default_id=workspace_id, compact=True, show_label=False)
select_workspace_mini = SelectWorkspace(
    default_id=workspace_id, compact=True, show_label=False, size="mini"
)
select_workspace_small = SelectWorkspace(
    default_id=workspace_id, compact=True, show_label=False, size="small"
)
select_workspace_large = SelectWorkspace(
    default_id=workspace_id, compact=True, show_label=False, size="large"
)
card = Card(
    title="Select Workspace",
    content=Container(
        widgets=[
            select_workspace,
            select_workspace_mini,
            select_workspace_small,
            select_workspace_large,
        ]
    ),
)
```

![size](https://user-images.githubusercontent.com/120389559/221558737-7a9ecd44-dae9-4d39-ad6f-319ff1ae3ab7.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                   |
| :--------------------: | ----------------------------- |
|  `get_selected_id()`   | Return selected workspace id. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/012_select_workspace/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/012_select_workspace/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, SelectWorkspace
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare `team_id` and `workspace_id`

```python
team_id = int(os.environ["modal.state.slyTeamId"])
workspace_id = int(os.environ["modal.state.slyWorkspaceId"])
```

### Initialize `SelectWorkspace` widget

```python
select_workspace = SelectWorkspace(
    default_id=workspace_id,
    team_id=team_id,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select Workspace",
    content=Container(widgets=[select_project]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218032788-64ba31fc-65b1-4194-99cb-0668b48e80d0.png)
