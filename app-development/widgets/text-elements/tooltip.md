# Tooltip

## Introduction

**`Tooltip`** widget in Supervisely is a user interface element that displays prompt information for mouse hover.
Uses like a wrapper around a main UI elements, such as **`Button`** for example.

## Function signature

```python
Tooltip(
    text="Tooltip text",
    widget=Button("Push")
)
```

<img height="260" src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/57998637/7b3fe0aa-fdde-41cd-bf4f-ee6e56a14db5" alt="default" />

## Parameters

|   Parameters    |                    Type                     |                                                                                              Description                                                                                              |
| :-------------: | :-----------------------------------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|     `text`      |           `Union[str, List[str]]`           |                                                      Tooltip text. For a multi-line view, use `List[str]` with each line as a value in the list                                                       |
|    `widget`     |                  `Widget`                   |                                                                  `Widget` of the UI element for which the tooltip will be displayed                                                                   |
|  `color_theme`  |         `Literal["dark", "light"]`          |                                                                                 One of `"dark", "light"` color theme                                                                                  |
|   `placement`   |      `Literal["top","top-start", ..]`       | One of `"top","top-start","top-end","bottom","bottom-start","bottom-end","left","left-start","left-end","right","right-start","right-end"` - place around the element where tooltip will be displayed |
|    `offset`     |                    `int`                    |                                                                                    Offset of the tooltip in pixels                                                                                    |
|  `transition`   | `Literal["el-fade-in-linear","el-fade-in"]` |                                                      One of `"el-fade-in-linear", "el-fade-in"` describes the disappearance animation for widget                                                      |
| `visible_arrow` |                   `bool`                    |                                                            Determines whether the tooltip should have an arrow pointing to the item or not                                                            |
|  `open_delay`   |                    `int`                    |                                                                                     Display delay in milliseconds                                                                                     |
|   `enterable`   |                   `bool`                    |                                                                    Determines whether the cursor can enter the tooltip area or not                                                                    |
|  `hide_after`   |                    `int`                    |                                     Hide delay in milliseconds. With the default value `0`, it will not be hidden as long as the mouse is inside the UI element.                                      |
|   `widget_id`   |                    `str`                    |                                                                                           ID of the widget                                                                                            |

### text

Tooltip text. For a multi-line view, use `List[str]` with each line as a value in the list

**type:** `Union[str, List[str]]`

Example with `['Tooltip text line 1', 'Tooltip text line 2']` :

<img height="260" src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/57998637/835c3abe-1a6e-42d1-a1c4-b8de9e9baa16" alt="multi-line" />

### widget

The UI element for which the tooltip will be displayed.

**type:** `Widget`

```python
widget=Checkbox("Set option")
```

<img height="260" src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/57998637/1d66de46-d559-4e27-b6d9-e30957fd52cb" alt="element" />

### color_theme

Color theme of Tooltip widget.

**type:** `Literal["dark", "light"]`

**default value:** `"dark"`

<img height="260" src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/57998637/c9d356da-80d7-4282-8b6d-ff0cda7c73a9" alt="themes" />

### placement

Place around the element where tooltip will be displayed.
Must be one of `"top","top-start","top-end","bottom","bottom-start","bottom-end","left","left-start","left-end","right","right-start","right-end"` values.

**type:** `Literal["top","top-start", ..]`

**default value:** `"bottom"`

<img height="400" src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/57998637/8b6cb571-5344-41bf-b2c5-907740f8f41e" alt="themes" />

### offset

Offset of the Tooltip in pixels.

**type:** `int`

**default value:** `0`

### transition

Describes the disappearance animation for widget.
Must be one of `"el-fade-in-linear", "el-fade-in"` values.

**type:** `Literal["el-fade-in-linear","el-fade-in"]`

**default value:** `"el-fade-in-linear"`

### visible_arrow

Determines whether the tooltip should have an arrow pointing to the item or not.

**type:** `bool`

**default value:** `True`

Example with `False` :

<img height="260" src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/57998637/ccf067aa-5585-49e4-97fb-49513762ad6d" alt="arrow_hide" />

### open_delay

Display delay in milliseconds.

**type:** `int`

**default value:** `0`

Example with `2000` value:

<img height="260" src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/57998637/35363704-1c8f-4d20-84a4-940125a5fff8" alt="delay_2sec" />

### enterable

Determines whether the cursor can enter the tooltip area or not.

**type:** `bool`

**default value:** `True`

### hide_after

Hide delay in milliseconds.
With the default value, it will not be hidden as long as the mouse is inside the UI element.

**type:** `int`

**default value:** `0`

Example with `2000` value:

<img height="260" src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/57998637/d7a108b4-9034-4d6e-8f34-250891ac02bf" alt="delay_4sec" />

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                                                         Attributes and Methods                                                                         | Description                                                                                        |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------: | -------------------------------------------------------------------------------------------------- |
|                                                                `set_text(text: Union[str, List[str]])`                                                                 | Set text for tooltip. For a multi-line text, use `List[str]` with each line as a value in the list |
| `set_placement(placement: Literal["top","top-start","top-end","bottom","bottom-start","bottom-end","left","left-start","left-end","right","right-start","right-end"])` | Set place around the element where tooltip will be displayed                                       |
|                                                                       `set_offset(offset: int)`                                                                        | Set offset of the tooltip in pixels                                                                |
|                                                `set_transition(transition: Literal["el-fade-in-linear", "el-fade-in"])`                                                | Set animation for widget disappearance                                                             |
|                                                              `set_arrow_visibility(visible_arrow: bool)`                                                               | Switch off or on the tooltip arrow pointing to the item                                            |
|                                                                   `set_open_delay(open_delay: int)`                                                                    | Set the display delay in milliseconds                                                              |
|                                                                   `set_hide_after(hide_after: int)`                                                                    | Set the disappearance delay in milliseconds                                                        |

## Mini App Example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/text-elements/006_tooltip/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/text-elements/006_tooltip/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Button, Tooltip

```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Create widgets, that will be shown on app initialization.

Main widget for application `Button`, and interactive widget `Tooltip` that will display additional information for the main widget.

```python
new_button = Button("Push")
tooltip = Tooltip("Tooltip text", new_button)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
container = Container(widgets=[tooltip])
card = Card(title="Tooltip preview", content=container)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=card)
```
