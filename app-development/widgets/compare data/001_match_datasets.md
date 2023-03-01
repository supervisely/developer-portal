# Match Datasets

## Introduction

In this tutorial you will learn how to use `MatchDatasets` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/matchdatasets)

## Function signature

```python
MatchDatasets(left_datasets=None, right_datasets=None, left_name=None, right_name=None, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/221359482-93e1897f-2820-40da-bb99-c7bc057742bf.png)

## Parameters

|    Parameters    |        Type         |                            Description                            |
| :--------------: | :-----------------: | :---------------------------------------------------------------: |
| `left_datasets`  | `List[DatasetInfo]` | List of `NamedTuple`, containing information about left datasets  |
| `right_datasets` | `List[DatasetInfo]` | List of `NamedTuple`, containing information about right datasets |
|   `left_name`    |        `str`        |                      Left part datasets name                      |
|   `right_name`   |        `str`        |                     Right part datasets name                      |
|   `widget_id`    |        `str`        |                         Id of the widget                          |

### left_datasets

Determine information about left datasets.

**type:** `List[DatasetInfo]`

**default value:** `None`

### right_datasets

Determine information about right datasets.

**type:** `List[DatasetInfo]`

**default value:** `None`

```python
dataset_left_id = int(os.environ["modal.state.slyDatasetId_left"])
dataset_left = api.dataset.get_info_by_id(id=dataset_left_id)

dataset_right_id = int(os.environ["modal.state.slyDatasetId_right"])
dataset_right = api.dataset.get_info_by_id(id=dataset_right_id)

match_datasets = MatchDatasets(left_datasets=[dataset_left], right_datasets=[dataset_right])
```

![default](https://user-images.githubusercontent.com/120389559/221359482-93e1897f-2820-40da-bb99-c7bc057742bf.png)

### left_name

Determine left part datasets name.

**type:** `str`

**default value:** `None`

### right_name

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

![left_right](https://user-images.githubusercontent.com/120389559/221360077-aade1945-c38f-42f4-8e96-0e757fb1cc3d.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                                  Attributes and Methods                                                   | Description                                              |
| :-----------------------------------------------------------------------------------------------------------------------: | -------------------------------------------------------- |
| `set(left_datasets: List[DatasetInfo] = None, right_datasets: List[DatasetInfo] = None, left_name=None, right_name=None)` | Set `DatasetInfo` data in left and right part of widget. |
|                                                       `get_stat()`                                                        | Return datasets match statistics.                        |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/054_match_datasets/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/054_match_datasets/src/main.py)

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
dataset_left_id = int(os.environ["modal.state.slyDatasetId_left"])
dataset_left = api.dataset.get_info_by_id(id=dataset_left_id)

dataset_right_id = int(os.environ["modal.state.slyDatasetId_right"])
dataset_right = api.dataset.get_info_by_id(id=dataset_right_id)

dataset_left_id2 = int(os.environ["modal.state.slyDatasetId_left2"])
dataset_left2 = api.dataset.get_info_by_id(id=dataset_left_id2)

dataset_right_id2 = int(os.environ["modal.state.slyDatasetId_right2"])
dataset_right2 = api.dataset.get_info_by_id(id=dataset_right_id2)

dataset_left_id3 = int(os.environ["modal.state.slyDatasetId_left3"])
dataset_left3 = api.dataset.get_info_by_id(id=dataset_left_id3)

dataset_right_id3 = int(os.environ["modal.state.slyDatasetId_right3"])
dataset_right3 = api.dataset.get_info_by_id(id=dataset_right_id3)
```

### Initialize `MatchDatasets` widget

```python
match_datasets = MatchDatasets(
    left_datasets=[dataset_left, dataset_left2, dataset_left3],
    right_datasets=[dataset_right, dataset_right2, dataset_right3],
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

![layout](https://user-images.githubusercontent.com/120389559/221360552-53097a9d-585f-4391-99a4-d065636d8560.png)
