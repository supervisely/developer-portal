# BindedInputNumber

## Introduction

**`BindedInputNumber`** widget in Supervisely is a user interface element that allows users to input two numerical values and customize their behavior using the proportional, min, and max properties. With the BindedInputNumber widget, users can fine-tune specific parameters within supervisely apps that require two numerical inputs, such as defining a rectangular region of interest by specifying the `x` and `y` coordinates and the `width` and `height`. The `BindedInputNumber` widget provides a convenient and flexible way to input and manage these values, with customizable behavior to ensure accurate and precise inputs.

## Function signature

```python
BindedInputNumber(
    width=256,
    height=256,
    min=1,
    max=10000,
    proportional=False,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219939414-e601fd63-3cdb-420e-99c4-706b48710a41.png" alt=""><figcaption></figcaption></figure>

## Parameters

|   Parameters   |  Type  |                     Description                    |
| :------------: | :----: | :------------------------------------------------: |
|     `width`    |  `int` |                     Width value                    |
|    `height`    |  `int` |                    Weight value                    |
|      `min`     |  `int` |                Minimum allowed value               |
|      `max`     |  `int` |                Maximum allowed value               |
| `proportional` | `bool` | Synchronize changes in width and height parameters |
|   `widget_id`  |  `str` |                  ID of the widget                  |

### width

Determine width value.

**type:** `int`

**default value:** `256`

```python
binded_input_number = BindedInputNumber(width=15)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219939697-abd65ad3-b85d-410f-8424-15fd2e34bff8.png" alt=""><figcaption></figcaption></figure>

### height

Determine height value.

**type:** `int`

**default value:** `256`

```python
binded_input_number = BindedInputNumber(height=15)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219939756-0791330e-e047-49be-abba-a54957ebe322.png" alt=""><figcaption></figcaption></figure>

### min

Minimum allowed value.

**type:** `int`

**default value:** `1`

### max

Maximum allowed value.

**type:** `int`

**default value:** `10000`

```python
binded_input_number = BindedInputNumber(
    width=12,
    height=12,
    min=10,
    max=14,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221354865-018d95d0-c540-462c-888f-b9b1fc71a389.gif" alt=""><figcaption></figcaption></figure>

### proportional

Synchronize changes in width and height parameters.

**type:** `bool`

**default value:** `false`

```python
binded_input_number = BindedInputNumber(proportional=True)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221354956-e537a019-80ec-4964-b4b3-fbbf5052747b.gif" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|      Attributes and Methods      | Description                             |
| :------------------------------: | --------------------------------------- |
| `value(width: int, height: int)` | Set widgets `width` and `height` filed. |
|           `get_value()`          | Get input `width` and `height` values.  |
|    `proportional(value: bool)`   | Set `proportional` value.               |
|         `min(value: int)`        | Set `min` value.                        |
|         `max(value: int)`        | Set `max` value.                        |
|            `disable()`           | Disable widget.                         |
|            `enable()`            | Enable widget.                          |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/input/004\_binded\_input\_number/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/input/004\_binded\_input\_number/src/main.py)

### Import libraries

```python
import supervisely as sly
from supervisely.app.widgets import Container, BindedInputNumber, Card
```

### Initialize `BindedInputNumber` widget

```python
binded_input_number = BindedInputNumber()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Binded Input Number",
    content=Container(widgets=[binded_input_number]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219942364-cb93b2fb-ca7b-40e7-8d8f-591525092bf1.gif" alt=""><figcaption></figcaption></figure>
