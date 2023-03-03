# Switch

## Introduction

**`Switch`** widget in Supervisely is a user interface element that allows users to toggle between two states, such as on/off or true/false. With the Switch widget, users can easily turn on or off a specific feature or setting. The Switch widget is useful for creating project dashboards that require users to toggle specific options quickly and easily. Users can customize the appearance and behavior of the Switch widget to match their project requirements, making it a flexible tool for creating user interfaces in Supervisely apps. Overall, the Switch widget is a simple yet effective tool for improving the usability and functionality of Supervisely apps.

## Function signature

```python
Switch(
    switched=False,
    width=58,
    on_text="ON",
    off_text="OFF",
    on_color=None,
    off_color=None,
    on_content=None,
    off_content=None,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222439328-eb096aca-5c22-4ef2-8410-7339580c6178.gif" alt=""><figcaption></figcaption></figure>

## Parameters

|   Parameters  |   Type   |                       Description                       |
| :-----------: | :------: | :-----------------------------------------------------: |
|   `switched`  |  `bool`  |             Determine `Switch` is ON or OFF             |
|    `width`    |   `int`  |                    Width of `Switch`                    |
|   `on_text`   |   `str`  |         Text displayed when `Switch` in ON state        |
|   `off_text`  |   `str`  |        Text displayed when `Switch` in OFF state        |
|   `on_color`  |   `str`  |        Background color when `Switch` in ON state       |
|  `off_color`  |   `str`  |       Background color when `Switch` in OFF state       |
|  `on_content` | `Widget` |  Determine active `Widget` when `Switch` is in ON state |
| `off_content` | `Widget` | Determine active `Widget` when `Switch` is in OFF state |
|  `widget_id`  |   `str`  |                     ID of the widget                    |

### switched

Determine `Switch` is ON or OFF.

**type:** `bool`

**default value:** `false`

```python
switch = Switch(switched=True)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222439526-3fc99d57-7a29-4490-8fd8-66369e3c6be2.png" alt=""><figcaption></figcaption></figure>

### width

Determine `Switch` width.

**type:** `int`

**default value:** `58`

```python
switch = Switch(width=150)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222440545-87e0fab8-c1a2-4c2d-b5c9-cfc4e3ec3a48.png" alt=""><figcaption></figcaption></figure>

### on\_text

Determine text displayed when `Switch` in ON state.

**type:** `str`

**default value:** `ON`

### off\_text

Determine text displayed when `Switch` in OFF state.

**type:** `str`

**default value:** `OFF`

```python
switch = Switch(
    on_text="on example",
    off_text="off example",
    width=140,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222441017-7585e940-6d66-4b49-a545-11df5524de22.gif" alt=""><figcaption></figcaption></figure>

### on\_color

Determine background color when `Switch` in ON state.

**type:** `str`

**default value:** `None`

```python
switch = Switch(on_color="#FF0000")
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222441521-db70503f-f010-4df8-9e60-4a7422247c6b.png" alt=""><figcaption></figcaption></figure>

### off\_color

Determine background color when `Switch` in OFF state.

**type:** `str`

**default value:** `None`

```python
switch = Switch(off_color="#7F00FF")
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222441494-76bbedad-6a69-4c7b-b674-e3c4cf84ad1e.png" alt=""><figcaption></figcaption></figure>

### on\_content

Determine active `Widget` when `Switch` in ON state.

**type:** `Widget`

**default value:** `None`

### off\_content

Determine active `Widget` when `Switch` in OFF state.

**type:** `Widget`

**default value:** `None`

```python
switch = Switch(
    on_content=Text("ON Content"),
    off_content=Text("OFF Content"),
)
switch_one_of = OneOf(switch)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222442744-b1df7f05-07a3-4a9c-bb9a-6769cb1731f3.gif" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|    Attributes and Methods   | Description                                                  |
| :-------------------------: | ------------------------------------------------------------ |
|       `is_switched()`       | Check `Switch` state.                                        |
|            `on()`           | Set `Switch` in ON state.                                    |
|           `off()`           | Set `Switch` in OFF state.                                   |
|        `get_width()`        | Return `Switch` width.                                       |
|       `get_on_text()`       | Return `Switch` ON text.                                     |
|       `set_on_text()`       | Set `Switch` ON text.                                        |
|       `get_off_text()`      | Return `Switch` OFF text.                                    |
|  `set_off_text(value: str)` | Set `Switch` OFF text.                                       |
|       `get_on_color()`      | Return `Switch` ON color.                                    |
|  `set_on_color(value: str)` | Set `Switch` ON color.                                       |
|      `get_off_color()`      | Return `Switch` OFF color.                                   |
| `set_off_color(value: str)` | Set `Switch` OFF color.                                      |
|       `@value_changed`      | Decorator function is handled when `Switch` value is change. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/controls/004\_switch/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/controls/004\_switch/src/main.py)

### Import libraries

```python
import supervisely as sly
from supervisely.app.widgets import Container, Switch, OneOf, Text, Card
```

### Initialize `Switch` widget

```python
switch = Switch(
    on_content=Text("ON Content"),
    off_content=Text("OFF Сontent"),
)
```

### Initialize `OneOf` widget to switch between `Cards`

```python
switch_one_of = OneOf(switch)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
layout = Container(widgets=[
    Card(title="Switch", content=switch),
    Card(title="OneOf", content=switch_one_of),
])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222442744-b1df7f05-07a3-4a9c-bb9a-6769cb1731f3.gif" alt=""><figcaption></figcaption></figure>
