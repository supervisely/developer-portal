# Menu

## Introduction

**`Menu`** widget from Supervisely is a useful tool for organizing and navigating in Supervisely apps. With `Menu` widget, users can create custom menus that shows different parts and widgets of the app. These menus can be organized by groups.

## Function signature

```python
Menu(
    items=None,
    groups=None,
    index=None,
    width_percent=25,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/220916979-223e32ad-ed9a-4382-be74-148b2f792012.gif" alt=""><figcaption></figcaption></figure>

## Parameters

|    Parameters   |        Type        |                Description                |
| :-------------: | :----------------: | :---------------------------------------: |
|     `items`     |  `List[Menu.Item]` |           List of `Menu` `Items`          |
|     `groups`    | `List[Menu.Group]` |          List of `Menu` `Groups`          |
|     `index`     |        `str`       | Sets the active `Menu` at the given index |
| `width_percent` |        `int`       |   Width of the left part of `Menu` in %   |
|   `widget_id`   |        `str`       |              ID of the widget             |

### items

Determine list of menu `Items`.

**type:** `List[Menu.Item]`

**default value:** `None`

```python
text = Text(text="text", status="success")
done_label = DoneLabel("done")

items = [
    Menu.Item(title="menu item 1", content=text),
    Menu.Item(title="menu item 2", content=done_label),
]
menu = Menu(items=items)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224021334-69b0ac2a-219c-49ba-9614-71fef96825d6.gif" alt=""><figcaption></figcaption></figure>

### groups

Determine list of menu `Groups`.

**type:** `List[Menu.Group]`

**default value:** `None`

```python
l = Text(text="left part", status="success")
items = [
    Select.Item(label="CPU", value="cpu"),
    Select.Item(label="GPU 0", value="cuda:0"),
    Select.Item(value="option3"),
]
r = Select(items=items, filterable=True)

g1_items = [
    Menu.Item(title="m1", content=r),
    Menu.Item(title="m2", content=l),
]
g2_items = [
    Menu.Item(title="m3", content=Empty()),
    Menu.Item(title="m4"),
]
g1 = Menu.Group("group_1", g1_items)
g2 = Menu.Group("group_2", g2_items)
menu = Menu(groups=[g1, g2])
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224022265-631fb03e-913d-4e1b-86b6-76ed0c04148b.gif" alt=""><figcaption></figcaption></figure>

### index

Sets the active `Menu` at the given index.

**type:** `str`

**default value:** `None`

```python
text = Text(text="text", status="success")
done_label = DoneLabel("done")

items = [
    Menu.Item(title="menu item 1", content=text),
    Menu.Item(title="menu item 2", content=done_label),
]
menu = Menu(items=items, index="menu item 2")
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224022882-8d541059-1f03-46f4-81c1-7602c79e766b.png" alt=""><figcaption></figcaption></figure>

### width\_percent

Determines width of the left part of `Menu` in %.

**type:** `int`

**default value:** `25`

```python
text = Text(text="text", status="success")
done_label = DoneLabel("done")

items = [
    Menu.Item(title="menu item 1", content=text),
    Menu.Item(title="menu item 2", content=done_label),
]
menu = Menu(items=items, width_percent=50)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224023137-076cb0c9-1875-4f65-a740-6f498af5f166.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/layouts and containers/007\_menu/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/007\_menu/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Empty, Menu, Select, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Menu` items

```python
text = Text(text="text", status="success")
done_label = DoneLabel("done")

g1_items = [
    Menu.Item(title="menu item 1", content=text),
    Menu.Item(title="menu item 2", content=done_label),
]

g2_items = [
    Menu.Item(title="menu item 3", content=Empty()),
    Menu.Item(title="menu item 4"),
]
```

### Initialize `Menu` groups

```python
group_1 = Menu.Group("group 1", g1_items)
group_2 = Menu.Group("group 2", g2_items)
```

### Initialize `Menu` widget

```python
menu = Menu(groups=[group_1, group_2])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=menu)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224023706-c742c1ff-9d5c-4cf0-9a51-564951e1f736.png" alt=""><figcaption></figcaption></figure>
