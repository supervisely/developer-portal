# MatchDatasets

## Introduction

**`MatchDatasets`** widget in Supervisely allows users to compare datasets by matching objects and displaying the results in a table. It also provides the comparison results in the form of a dictionary grouped into `matched`, `unique_left`, and `unique_right` categories. This widget helps users identify differences and similarities between datasets quickly and efficiently.

## Function signature

```python
MatchDatasets(
    left_datasets=None,
    right_datasets=None,
    left_name=None,
    right_name=None,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221359482-93e1897f-2820-40da-bb99-c7bc057742bf.png" alt=""><figcaption></figcaption></figure>

## Parameters

|    Parameters    |         Type        |                            Description                            |
| :--------------: | :-----------------: | :---------------------------------------------------------------: |
|  `left_datasets` | `List[DatasetInfo]` |  List of `NamedTuple`, containing information about left datasets |
| `right_datasets` | `List[DatasetInfo]` | List of `NamedTuple`, containing information about right datasets |
|    `left_name`   |        `str`        |                      Left part datasets name                      |
|   `right_name`   |        `str`        |                      Right part datasets name                     |
|    `widget_id`   |        `str`        |                          ID of the widget                         |

### left\_datasets

Determine information about left datasets.

**type:** `List[DatasetInfo]`

**default value:** `None`

### right\_datasets

Determine information about right datasets.

**type:** `List[DatasetInfo]`

**default value:** `None`

```python
dataset_left_id = 55830
dataset_left = api.dataset.get_info_by_id(id=dataset_left_id)

dataset_right_id = 55826
dataset_right = api.dataset.get_info_by_id(id=dataset_right_id)

match_datasets = MatchDatasets(left_datasets=[dataset_left], right_datasets=[dataset_right])
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221359482-93e1897f-2820-40da-bb99-c7bc057742bf.png" alt=""><figcaption></figcaption></figure>

### left\_name

Determine left part datasets name.

**type:** `str`

**default value:** `None`

### right\_name

Determine right part datasets name.

**type:** `str`

**default value:** `None`

```python
match_datasets = MatchDatasets(
    left_datasets=[dataset_left],
    right_datasets=[dataset_right],
    left_name="left_ds",
    right_name="right_ds",
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221360077-aade1945-c38f-42f4-8e96-0e757fb1cc3d.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                                   Attributes and Methods                                                  | Description                                              |
| :-----------------------------------------------------------------------------------------------------------------------: | -------------------------------------------------------- |
| `set(left_datasets: List[DatasetInfo] = None, right_datasets: List[DatasetInfo] = None, left_name=None, right_name=None)` | Set `DatasetInfo` data in left and right part of widget. |
|                                                        `get_stat()`                                                       | Return datasets match statistics.                        |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/compare data/001\_match\_datasets/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/compare%20data/001\_match\_datasets/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, MatchDatasets
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare datasets we will matched

```python
project_id1 = 17198
project_id2 = 17199

datasets_left = api.dataset.get_list(project_id1)
datasets_right = api.dataset.get_list(project_id2)
```

### Initialize `MatchDatasets` widget

```python
match_datasets = MatchDatasets(
    left_datasets=datasets_left,
    right_datasets=datasets_right,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Match Datasets",
    content=match_datasets,
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221360552-53097a9d-585f-4391-99a4-d065636d8560.png" alt=""><figcaption></figcaption></figure>
