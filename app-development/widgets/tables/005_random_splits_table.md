# Random Splits Table

## Introduction

In this tutorial you will learn how to use `RandomSplitsTable` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/randomsplitstable)

## Function signature

```python
RandomSplitsTable(items_count, start_train_percent=80, disabled=False, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/221407209-a8049b1b-4807-4104-a876-dce63ea8bbc2.gif)

## Parameters

|      Parameters       |  Type  |       Description        |
| :-------------------: | :----: | :----------------------: |
|     `items_count`     | `int`  | Number of items to split |
| `start_train_percent` | `int`  | Start `%` to split items |
|      `disabled`       | `bool` |      Disable widget      |
|      `widget_id`      | `str`  |     Id of the widget     |

### items_count

Determine number of items to split.

**type:** `int`

```python
random_splits_table = RandomSplitsTable(items_count=777)
```

![items_count](https://user-images.githubusercontent.com/120389559/221407395-0afc810b-8048-446e-8ab5-f1ba678d2748.png)

### start_train_percent

Determine start `%` to split items. If `start_train_percent` not in range [1; 99] raise `ValueError`.

**type:** `int`

**default value:** `80`

```python
random_splits_table = RandomSplitsTable(items_count=100, start_train_percent=10)
```

![start_train_percent](https://user-images.githubusercontent.com/120389559/221407545-ec7300a6-2903-4104-b619-0efe30d6bfb7.png)

### disabled

Disable widget.

**type:** `bool`

**default value:** `False`

```python
random_splits_table = RandomSplitsTable(items_count=100, disabled=True)
```

![disabled](https://user-images.githubusercontent.com/120389559/221407635-d8b5f3a4-9a56-45e3-a881-c56fd3edb406.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                             |
| :--------------------: | --------------------------------------- |
| `get_splits_counts()`  | Returns the result of separating items. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/058_random_splits_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/058_random_splits_table/src/main.py)

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

### Initialize `RandomSplitsTable` widget

```python
random_splits_table = RandomSplitsTable(items_count=500, start_train_percent=40)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Random Splits Table",
    content=random_splits_table,
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/221407876-a58187dd-bd4c-4469-9bef-00030e6b6f93.gif)
