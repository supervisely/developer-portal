# Hotkeys

## Introduction

**`Hotkeys`** widget in Supervisely lets an app react to keyboard shortcuts anywhere on the page, not only while a specific element has focus. It has no visual representation &mdash; you register the key combinations you care about, then attach a Python callback to react when one of them is pressed. Active inputs (text fields, textareas, selects, contenteditable elements) always take priority: while the user is typing, none of the registered hotkeys fire, so shortcuts never collide with normal text input.

## Function signature

```python
Hotkeys(
    hotkeys=None,
    prevent_default=True,
    ignore_input_focus=True,
    widget_id=None,
)
```

## Parameters

|      Parameters      |    Type     |                                        Description                                         |
| :-------------------: | :---------: | :------------------------------------------------------------------------------------------: |
|       `hotkeys`       | `List[str]` |             Key combinations to catch, e.g. `["ctrl+s", "shift+arrowright", "a"]`             |
|   `prevent_default`   |   `bool`    | If `True`, stops the browser's own shortcut for a matched combination (e.g. Ctrl+S saving the page) |
| `ignore_input_focus`  |   `bool`    |     If `True`, hotkeys are not caught while an input/textarea/select/contenteditable has focus     |
|      `widget_id`      |    `str`    |                                      ID of the widget                                      |

### hotkeys

Key combinations to catch. Modifiers must be listed before the key itself, in the order `ctrl`, `alt`, `shift`, joined with `+` (e.g. `"ctrl+shift+z"`). A plain key with no modifiers is just the key name (e.g. `"a"`, `"arrowup"`, `"enter"`, `"space"`).

**type:** `List[str]`

**default value:** `None`

```python
hotkeys = Hotkeys(hotkeys=["ctrl+s", "ctrl+z", "arrowup", "arrowdown"])
```

### prevent_default

If `True`, calls `event.preventDefault()` for matched combinations, so the browser's own shortcut (e.g. Ctrl+S opening the "Save page" dialog) does not trigger alongside your handler.

**type:** `bool`

**default value:** `True`

### ignore_input_focus

If `True`, a registered combination is not caught while the user is typing in an `<input>`, `<textarea>`, `<select>`, or a contenteditable element &mdash; legitimate typing always takes priority over hotkeys.

**type:** `bool`

**default value:** `True`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|              Attributes and Methods              | Description                                                                                                          |
| :-------------------------------------------------: | ---------------------------------------------------------------------------------------------------------------------- |
|                     `hotkeys`                      | Get the list of key combinations this widget catches.                                                                |
|                   `pressed_key`                     | Get the last key combination that was caught.                                                                        |
|            `add_hotkey(combo: str)`                | Add a new key combination to catch, without restarting the app.                                                      |
|         `@key_pressed(combo: str = None)`          | Decorator, called when `combo` is pressed. If `combo` is omitted, called for every combination caught by this widget. |

{% hint style="info" %}
The `Hotkeys` widget has no visual representation, but it still has to be placed somewhere in the layout (e.g. inside a `Container`) &mdash; otherwise its keyboard listener never mounts in the browser.
{% endhint %}

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/controls/011\_hotkeys/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/controls/011\_hotkeys/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Hotkeys, Input, Text
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Initialize `Text`, `Input` and `Hotkeys` widgets

We add two inputs on purpose &mdash; a single-line `Input` and a textarea `Input` &mdash; so you can verify that typing `a` or pressing `Ctrl+A` inside either of them does **not** trigger the hotkeys below.

```python
status_text = Text(text="No hotkey pressed yet", status="text")
counter_text = Text(text="Presses: 0", status="text")

text_input = Input(placeholder="Single-line input: try 'a' or Ctrl+A here...")
textarea_input = Input(placeholder="Textarea: try the same here...", type="textarea")

hotkeys = Hotkeys(hotkeys=["a", "ctrl+a", "ctrl+s", "ctrl+z"])
```

### Create app layout

Prepare a layout using the `Card` widget with the `content` parameter. Remember to include the `Hotkeys` widget itself somewhere in the tree, even though it renders nothing visible.

```python
layout = Card(
    title="Hotkeys",
    content=Container([status_text, counter_text, text_input, textarea_input, hotkeys]),
)
```

### Create app using layout

```python
app = sly.Application(layout=layout)
```

### Handle hotkey presses

Use the `@hotkeys.key_pressed()` decorator (no argument) to react to any of the registered combinations, and look at which one fired from the argument passed to the handler:

```python
press_count = 0

combo_to_label = {
    "a": ("text", "You pressed A"),
    "ctrl+a": ("info", "You pressed Ctrl+A (Select All)"),
    "ctrl+s": ("success", "You pressed Ctrl+S (Save)"),
    "ctrl+z": ("warning", "You pressed Ctrl+Z (Undo)"),
}


@hotkeys.key_pressed()
def on_hotkey_pressed(combo):
    global press_count
    press_count += 1
    status, label = combo_to_label.get(combo, ("text", f"You pressed: {combo}"))
    status_text.set(f"{label}!", status)
    counter_text.text = f"Presses: {press_count}"
```

You can also bind a handler to one specific combination:

```python
@hotkeys.key_pressed("ctrl+s")
def on_save():
    print("Ctrl+S pressed")
```

Try it: press `A`, `Ctrl+A`, `Ctrl+S`, or `Ctrl+Z` anywhere on the page and watch the counter go up. Then click into one of the inputs and press the same keys &mdash; the counter won't change, because typing always wins.
