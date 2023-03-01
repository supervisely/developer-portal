# Empty

## Introduction

**`Empty`** widget is a simple placeholder widget that can be used to create empty spaces within a layout. 

It has no content or functionality of its own and is typically used to provide spacing or alignment within a larger widget or layout. This widget is particularly useful for creating clean, organized interfaces with clearly defined sections and layouts.




[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/empty)

## Function signature

```python
empty = Empty(widget_id=None)
```

## Parameters

| Parameters | Type |   Description    |
| :--------: | :--: | :--------------: |
| `widget_id`  | `str`  | ID of the widget |

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/022_empty/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/022_empty/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Empty, OneOf, Select, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Empty` widget

```python
empty = Empty()
```

### Prepare some widgets 

> This widget is useful in other widgets, for example,  `Select.Item`

```python
text = Text("Some text")
items = [
    Select.Item("item_1", content=Empty()),
    Select.Item("item_2", content=text)
]
select = Select(items=items)
one_of = OneOf(select)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Empty",
    content=empty,
)
layout = Container(widgets=[select, one_of])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

In this video demonstrated `Empty` (invisible) and `Text` widgets that we used in `Select` widget.

![empty-app](https://user-images.githubusercontent.com/79905215/219017739-1dc185a9-bb16-489a-b412-2dda94294398.gif)

