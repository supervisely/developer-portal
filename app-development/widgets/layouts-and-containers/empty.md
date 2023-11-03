# Empty

## Introduction

**`Empty`** widget is a simple placeholder widget that can be used to create empty spaces within a layout.

It has no content or functionality of its own and is typically used to provide spacing or alignment within a larger widget or layout. This widget is particularly useful for creating clean, organized interfaces with clearly defined sections and layouts.

## Function signature

```python
empty = Empty(widget_id=None)
```

## Parameters

| Parameters  | Type  |               Description                |
| :---------: | :---: | :--------------------------------------: |
|   `style`   | `str` | Specifies an inline style for an element |
| `widget_id` | `str` |             ID of the widget             |

### style

Specifies an inline style for an element.

**type:** `str`

**default value:** ""

```python
empty = Empty(style="padding: 5px;")
```

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/layouts and containers/003\_empty/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/003\_empty/src/main.py)

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

> This widget is useful in other widgets, for example, `Select.Item`

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

<figure><img src="https://user-images.githubusercontent.com/79905215/223943970-f4338bd1-0f2b-4f6b-96d5-c2a3086aca0c.gif" alt=""><figcaption></figcaption></figure>
