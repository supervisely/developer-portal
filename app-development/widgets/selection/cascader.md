# Cascader

## Introduction

**`Cascader`** is a dropdown list with hierarchical options.

## Function signature

```python
Cascader(
    items=None,
    filterable=False,
    placeholder="select",
    size=None,
    expand_trigger="click",
    disabled=False,
    clearable=False,
    show_all_levels=True,
    change_on_select=False,
    selected_options=None,
    widget_id=None,
)
```

Example of input data we will use.

```python
pussy_black_cat = Cascader.Item(value="pussy cat", label="pussy cat")
smooth_haired_cat = Cascader.Item(value="smooth-haired cat", label="smooth-haired cat")
black_cat = Cascader.Item(
    value="black cat", label="black cat", children=[pussy_black_cat, smooth_haired_cat]
)
white_cat = Cascader.Item(value="white cat", label="white cat")
red_cat = Cascader.Item(value="red cat", label="red cat")

angry_dog = Cascader.Item(value="angry dog", label="angry dog", disabled=True)
kind_dog = Cascader.Item(value="kind dog", label="kind dog")

animals = [
    Cascader.Item(value="cat", label="cat", children=[black_cat, white_cat, red_cat]),
    Cascader.Item(value="dog", label="dog", children=[angry_dog, kind_dog]),
    Cascader.Item(value="horse", label="horse"),
    Cascader.Item(value="sheep", label="sheep"),
]

select_items = Cascader(items=animals)
```

