# TreeSelect

## Introduction

The **`TreeSelect`** widget in Supervisely is a user interface element that allows users to select items from a tree-like structure that can be useful when working with hierarchical data.
The `TreeSelect` widget has event handlers that are triggered when items in the right lists are changed. This can be useful for applications that require dynamic change handling, such as showing data from the selected items.

[Read this tutorial in the developer portal.](https://developer.supervise.ly/app-development/widgets/selection/treeselect)

## Function signature

```python
TreeSelect(
    items = None,
    multiple_select = False,
    flat = False,
    always_open = False,
    widget_id = None,
)
```

![default](https://github.com/user-attachments/assets/890d44aa-2f2b-4d14-8946-e73ecbc2c148)

## Parameters

|    Parameters     |               Type                |                                     Description                                      |
| :---------------: | :-------------------------------: | :----------------------------------------------------------------------------------: |
|      `items`      | `Optional[List[TreeSelect.Item]]` |              A list of `TreeSelect.Item` to be displayed in the widget.              |
| `multiple_select` |              `bool`               |            If `True`, multiple items can be selected. Default is `False`.            |
|      `flat`       |              `bool`               | If `True`, selecting a parent item will not select its children. Default is `False`. |
|   `always_open`   |              `bool`               |          If `True`, the tree will be opened when the widget is initialized.          |
|    `widget_id`    |               `str`               |                            An optional ID for the widget.                            |

### items

Determine the list of `TreeSelect.Item` items, each of which must have a unique `id` and an optional `label` to be displayed in the widget.
The `children` field is a list of `TreeSelect.Item` items that are children of the current item.

**type:** `Optional[List[TreeSelect.Item]]`

**default value:** `None`

Prepare the items:

```python
items = [
    TreeSelect.Item(id="1", label="Item 1"),
    TreeSelect.Item(id="2", label="Item 2"),
    TreeSelect.Item(id="3", label="Item 3", children=[
        TreeSelect.Item(id="4", label="Item 3.1"),
        TreeSelect.Item(id="5", label="Item 3.2"),
    ]),
]
```

Initialize the widget with the items:

```python
widget = TreeSelect(items=items)
```

### multiple_select

If `True`, multiple items can be selected. Default is `False`.

**type:** `bool`

**default value:** `False`

Initialize the widget with multiple selection:

```python
widget = TreeSelect(items=items, multiple_select=True)
```

![multiple_select](https://github.com/user-attachments/assets/890d44aa-2f2b-4d14-8946-e73ecbc2c148)

### flat

If `True`, selecting a parent item will not select its children. Default is `False`.

**type:** `bool`

**default value:** `False`

Initialize the widget with the flat option:

```python
widget = TreeSelect(items=items, flat=True)
```

### always_open

If `True`, the tree will be opened when the widget is initialized.

**type:** `bool`

**default value:** `False`

Initialize the widget with the always open option:

```python
widget = TreeSelect(items=items, always_open=True)
```

### widget_id

An optional ID for the widget.

**type:** `str`

**default value:** `None`

Initialize the widget with an ID:

```python
widget = TreeSelect(items=items, widget_id="tree_select")
```

## Methods and attributes

|                        Methods and attributes                        |                           Description                            |
| :------------------------------------------------------------------: | :--------------------------------------------------------------: |
|              `set_items(items: List[TreeSelect.Item])`               | Set the items to be displayed in the widget (overrides current). |
|              `add_items(items: List[TreeSelect.Item])`               |      Add items to the current list of items in the widget.       |
|                           `clear_items()`                            |                 Clear all items from the widget.                 |
|                           `get_selected()`                           |              Get the selected items in the widget.               |
| `set_selected(value: Union[List[TreeSelect.Item], TreeSelect.Item])` |              Set the selected items in the widget.               |

## Mini app example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/selection/021_tree_select/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/021_tree_select/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from typing import List
from supervisely.app.widgets import Card, TreeSelect, Container, Text, Button, Flexbox
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Prepare the items

Prepare the items to be displayed in the `TreeSelect` widget:

```python
domestic_animals = [
    TreeSelect.Item(
        id="cat",
        label="Cat",
        children=[
            TreeSelect.Item(id="siamese", label="Siamese"),
            TreeSelect.Item(id="persian", label="Persian"),
        ],
    ),
    TreeSelect.Item(
        id="dog",
        label="Dog",
        children=[
            TreeSelect.Item(id="bulldog", label="Bulldog"),
            TreeSelect.Item(id="beagle", label="Beagle"),
        ],
    ),
    TreeSelect.Item(id="cow", label="Cow"),
    TreeSelect.Item(id="sheep", label="Sheep"),
]
```

### Initialize the `TreeSelect` widget

Let's initialize the `TreeSelect` widget without the items:

```python
tree_select = TreeSelect(
    multiple_select=True,
    flat=True,
    always_open=False,
)
```

And then set the items:

```python
tree_select.set_items(domestic_animals)
```

### Dynamically add items

Let's prepare a new list of items, which will be added dynamically to the widget.

```python
wild_animals = [
    TreeSelect.Item(
        id="mouse",
        label="Mouse",
        children=[
            TreeSelect.Item(id="field_mouse", label="Field mouse"),
            TreeSelect.Item(id="house_mouse", label="House mouse"),
        ],
    ),
    TreeSelect.Item(id="bird", label="Bird"),
]
```

And prepare a button, which will add the new items to the widget.

```python
add_button = Button("Add wild animals")
```

We also need to set the handler for the button click event.

```python
@add_button.click
def add_wild_animals():
    print("Adding wild animals")
    tree_select.add_items(wild_animals)

    add_button.disable()
```

### Show selected items in UI

Now, let's create a button, which will show selected items in UI. And create a simple text widget to show the selected items.

```python
show_selected_button = Button("Show selected")
selected_info = Text(status="info")
selected_info.hide()
```

Same as for the previous button, we need to set the handler for the click event.

```python
@show_selected_button.click
def show_selected():
    print("Showing selected items")
    selected_items = tree_select.get_selected()
    if selected_items and isinstance(selected_items, list):
        labels = ", ".join([item.label for item in selected_items])
        text = "Selected items: {}".format(labels)

        selected_info.text = text
        selected_info.show()
    else:
        selected_info.hide()
```

### Create an event handler for the `TreeSelect` widget

Now, let's create a event handler for the TreeSelect widget, when the value of the widget is changed.

```python
value_changed_info = Text(status="info")
value_changed_info.hide()


@tree_select.value_changed
def new_items_selected(items: List[TreeSelect.Item]):
    print("Value changed")
    if items:
        # We're processing the event the same way as for the button click event.
        labels = ", ".join([item.label for item in items])
        text = "Value changed: {}".format(labels)

        # Setting the text to the widget and showing it.
        value_changed_info.text = text
        value_changed_info.show()
    else:
        value_changed_info.hide()
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget. Let's group our buttons in a Flexbox widget to make them look better.

```python
buttons_flexbox = Flexbox(
    widgets=[add_button, show_selected_button],
)

card = Card(
    title="TreeSelect",
    content=Container(widgets=[buttons_flexbox, selected_info, value_changed_info, tree_select]),
)

layout = Container(widgets=[card])
```

### Create an app using the layout

Create an app object with the layout parameter.

```python
app = sly.Application(layout=layout)
```

![mini_app](https://github.com/user-attachments/assets/233e4878-9f04-4aa1-adbb-602eb9accad1)
