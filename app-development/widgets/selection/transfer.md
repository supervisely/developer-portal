# Transfer

## Introduction

The **`Transfer`** widget in Supervisely is a user interface element that allows users to transfer items between two lists. The Transfer widget is useful for creating project dashboards that require users to transfer items between two lists quickly and easily. Users can customize the appearance and behavior of the Transfer widget to match their project requirements, such as adding filterable options and customizing the names of the lists, buttons, and more. Overall, the Transfer widget is a simple tool to select needed or remove unnecessary items from the list.
The `Transfer` widget has event handlers that are triggered when items in the right lists are changed. This can be useful for applications that require dynamic change handling, such as showing data from the selected items.

[Read this tutorial in the developer portal.](https://developer.supervise.ly/app-development/widgets/selection/transfer)

## Function signature

```python
Transfer(
    items=None, transferred_items=None,
    filterable=False, filter_placeholder=None,
    titles=None, button_texts=None,
    left_checked=None, right_checked=None,
    widget_id=None
)
```

![default](https://user-images.githubusercontent.com/118521851/231207149-59b6e471-ed43-4dc2-b040-7b8f2ac46750.png)

## Parameters

|      Parameters      |              Type              |                    Description                     |
| :------------------: | :----------------------------: | :------------------------------------------------: |
|       `items`        | `Union[List[Item], List[str]]` | List of `Transfer.Item` widgets or list of strings |
| `transferred_items`  |          `List[str]`           |   List of strings, containing keys from `items`    |
|     `filterable`     |             `bool`             |          Whether `Transfer` is filterable          |
| `filter_placeholder` |             `str`              |    Placeholder for filter if `filterable=True`     |
|       `titles`       |           `List[str`           |      List of titles for left and right lists       |
|    `button_texts`    |           `List[str`           |   List of button texts for left and right lists    |
|    `left_checked`    |           `List[str`           |        List of checked items for left list         |
|   `right_checked`    |           `List[str`           |        List of checked items for right list        |
|     `widget_id`      |             `str`              |                  ID of the widget                  |

### items

Determine the list of `Transfer.Item` widgets or list of strings, containing keys for items, from which `Transfer.Item` widgets will be created.

**type:** `Union[List[Item], List[str]]`

**default value:** `None`

Prepare transfer items:

```python
animals_domestic = [
    Transfer.Item(key="cat", label="Cat", disabled=True),
    Transfer.Item(key="dog", label="Dog"),
]
animals_wild = [
    Transfer.Item(key="mouse"),
]
```

Initialize widget with given items:

```python
transfer = Transfer(
    items=animals_domestic + animals_wild,
)
```

![items](https://user-images.githubusercontent.com/118521851/231207097-c9a63aa5-f1b3-4115-88a0-9838df468313.png)

### transferred_items

Determine the list of strings, containing keys for items, that will be shown in the right list.
_Note: `transferred_items` should be a subset of keys of `items`, otherwise the error will be raised._

**type:** `List[str]`

**default value:** `None`

Initialize widget with transferred items:

```python
transfer = Transfer(
    items=animals_domestic + animals_wild,
    transferred_items=["dog"],
)
```

![transferred_items](https://user-images.githubusercontent.com/118521851/231207110-93c52970-d098-41f1-9508-2f25c5f7d653.png)

### filterable

Determine whether the `Transfer` is filterable or not.

**type:** `bool`

**default value:** `False`

Initialize widget with filterable option:

```python
transfer = Transfer(
    items=animals_domestic + animals_wild,
    filterable=True,
)
```

![filterable](https://user-images.githubusercontent.com/118521851/231207116-e26380a3-a492-4038-a395-ee3873e2563d.png)

### filter_placeholder

Determine the placeholder for the filter field if `filterable=True`.

**type:** `str`

**default value:** `None`

Initialize widget with filter placeholder:

```python
transfer = Transfer(
    items=animals_domestic + animals_wild,
    filterable=True,
    filter_placeholder="Filter animals",
)
```

![filter_placeholder](https://user-images.githubusercontent.com/118521851/231207120-771c23c7-4544-4ce1-bedf-d724d86b9bb3.png)

### titles

Determine the list of titles for left and right lists.

**type:** `List[str]`

**default value:** `None`

Initialize widget with titles:

```python
transfer = Transfer(
    items=animals_domestic + animals_wild,
    titles=["Domestic animals", "Wild animals"],
)
```

![titles](https://user-images.githubusercontent.com/118521851/231207127-e43bc9f9-e02d-4de0-babc-ee2d8066f74c.png)

### button_texts

Determine the list of texts for each button.

**type:** `List[str]`

**default value:** `None`

Initialize widget with button texts:

```python
transfer = Transfer(
    items=animals_domestic + animals_wild,
    button_texts=["Remove", "Add"],
)
```

![button_texts](https://user-images.githubusercontent.com/118521851/231207130-dd9c7f89-2bc4-4a5a-9c87-0cac6865f430.png)

### left_checked

Determine the list of items that will be checked in the left list when the widget is initialized.

**type:** `List[str]`

**default value:** `None`

Initialize widget with checked items:

```python
transfer = Transfer(
    items=animals_domestic + animals_wild,
    left_checked=["dog"],
)
```

![left_checked](https://user-images.githubusercontent.com/118521851/231207135-86f0a3ad-68cf-4463-a3fc-04256ab38508.png)

### right_checked

Determine the list of items that will be checked in the right list when the widget is initialized.

**type:** `List[str]`

**default value:** `None`

Initialize widget with checked items:

```python
transfer = Transfer(
    items=animals_domestic + animals_wild,
    transferred_items=["dog"],
    right_checked=["dog"],
)
```

![right_checked](https://user-images.githubusercontent.com/118521851/231207141-ee6023a4-f9d5-4918-90a3-aa1ae7839a36.png)

### widget_id

The ID of the widget.

**type:** `str`

**default value:** `None`

Initialize widget with ID:

```python
transfer = Transfer(
    items=animals_domestic + animals_wild,
    widget_id="transfer",
)
```

## Methods and attributes

|                  Methods and attributes                   |                         Description                          |
| :-------------------------------------------------------: | :----------------------------------------------------------: |
|                 `get_transferred_items()`                 |       Get the list of keys of items in the right list        |
|                `get_untransferred_items()`                |        Get the list of keys of items in the left list        |
|                     `@value_changed`                      | Event that is triggered when items in the right list changed |
| `set_items(items: Union[List[Transfer.Item], List[str]])` |       Set the items list (replacing it) for the widget       |
|   `set_transferred_items(transferred_items: List[str])`   | Set the transferred items list (replacing it) for the widget |
|        `add(items: Union[List[Item], List[str]]):`        |                 Adds new items to the widget                 |
|              `remove(items_keys: List[str])`              |             Remove items from the widget by key              |
|                    `get_items_keys()`                     |         Get the list of keys of items in the widget          |

## Mini app example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/selection/010_transfer/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/010_transfer/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Transfer, Container, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare items for `Transfer` widget

```python
# Preparing list of items as Transfer.Item objects.
domestic_animals = [
    Transfer.Item(key="cat", label="Cat", disabled=True),
    Transfer.Item(key="dog"),
    Transfer.Item(key="cow"),
    Transfer.Item(key="sheep"),
]
# Preparing list of items as strings, which will be converted to Transfer.Item objects.
wild_animals = ["mouse", "bird"]
```

### Initialize `Transfer` widget

```python
# Initializing Transfer widget with first list of items.
transfer = Transfer(
    # List of all items for the widget (both left and right).
    items=domestic_animals,
    # List of items that will in the right list when the widget is initialized.
    transferred_items=["cat", "cow"],
    # Making the widget lists filterable.
    filterable=True,
    # Setting the placeholder for the filter input.
    filter_placeholder="Search...",
    # Setting the title for the both lists.
    titles=["Available", "Transferred"],
    # Setting the names for the buttons.
    button_texts=["Remove", "Add"],
    # Setting the items which will be checked in the left list when the widget is initialized.
    left_checked=["dog"],
    # Setting the items which will be checked in the right list when the widget is initialized.
    right_checked=["cow"],
)
# Adding new items to the widget.
transfer.add(wild_animals)
```

### Changing the list of transferred items

```python
# Updating the transferred items, which will be shown in the right list.
# Note: if you want to ADD items (not REPLACE them), don't forget to pass the old transferred items to the method.
transfer.set_transferred_items(transfer.get_transferred_items() + ["mouse"])
```

### Removing an item and getting keys of items (all, transferred, untransferred)

```python
# Printing the keys of items in the left list.
print("Keys of items in the left list ", transfer.get_untransferred_items())
# Printing the keys of items in the right list.
print("Keys of items in the right list ", transfer.get_transferred_items())

# Removing item from widget by its key.
transfer.remove(["sheep"])

# Printing the keys of all items in the widget.
print("Keys of all items in the widget ", transfer.get_items_keys())
```

### Using `@value_changed` event

```python
# Preparing text widget to show the message about the value change.
value_changed_message = Text(status="info")


# Creating decorated function to show the current state of the widget.
@transfer.value_changed
def show_message_with_keys(Items):
    # The Items object is a named tuple containing two lists:
    # keys of items in the left list and keys of items in the right list.
    value_changed_message.text = (
        f"Left list: {Items.untransferred_items}. Right list: {Items.transferred_items}."
    )
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
# Creating Card widget, which will contain the Transfer widget and the Text widget.
card = Card(title="Transfer", content=Container(widgets=[transfer, value_changed_message]))
# Creating the application layout.
layout = Container(widgets=[card])
```

### Create an app using the layout

Create an app object with the layout parameter.

```python
# Initializing the application.
app = sly.Application(layout=layout)
```

![mini_app](https://user-images.githubusercontent.com/118521851/231207149-59b6e471-ed43-4dc2-b040-7b8f2ac46750.png)