![](https://user-images.githubusercontent.com/120389559/226871849-1cb5f79a-6a80-438f-b95e-6983a8a5feba.gif)

## Parameters

|     Parameters     |                    Type                   |                            Description                           |
| :----------------: | :---------------------------------------: | :--------------------------------------------------------------: |
|       `items`      |           `List[Cascader.Item]`           |                       Input `Cascader` data                      |
|    `filterable`    |                   `bool`                  |                Whether the options can be searched               |
|    `placeholder`   |                   `str`                   |                         Input placeholder                        |
|       `size`       | `Literal["large", "small", "mini", None]` |                           Size of input                          |
|  `expand_trigger`  |        `Literal["click", "hover"]`        |              Trigger mode of expanding current item              |
|     `clearable`    |                   `bool`                  |               Whether selected value can be cleared              |
|  `show_all_levels` |                   `bool`                  | Whether to display all levels of the selected value in the input |
| `change_on_select` |                   `bool`                  |       Whether selecting an option of any level is permitted      |
| `selected_options` |                `List[str]`                |                      Current selected option                     |
|     `widget_id`    |                   `str`                   |                         ID of the widget                         |

### items

Determine input `Cascader` data.

**type:** `List[Cascader.Item]`

**default value:** `None`

### filterable

Determine whether the options can be searched.

**type:** `bool`

**default value:** `False`

```python
select_items = Cascader(items=animals, filterable=True)
```

![](https://user-images.githubusercontent.com/120389559/226874587-f906fe5d-6b1a-458f-8941-056fe857df17.gif)

### placeholder

Determine input placeholder.

**type:** `str`

**default value:** `select`

```python
select_items = Cascader(items=animals, placeholder="Choose the pet")
```

![](https://user-images.githubusercontent.com/120389559/226875192-41b07853-71c6-41a3-9f4d-b382eb84ad07.png)

### size

Determine size of input.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
select_items = Cascader(items=animals)
select_items_mini = Cascader(items=animals, size="mini")
select_items_small = Cascader(items=animals, size="small")
select_items_large = Cascader(items=animals, size="large")
```

![](https://user-images.githubusercontent.com/120389559/226876380-84ca158a-b88e-4de4-937a-a71bceb07f81.png)

### expand\_trigger

Trigger mode of expanding current item.

**type:** `Literal["click", "hover"]`

**default value:** `"click"`

```python
select_items = Cascader(items=animals, expand_trigger="hover")
```

![](https://user-images.githubusercontent.com/120389559/226878119-6f1dd044-752a-4330-8093-4f4ce47fe113.gif)

### clearable

Determine whether selected value can be cleared.

**type:** `bool`

**default value:** `False`

```python
select_items = Cascader(items=animals, clearable=True)
```

![](https://user-images.githubusercontent.com/120389559/226880844-5d624845-92cf-4742-9919-47d6730e1d16.gif)

### show\_all\_levels

Determine whether to display all levels of the selected value in the input.

**type:** `bool`

**default value:** `True`

```python
select_items = Cascader(items=animals, show_all_levels=False)
```

![](https://user-images.githubusercontent.com/120389559/226881351-511fe63a-11cb-4e84-a1c6-d704e3aed347.gif)

### change\_on\_select

Determine whether selecting an option of any level is permitted.

**type:** `bool`

**default value:** `False`

```python
select_items = Cascader(items=animals, change_on_select=True)
```

![](https://user-images.githubusercontent.com/120389559/226882071-70e9b95d-b430-4c0c-b614-3df06caa8fa9.gif)

### selected\_options

Determine current selected option.

**type:** `List[str]`

**default value:** `None`

```python
select_items = Cascader(items=animals, selected_options=["horse"])
```

![](https://user-images.githubusercontent.com/120389559/226882894-11d507a4-00ef-4cf8-8a56-37898a88a9e2.png)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|          Attributes and Methods         | Description                                         |
| :-------------------------------------: | --------------------------------------------------- |
|              `get_value()`              | Return `Cascader` selected value.                   |
|      `set_value(value: List[str])`      | Set `Cascader` selected value.                      |
|              `get_items()`              | Return `Cascader` items.                            |
|               `get_item()`              | Return `Cascader.Item` item.                        |
| `set_items(value: List[Cascader.Item])` | Set `Cascader` items.                               |
| `add_items(value: List[Cascader.Item])` | Add items in `Cascader`.                            |
|           `expand_to_hover()`           | Set `expand_trigger` to `hover` mode.               |
|           `expand_to_click()`           | Set `expand_trigger` to `click` mode.               |
|             `set_disabled()`            | Set `disabled` to `True`.                           |
|             `set_unabled()`             | Set `disabled` to `False`.                          |
|             `@value_changed`            | Decorator function to handle selected value change. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/selection/015\_cascader/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/015\_cascader/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Cascader, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare items for cascader

```python
pussy_black_cat = Cascader.Item(value="pussy cat", label="pussy cat")
smooth_haired_cat = Cascader.Item(value="smooth-haired cat", label="smooth-haired cat")
black_cat = Cascader.Item(
    value="black cat", label="black cat", children=[pussy_black_cat, smooth_haired_cat]
)
white_cat = Cascader.Item(value="white cat", label="white cat")
red_cat = Cascader.Item(value="red cat", label="red cat")

angry_dog = Cascader.Item(value="angry dog", label="angry dog", disabled=True)
kind_dog = Cascader.Item(value="kind dog", label="kind dog")

animals = [
    Cascader.Item(value="cat", label="cat", children=[black_cat, white_cat, red_cat]),
    Cascader.Item(value="dog", label="dog", children=[angry_dog, kind_dog]),
    Cascader.Item(value="horse", label="horse"),
    Cascader.Item(value="sheep", label="sheep"),
]
```

### Initialize `Cascader` and `Text` widgets

```python
select_items = Cascader(items=animals)

text = Text()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
card = Card(
    title="Cascader",
    content=Container(widgets=[select_items, text]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=card)
```

### Add functions to control widgets from python code

```python
@select_items.value_changed
def show_item(res):
    info = f"You choise: {'/'.join(str(x) for x in res)} item"
    text.set(text=info, status="info")
```

<div align="center">

<img src="https://user-images.githubusercontent.com/120389559/226883842-8d55d09a-fdaf-421d-a720-bf20ef64d164.gif" alt="">

</div>
