# Select

## Introduction

This Supervisely widget allows you to create a dropdown menu that lets users select an option from a list of choices. The `Select` widget has event handler that is triggered when the user selects an option from the dropdown menu. This can be useful for applications that require users to take an action based on the selected option, such as filtering content or displaying specific information.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/select)

## Function signature

```python
Select(items=None, groups=None, filterable=False, placeholder="select", size=None, multiple=False, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/217835655-5a888104-dd74-4dc9-b68f-88ab956898f3.png)

## Parameters

|  Parameters   |                   Type                    |             Description              |
| :-----------: | :---------------------------------------: | :----------------------------------: |
|    `items`    |            `List[Select.Item]`            |    List of `Select.Item` widgets     |
|   `groups`    |           `List[Select.Group]`            |    List of `Select.Group` widgets    |
| `filterable`  |                  `bool`                   |    Whether `Select` is filterable    |
| `placeholder` |                   `str`                   |             Placeholder              |
|    `size`     | `Literal["large", "small", "mini", None]` |            Size of input             |
|  `multiple`   |                  `bool`                   | Whether multiple-select is activated |
|  `widget_id`  |                   `str`                   |           Id of the widget           |

### items

Determine list of `Select.Item` widgets.

**type:** `List[Select.Item]`

**default value:** `None`

### groups

Determine list of `Select.Group` widgets.

**type:** `List[Select.Group]`

**default value:** `None`

**Function signature**

Prepare select items and groups:

```python
animals_domestic = [
    Select.Item(value="cat", label="cat"),
    Select.Item(value="dog", label="dog"),
]
animals_wild = [
    Select.Item(value="squirrel", label="squirrel"),
]
```

Initialize widget with given items:

```python
select_items = Select(
    items=animals_domestic + animals_wild,
    filterable=True,
)
```

![items](https://user-images.githubusercontent.com/120389559/217835655-5a888104-dd74-4dc9-b68f-88ab956898f3.png)

or initialize widget with given groups of items:

```python
groups = [
    Select.Group(label="domestic", items=animals_domestic),
    Select.Group(label="wild", items=animals_wild),
]

select_groups = Select(groups=groups)
```

![groups](https://user-images.githubusercontent.com/120389559/217837265-105f89c7-3c12-4232-94c8-59692bfb160a.png)

### filterable

Whether `Select` is filterable.

**type:** `bool`

**default value:** `false`

### placeholder

Placeholder.

**type:** `str`

**default value:** `select`

### size

Size of input.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
select = Select(items=animals_domestic)
select_mini = Select(items=animals_domestic, size="mini")
select_small = Select(items=animals_domestic, size="small")
select_large = Select(items=animals_domestic, size="large")

card = Card(content=Container(widgets=[select, select_mini, select_small, select_large]))
```

![size](https://user-images.githubusercontent.com/120389559/218707061-ce71b019-7db3-4de6-bdcf-06f7b3ca72d7.png)

### multiple

Whether multiple-select is activated.

**type:** `bool`

**default value:** `false`

```python
select_items = Select(
    items=animals_domestic + animals_wild,
    multiple=True,
)
```

![multiple](https://user-images.githubusercontent.com/120389559/218096915-b300c3d6-7a17-4cca-befe-36a2ee4828de.gif)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                          Attributes and Methods                           | Description                                 |
| :-----------------------------------------------------------------------: | ------------------------------------------- |
|                               `get_value()`                               | Return selected item value.                 |
| `set(items: List[Select.Item] = None, groups: List[Select.Group] = None)` | Set `Select` input items or group of items. |
|                               `get_items()`                               | Return list of items from widget.           |
|                             `value_changed()`                             | Handled when input value is changed.        |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/009_select/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/009_select/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Select
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare items and groups of items for `Select` widget using `Select.Item` and `Select.Group`

```python
animals_domestic = [
    Select.Item(value="cat", label="cat"),
    Select.Item(value="dog", label="dog"),
    Select.Item(value="horse", label="horse"),
    Select.Item(value="sheep", label="sheep"),
]
animals_wild = [
    Select.Item(value="squirrel", label="squirrel"),
]
```

```python
groups = [
    Select.Group(label="domestic", items=animals_domestic),
    Select.Group(label="wild", items=animals_wild),
]
```

### Initialize `Select` widget

```python
select_items = Select(
    items=animals_domestic + animals_wild,
    filterable=True,
)
```

```python
select_groups = Select(groups=groups)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select",
    content=Container(widgets=[select_items, select_groups]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218097482-c5d94009-036c-43dd-85c4-f8f6c413f984.gif)
