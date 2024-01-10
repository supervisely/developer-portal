# SelectProject

## Introduction

**`SelectProject`** widget in Supervisely is a dropdown menu that allows users to select a project from a list of available projects in current workspace. Clicking on it can be processed from python code. This widget is particularly useful when working with multiple projects in Supervisely and allows to easily switch between projects in applications.

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

<figure><img src="https://user-images.githubusercontent.com/120389559/217844120-65f36676-0e42-4a75-a74b-3ae2f8d18167.png" alt=""><figcaption></figcaption></figure>

## Parameters

|    Parameters   |                    Type                   |                       Description                       |
| :-------------: | :---------------------------------------: | :-----------------------------------------------------: |
|   `default_id`  |                   `int`                   |                        Project ID                       |
|  `workspace_id` |                   `int`                   |                       Workspace ID                      |
|    `compact`    |                   `bool`                  |       Show compact view of project selector widget      |
| `allowed_types` |            `List[ProjectType]`            | List of project types witch will be available to select |
|   `show_label`  |                   `bool`                  |                        Show label                       |
|      `size`     | `Literal["large", "small", "mini", None]` |             Selector size (large/small/mini)            |
|   `widget_id`   |                   `str`                   |                     ID of the widget                    |

### default\_id

Determine `Project` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_project = SelectProject(default_id=project_id)
```

### workspace\_id

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

<figure><img src="https://user-images.githubusercontent.com/120389559/217844837-4b142a81-b456-4f1d-a57f-500e7f38adb8.png" alt=""><figcaption></figcaption></figure>

### allowed\_types

List of project types witch will be available to select. Set one of available `sly.ProjectType`s: `IMAGES`, `VIDEOS`, `VOLUMES`, `POINT_CLOUDS`, `POINT_CLOUD_EPISODES`.

**type:** `List[ProjectType]`

**default value:** `[]`

### show\_label

Determine show text `Project` on widget or not. This parameter only affects when parameter `compact` is `True`.

**type:** `bool`

**default value:** `True`

```python
select_project = SelectProject(
    default_id=project_id, workspace_id=workspace_id, compact=True, show_label=False
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/217845166-41f15aae-febf-4c27-9084-ba88e9e5550a.png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="https://user-images.githubusercontent.com/120389559/218711520-86b459e0-4dcb-45e5-aa76-f68a1638f718.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                       |
| :--------------------: | ------------------------------------------------- |
|   `get_selected_id()`  | Return selected project id.                       |
|   `set_project_id()`   | Set current project id.                           |
|    `@value_changed`    | Decorator function to detect changes in selector. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/selection/004\_select\_project/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/004\_select\_project/src/main.py)

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
workspace_id = sly.env.workspace_id()
project_id = sly.env.project_id()
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

<figure><img src="https://user-images.githubusercontent.com/120389559/217844120-65f36676-0e42-4a75-a74b-3ae2f8d18167.png" alt=""><figcaption></figcaption></figure>
