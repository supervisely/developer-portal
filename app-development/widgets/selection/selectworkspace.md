# SelectWorkspace

## Introduction

**`SelectWorkspace`** widget in Supervisely is a dropdown menu that allows users to select a workspace from a list of available workspaces. Clicking on it can be processed from python code. This widget is particularly useful when working with multiple workspaces in Supervisely and allows to easily switch between workspaces in applications.

## Function signature

```python
SelectWorkspace(
    default_id=None,
    team_id=None,
    compact=False,
    show_label=True,
    size=None,
    widget_id=None
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222355186-ce11c293-4cfd-4a16-96b0-029589002e3f.png" alt=""><figcaption></figcaption></figure>

## Parameters

|  Parameters  |                    Type                   |            Description           |
| :----------: | :---------------------------------------: | :------------------------------: |
| `default_id` |                   `int`                   |          `Workspace` ID          |
|   `team_id`  |                   `int`                   |             `Team` ID            |
|   `compact`  |                   `bool`                  |    Show only workspace select    |
| `show_label` |                   `bool`                  |            Show label            |
|    `size`    | `Literal["large", "small", "mini", None]` | Selector size (large/small/mini) |
|  `widget_id` |                   `str`                   |         ID of the widget         |

### default\_id

Determine `Workspace` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_workspace = SelectWorkspace(default_id=workspace_id)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222355688-b92e4bcc-a24f-4dd9-821f-c4067759868e.png" alt=""><figcaption></figcaption></figure>

### team\_id

Determine `Team` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_workspace = SelectWorkspace(team_id=team_id)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222355932-1fac2c04-3c0f-47a9-86f8-4ea5e3daf40c.png" alt=""><figcaption></figcaption></figure>

### compact

Show only `Workspace` select.

**type:** `bool`

**default value:** `false`

```python
select_workspace = SelectWorkspace(default_id=workspace_id, compact=True)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222358488-dbec5846-ddc0-4ad0-a33f-7aad84048f60.png" alt=""><figcaption></figcaption></figure>

### show\_label

Determine show text `Workspace` on widget or not, work only if `compact` is True.

**type:** `bool`

**default value:** `True`

```python
select_workspace = SelectWorkspace(
    default_id=workspace_id, team_id=team_id, compact=True, show_label=False
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221558086-abb3aaa3-8f2c-46f5-8ec4-3f66125aed9e.png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="https://user-images.githubusercontent.com/120389559/221558737-7a9ecd44-dae9-4d39-ad6f-319ff1ae3ab7.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                   |
| :--------------------: | ----------------------------- |
|   `get_selected_id()`  | Return selected workspace id. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/selection/003\_select\_workspace/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/003\_select\_workspace/src/main.py)

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
team_id = sly.env.team_id()
workspace_id = sly.env.workspace_id()
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

<figure><img src="https://user-images.githubusercontent.com/79905215/222355186-ce11c293-4cfd-4a16-96b0-029589002e3f.png" alt=""><figcaption></figcaption></figure>
