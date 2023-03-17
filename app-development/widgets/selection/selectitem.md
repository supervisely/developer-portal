# SelectItem

## Introduction

**`SelectItem`** widget can show selector for project items (depending on project type:`image`, `video`, `volume`, `point_cloud` or `point_cloud_episode`), clicking on it can be processed from python code. In this tutorial you will learn how to use **`SelectItem` ** widget in Supervisely app.

## Function signature

```python
SelectItem(dataset_id=None, show_label=True, size=None, widget_id=None)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218035492-9a07432d-8fb0-4dad-b5ff-ccd8ce03a137.png" alt=""><figcaption></figcaption></figure>

## Parameters

|  Parameters  |                    Type                   |        Description       |
| :----------: | :---------------------------------------: | :----------------------: |
| `dataset_id` |                   `int`                   |       `Dataset` ID       |
|   `compact`  |                   `bool`                  | Show only dataset select |
| `show_label` |                   `bool`                  |        Show label        |
|    `size`    | `Literal["large", "small", "mini", None]` |       Selector size      |
|  `widget_id` |                   `str`                   |     ID of the widget     |

### dataset\_id

Determine Supervisely dataset from which items will be selected.

**type:** `int`

**default value:** `None`

```python
select_item = SelectItem(dataset_id=dataset_id)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218035699-aa403402-6a7d-41af-a93c-ffb0b4f2df7c.png" alt=""><figcaption></figcaption></figure>

### compact

Show only `Dataset` select.

**type:** `bool`

**default value:** `true`

```python
select_item = SelectItem(dataset_id=dataset_id, compact=False)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221548050-4e7707fb-a665-43c6-a7db-8f969b212c63.png" alt=""><figcaption></figcaption></figure>

### show\_label

Determine show text `Item` on widget or not.

**type:** `bool`

**default value:** `True`

```python
select_item = SelectItem(dataset_id=dataset_id, show_label=False)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218035951-70b5d164-d7f4-44a2-85f8-4da65c112cae.png" alt=""><figcaption></figcaption></figure>

### size

Size of input.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
select_item = SelectItem(dataset_id=dataset_id, show_label=False)
select_mini = SelectItem(dataset_id=dataset_id, show_label=False, size="mini")
select_small = SelectItem(dataset_id=dataset_id, show_label=False, size="small")
select_large = SelectItem(dataset_id=dataset_id, show_label=False, size="large")
card = Card(
    title="Select Item",
    content=Container(widgets=[select_item, select_mini, select_small, select_large]),
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218725835-a36971d3-cc88-4169-9366-b7b5b383486e.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                  Attributes and Methods                  | Description                       |
| :------------------------------------------------------: | --------------------------------- |
|                    `get_selected_id()`                   | Return selected item id.          |
| `refresh_items(dataset_id: int = None, limit: int = 50)` | Update items by given dataset id. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/selection/006\_select\_item/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/006\_select\_item/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, SelectItem
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare `Dataset` ID

```python
dataset_id = sly.env.dataset_id()
```

### Initialize `SelectItem` widget

```python
select_item = SelectItem(dataset_id=dataset_id)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select Item",
    content=Container(widgets=[select_item]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218036360-09d6f530-42c7-43bd-a2f7-05d7d3f6f252.png" alt=""><figcaption></figcaption></figure>
