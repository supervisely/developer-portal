# SelectDataset

## Introduction

**`SelectDataset`** widget is a dropdown menu that allows users to select a dataset or multiple datasets from a list of available datasets in the current project. It displays the name of each dataset in the list.

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
    select_all_datasets = False,
    allowed_project_types = [],
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222369248-c5faf156-bf52-4685-906c-1b028a9f0f7d.png" alt=""><figcaption></figcaption></figure>

## Parameters

|       Parameters        |                   Type                    |                   Description                   |
| :---------------------: | :---------------------------------------: | :---------------------------------------------: |
|      `default_id`       |                   `int`                   |                  `Dataset` ID                   |
|      `project_id`       |                   `int`                   |                  `Project` ID                   |
|      `multiselect`      |                  `bool`                   | Allow to select all datasets in current project |
|        `compact`        |                  `bool`                   |            Show only dataset select             |
|      `show_label`       |                  `bool`                   |                   Show label                    |
|         `size`          | `Literal["large", "small", "mini", None]` |                  Selector size                  |
|       `disabled`        |                  `bool`                   |             Disable dataset select              |
|       `widget_id`       |                   `str`                   |                ID of the widget                 |
|  `select_all_datasets`  |                  `bool`                   |               Select all datasets               |
| `allowed_project_types` |            `List[ProjectType]`            |          List of allowed project types          |

### default\_id

Determine `Dataset` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_project = SelectDataset(default_id=dataset_id)
```

### project\_id

Determine `Project` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_project = SelectDataset(project_id=project_id)
```

### multiselect

Allow to select multiple datasets in current project.

**type:** `bool`

**default value:** `false`

```python
select_dataset = SelectDataset(default_id=dataset_id, project_id=project_id, multiselect=True)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222367677-cdee343d-a841-4868-9106-10d3f44d9e76.gif" alt=""><figcaption></figcaption></figure>

### compact

Show only `Dataset` select.

**type:** `bool`

**default value:** `false`

```python
select_dataset = SelectDataset(default_id=dataset_id, project_id=project_id, compact=True)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/217848891-7caf3883-fcb2-48d0-b0bb-4393a159ba6a.png" alt=""><figcaption></figcaption></figure>

### show\_label

Determine show text `Dataset` on widget or not, work only if `compact` is True.

**type:** `bool`

**default value:** `True`

```python
select_dataset = SelectDataset(
    default_id=dataset_id, project_id=project_id, compact=True, show_label=False
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/217849159-d53c6dbd-520a-410a-b0bc-7e1fbc064f8d.png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="https://user-images.githubusercontent.com/120389559/218713836-2e03438c-2ce3-49be-8a9c-292c617cca14.png" alt=""><figcaption></figcaption></figure>

### disabled

Determine `Dataset` select ability.

**type:** `bool`

**default value:** `false`

```python
select_dataset = SelectDataset(default_id=dataset_id, project_id=project_id, disabled=True)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222368297-d49c243d-a2bb-4df4-b474-e6a25f9a5a85.png" alt=""><figcaption></figcaption></figure>

### select_all_datasets

Select all datasets in current project. Work only if `multiselect` is `True`.

**type:** `bool`

**default value:** `false`

```python
select_dataset = SelectDataset(
    default_id=dataset_id,
    project_id=project_id,
    multiselect=True,
    select_all_datasets=True
)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/09a70fcd-2707-4ece-80e0-cea809c941c5" alt=""><figcaption></figcaption></figure>


### allowed_project_types

List of allowed project types.

**type:** `List[ProjectType]`

**default value:** `None`

```python
select_dataset = SelectDataset(
    default_id=dataset_id,
    project_id=project_id,
    allowed_project_types=[sly.ProjectType.POINT_CLOUD_EPISODES]
)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/bbb4cfc8-5ce7-4db2-9db7-39cfa4de1919" alt=""><figcaption></figcaption></figure>

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
|    `@value_changed`    | Decorator functions is handled when selected `dataset ID` is changed.          |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/selection/005\_select\_dataset/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/005\_select\_dataset/src/main.py)

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
project_id = sly.env.project_id()
dataset_id = sly.env.dataset_id()
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
