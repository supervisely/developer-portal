# ReloadableArea

## Introduction

**`ReloadableArea`** widget in Supervisely is a tool that allows dynamically creating and changing widgets in the application UI without reloading the whole page. It can be used to create a dynamic UI that changes depending on user actions without any noticeable delays.

[Read this tutorial in the developer portal.](https://developer.supervise.ly/app-development/widgets/selection/reloadablearea)

## Function signature

```python
ReloadableArea(
    content=None,
    widget_id=None,
)
```

<figure><img src="https://camo.githubusercontent.com/abf9f7ceacdfc6c0cc3976c4ef9e5512c60b7c786e3bce4ca7f388b964cf44f5/68747470733a2f2f6769746875622d70726f64756374696f6e2d757365722d61737365742d3632313064662e73332e616d617a6f6e6177732e636f6d2f3131383532313835312f3235303235373932382d38643762633865612d333230632d346238332d616436652d6237373662383665613134622e676966" alt=""><figcaption></figcaption></figure>

## Parameters

|  Parameters |   Type   |             Description             |
| :---------: | :------: | :---------------------------------: |
|  `content`  | `Widget` | Widget to be displayed in the area. |
| `widget_id` |   `str`  |           ID of the widget          |

### content

Determine the `Widget` to be displayed in the ReloadableArea.

**type:** `Widget`

**default value:** `None`

Prepare widgets and put them into a container:

```
text = Text("This is a text widget", status="info")
checkbox = Checkbox("Checkbox")
container = Container(widgets=[text, checkbox])
```

Initialize the ReloadableArea with the container:

```python
reloadable_area = ReloadableArea(content=container)
```

### widget\_id

The ID of the widget.

**type:** `str`

**default value:** `None`

Initialize widget with ID:

```python
reloadable_area = ReloadableArea(widget_id="reloadable_area")
```

## Methods and attributes

|     Methods and attributes     |                       Description                      |
| :----------------------------: | :----------------------------------------------------: |
| `set_content(content: Widget)` | Replaces content of the ReloadableArea with new widget |
|           `reload()`           | Reloads the content of the ReloadableArea widget in UI |

## Mini app example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/layouts\_and\_containers/014\_reloadable\_area/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/tree/master/layouts%20and%20containers/014\_reloadable\_area/src/main.py)

### Import libraries

We're importing all those widgets just to use them in the demonstration of ReloadableArea.

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import (
    Card,
    ReloadableArea,
    Container,
    Text,
    Input,
    InputNumber,
    Checkbox,
    Button,
    Flexbox,
    Switch,
    Slider,
)
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Preparing widgets for app initialization

```python
# Create widgets, that will be shown on app initialization.
start_text = Text("Click a button to add or remove random widget.", status="info")
add_widget_button = Button("Add widget")
remove_widget_button = Button("Remove widget", button_type="danger")
buttons_flexbox = Flexbox([add_widget_button, remove_widget_button])
```

### Initialize ReloadableArea

```python
# Create a reloadable area.
reloadable_area = ReloadableArea()

# Put widgets into the Container.
main_container = Container(widgets=[start_text, buttons_flexbox])

# Add the Container to the reloadable area.
reloadable_area.set_content(main_container)
```

### Adding event handlers for buttons to add and remove widgets

This event will be triggered when the user clicks the `Add widget` button. The random widget will be chosen from the list and added to the UI.

```python
@add_widget_button.click
def add_widget():
    # Picking random widget from the list to add to the UI.
    random_widget = choice(
        [
            Input("Random input"),
            Checkbox("Random checkbox"),
            InputNumber(),
            Switch(),
            Slider(),
        ]
    )

    main_container._widgets.append(random_widget)

    # This is the most important part: we need to reload the ReloadableArea
    # to show the new widget in th UI.
    reloadable_area.reload()
```

This event will be triggered when the user clicks the `Remove widget` button. The last widget in the list will be removed from the UI. In both cases, we need to use the `.reload()` method of `ReloadableArea` to show the changes in the UI.

```python
@remove_widget_button.click
def remove_widget():
    if len(main_container._widgets) > 2:
        main_container._widgets.pop()

        # Same as above: 
        # we need to reload the ReloadableArea when removing the widget.
        reloadable_area.reload()
```

### Create an app using the layout

Prepare a layout for the app using `Card` widget with the `content` parameter. Then create an app object with the layout parameter.

```python
# Creating Card widget, which will contain the ReloadableArea widget.
card = Card(title="ReloadableArea", content=reloadable_area)

# Initializing the application.
app = sly.Application(layout=card)
```
