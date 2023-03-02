# RadioGroup

## Introduction

**`RadioGroup`** widget in Supervisely is a user interface element that allows users to create a group of mutually exclusive options that can be selected via radio buttons. With the RadioGroup widget, users can define a set of options, each with a corresponding radio button, and only one option (**`RadioGroup.Item`**) can be selected at a time. Overall, **`RadioGroup` ** widget is a valuable tool for simplifying user interactions and improving the usability of Supervisely apps.

## Function signature

```python
RadioGroup(
    items, size=None,
    direction="horizontal",
    gap=10,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218781501-c8849c3c-1070-4425-8b9f-e9cb9ad9a74b.gif" alt=""><figcaption></figcaption></figure>

## Parameters

|  Parameters |                    Type                   |           Description          |
| :---------: | :---------------------------------------: | :----------------------------: |
|   `items`   |          `List[RadioGroup.Item]`          |  List of `RadioGroup` content  |
|    `size`   | `Literal["large", "small", "mini", None]` |        `RadioGroup` size       |
| `direction` |    `Literal["vertical", "horizontal"]`    |  `RadioGroup` items direction  |
|    `gap`    |                   `int`                   | Gap between `RadioGroup` items |
| `widget_id` |                   `str`                   |        ID of the widget        |

### items

Determine list of `RadioGroup` items.

**type:** `List[RadioGroup.Item]`

### size

Determine `RadioGroup` size.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
radio_group = RadioGroup(items=items, size="large")
```

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

<figure><img src="https://user-images.githubusercontent.com/120389559/218785530-ddc74acc-8a88-4c52-9b8b-0bc5badbbe7b.png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="https://user-images.githubusercontent.com/120389559/218786092-16a0d2a3-3eaf-4945-8697-52ebca45dd11.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|        Attributes and Methods       | Description                                                       |
| :---------------------------------: | ----------------------------------------------------------------- |
|            `get_value()`            | Return selected `RadioGroup.Item` value.                          |
| `set(items: List[RadioGroup.Item])` | Set given list of `RadioGroup.Item` in `RadioGroup`.              |
|       `set_value(value: str)`       | Set given value by value of `RadioGroup.Item`.                    |
|           `@value_changed`          | Decorator function is handled when input `RadioGroup` is changed. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/controls/003\_radio\_group/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/controls/003\_radio\_group/src/main.py)

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
widgets = Container(widgets=[radio_group, one_of])

layout = Card(content=widgets, title="Radio group")
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218789546-34054150-6e4d-44cf-8673-f2e9e112b2e4.gif" alt=""><figcaption></figcaption></figure>
