# Select Dataset

## Introduction

This widget is a select `Dataset` input, clicking on it can be processed from python code. In this tutorial you will learn how to use `SelectDataset` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/selectdataset)

## Function signature

```python
SelectDataset(
    default_id=None,
    project_id=None,
    multiselect=False,
    compact=False,
    show_label=True,
    size=None,
    disabled=False,
    widget_id=None,
)
```

![default](https://user-images.githubusercontent.com/120389559/217846091-eb9b97a2-8a11-4e61-b8fe-2387b8a3137c.png)

## Parameters

|  Parameters   |                   Type                    |                   Description                   |
| :-----------: | :---------------------------------------: | :---------------------------------------------: |
| `default_id`  |                   `int`                   |                  `Dataset` ID                   |
| `project_id`  |                   `int`                   |                  `Project` ID                   |
| `multiselect` |                  `bool`                   | Allow to select all datasets in current project |
|   `compact`   |                  `bool`                   |            Show only dataset select             |
| `show_label`  |                  `bool`                   |                   Show label                    |
|    `size`     | `Literal["large", "small", "mini", None]` |        Selector size (large/small/mini)         |
|  `disabled`   |                  `bool`                   |             Disable dataset select              |
|  `widget_id`  |                   `str`                   |                Id of the widget                 |

### default_id

Determine `Dataset` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_project = SelectDataset(default_id=dataset_id)
```

### project_id

Determine `Project` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_project = SelectDataset(project_id=project_id)
```

### multiselect

Allow to select all datasets in current project.

**type:** `bool`

**default value:** `false`

```python
select_dataset = SelectDataset(default_id=dataset_id, project_id=project_id, multiselect=True)
```

![multiselect](https://user-images.githubusercontent.com/120389559/218098518-60cce7a0-d9d9-404e-af60-9310993296a8.gif)

### compact

Show only `Dataset` select.

**type:** `bool`

**default value:** `false`

```python
select_dataset = SelectDataset(default_id=dataset_id, project_id=project_id, compact=True)
```

![compact](https://user-images.githubusercontent.com/120389559/217848891-7caf3883-fcb2-48d0-b0bb-4393a159ba6a.png)

### show_label

Determine show text `Dataset` on widget or not, work only if `compact` is True.

**type:** `bool`

**default value:** `True`

```python
select_dataset = SelectDataset(
    default_id=dataset_id, project_id=project_id, compact=True, show_label=False
)
```

![show_label](https://user-images.githubusercontent.com/120389559/217849159-d53c6dbd-520a-410a-b0bc-7e1fbc064f8d.png)

### size

Size of input.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
select_dataset = SelectDataset(
    default_id=dataset_id, project_id=project_id, compact=True, show_label=False
)
select_mini = SelectDataset(
    default_id=dataset_id, project_id=project_id, compact=True, show_label=False, size="mini"
)
select_small = SelectDataset(
    default_id=dataset_id, project_id=project_id, compact=True, show_label=False, size="small"
)
select_large = SelectDataset(
    default_id=dataset_id, project_id=project_id, compact=True, show_label=False, size="large"
)
card = Card(
    title="Select Dataset",
    content=Container(widgets=[select_dataset, select_mini, select_small, select_large]),
)
```

![size](https://user-images.githubusercontent.com/120389559/218713836-2e03438c-2ce3-49be-8a9c-292c617cca14.png)

### disabled

Determine `Dataset` select ability.

**type:** `bool`

**default value:** `false`

```python
select_dataset = SelectDataset(default_id=dataset_id, project_id=project_id, disabled=True)
```

![disabled](https://user-images.githubusercontent.com/120389559/217849459-1944a41a-df7a-4cac-ba77-3519fb67c734.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                                                    |
| :--------------------: | ------------------------------------------------------------------------------ |
|  `get_selected_id()`   | Return selected `dataset ID`, if `multiselect` is `True` raise `ValueError`.   |
|  `get_selected_ids()`  | Return selected `dataset IDs`, if `multiselect` is `False` raise `ValueError`. |
|      `disable()`       | Set `disabled` attribute == `True`.                                            |
|       `enable()`       | Set `disabled` attribute == `False`.                                           |
|   `value_changed()`    | Handled when selected `dataset ID` is changed.                                 |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/011_select_dataset/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/011_select_dataset/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, SelectDataset
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare `project_id` and `dataset_id`

```python
project_id = int(os.environ["modal.state.slyProjectId"])
dataset_id = int(os.environ["modal.state.slyDatasetId"])
```

### Initialize `SelectDataset` widget

```python
select_dataset = SelectDataset(
    default_id=dataset_id,
    project_id=project_id,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select Dataset",
    content=Container(widgets=[select_dataset]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```
