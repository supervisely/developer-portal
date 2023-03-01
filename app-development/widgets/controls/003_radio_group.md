# Radio group

## Introduction

In this tutorial you will learn how to use `RadioGroup`, `RadioGroup.Item` widgets in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/radiogroup)

## Function signature

```python
RadioGroup(items, size=None, direction="horizontal", gap=10, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218781501-c8849c3c-1070-4425-8b9f-e9cb9ad9a74b.gif)

## Parameters

| Parameters  |                   Type                    |             Description              |
| :---------: | :---------------------------------------: | :----------------------------------: |
|   `items`   |          `List[RadioGroup.Item]`          |     List of `RadioGroup` content     |
|   `size`    | `Literal["large", "small", "mini", None]` | `RadioGroup` size (large/small/mini) |
| `direction` |    `Literal["vertical", "horizontal"]`    |     `RadioGroup` items direction     |
|    `gap`    |                   `int`                   |    Gap between `RadioGroup` items    |
| `widget_id` |                   `str`                   |           Id of the widget           |

### items

Determine list of `RadioGroup` items.

**type:** `List[RadioGroup.Item]`

### size

Determine `RadioGroup` size.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

### direction

Determine `RadioGroup` items direction.

**type:** `Literal["vertical", "horizontal"]`

**default value:** `horizontal`

```python
items = [
    RadioGroup.Item(value="cat"),
    RadioGroup.Item(value="dog"),
    RadioGroup.Item(value="bird"),
]

radio_group = RadioGroup(items=items, direction="vertical")
```

![direction](https://user-images.githubusercontent.com/120389559/218785530-ddc74acc-8a88-4c52-9b8b-0bc5badbbe7b.png)

### gap

Determine gap between `RadioGroup` items.

**type:** `int`

**default value:** `10`

```python
items = [
    RadioGroup.Item(value="cat"),
    RadioGroup.Item(value="dog"),
    RadioGroup.Item(value="bird"),
]

radio_group = RadioGroup(items=items, gap=100)
```

![gap](https://user-images.githubusercontent.com/120389559/218786092-16a0d2a3-3eaf-4945-8697-52ebca45dd11.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|       Attributes and Methods        | Description                                      |
| :---------------------------------: | ------------------------------------------------ |
|            `get_value()`            | Return selected `RadioGroup.Item` value.         |
|          `value_changed()`          | Handled when input `RadioGroup.Item` is changed. |
| `set(items: List[RadioGroup.Item])` | Set given `RadioGroup.Item` in `RadioGroup`.     |
|         `set_value(value)`          | Set given value in `RadioGroup.Item`.            |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/044_radio_group/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/044_radio_group/src/main.py)

### Import libraries

```python
import supervisely as sly
from supervisely.app.widgets import RadioGroup, Text, OneOf, Container, Card
```

### Initialize `RadioGroup.Item` to use in `RadioGroup`

```python
items=[
    RadioGroup.Item(value="cat", label="Cat", content=Text(text='Cat says "Meow!"')),
    RadioGroup.Item(value="dog", label="Dog", content=Text(text='Dot says "Woof!"')),
    RadioGroup.Item(value="bird", label="Bird", content=Text(text='Bird says "Tweet!"')),
]
```

### Initialize `RadioGroup` widget

```python
radio_group = RadioGroup(items=items, size="large")
```

### Initialize `OneOf` widget to have ability to choose `RadioGroup.Item` content

```python
one_of = OneOf(radio_group)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
widgets = Container(
    widgets=[radio_group, one_of],
)

layout = Card(content=widgets, title="Radio group")
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218789546-34054150-6e4d-44cf-8673-f2e9e112b2e4.gif)
