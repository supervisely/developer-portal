# RandomSplitsTable

## Introduction

**`RandomSplitsTable`** widget in Supervisely allows users to create random splits of their data for training, validation, and testing purposes. The widget enables users to define the percentage of data they want to allocate to each split, and then randomly assigns images or annotations to each split. This widget is particularly useful for machine learning projects, as it allows users to easily manage their training, validation, and testing data without having to manually split the data themselves. `RandomSplitsTable` widget provides a flexible and convenient way for users to organize their data splits, and can be customized to match the requirements of their project. `RandomSplitsTable` widget is a valuable tool for improving the accuracy and efficiency of machine learning projects that require data splits.

## Function signature

```python
RandomSplitsTable(
    items_count,
    start_train_percent=80,
    disabled=False,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221407209-a8049b1b-4807-4104-a876-dce63ea8bbc2.gif" alt=""><figcaption></figcaption></figure>

## Parameters

|       Parameters      |  Type  |        Description       |
| :-------------------: | :----: | :----------------------: |
|     `items_count`     |  `int` | Number of items to split |
| `start_train_percent` |  `int` | Start `%` to split items |
|       `disabled`      | `bool` |      Disable widget      |
|      `widget_id`      |  `str` |     ID of the widget     |

### items\_count

Determine number of items to split.

**type:** `int`

```python
random_splits_table = RandomSplitsTable(items_count=777)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221407395-0afc810b-8048-446e-8ab5-f1ba678d2748.png" alt=""><figcaption></figcaption></figure>

### start\_train\_percent

Determine start `%` to split items. If `start_train_percent` not in range \[1; 99] raise `ValueError`.

**type:** `int`

**default value:** `80`

```python
random_splits_table = RandomSplitsTable(
    items_count=100,
    start_train_percent=10,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221407545-ec7300a6-2903-4104-b619-0efe30d6bfb7.png" alt=""><figcaption></figcaption></figure>

### disabled

Disable widget.

**type:** `bool`

**default value:** `False`

```python
random_splits_table = RandomSplitsTable(
    items_count=100,
    disabled=True,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221407635-d8b5f3a4-9a56-45e3-a881-c56fd3edb406.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                                                               |
| :--------------------: | ----------------------------------------------------------------------------------------- |
|  `get_splits_counts()` | Returns the result of separating items `{ "total": <int>, "train": <int>, "val": <int>}`. |
| `set_train_split_percent()` | Set the percentage of objects that will be assigned to the training sample (train) in RandomSplitsTable.                                                               |
| `get_train_split_percent()` | Get training split percent.                                                               |
|  `set_val_split_percent()`  | Set the percentage of objects that will be assigned to the validation sample (val) in RandomSplitsTable.                                                             |
|  `get_val_split_percent()`  | Get validation split percent.                                                             |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/tables/005\_random\_splits\_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/tables/005\_random\_splits\_table/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, RandomSplitsTable
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare items count

```python
project_id = sly.env.project_id()

project = api.project.get_info_by_id(project_id)
items_count = project.items_count
```

### Initialize `RandomSplitsTable` widget

```python
random_splits_table = RandomSplitsTable(
    items_count=items_count,
    start_train_percent=40,
)
```

### Create `Button` and `Text` widgets

```python
button = Button("Button")
text = Text()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Random Splits Table",
    content=Container([random_splits_table, button, text]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add function to control widget from code

```python
@button.click
def show_split_settings():
    text.text = random_splits_table.get_splits_counts()
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222740724-389a1e0e-9913-4d3b-a97e-be6f47f21c0e.gif" alt=""><figcaption></figcaption></figure>
