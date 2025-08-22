# Input Tag List

## Introduction

**`InputTagList`** widget in Supervisely is a user interface element that allows users to work with multiple tag labels simultaneously. Unlike the single `InputTag` widget, `InputTagList` enables bulk operations on tag collections, making it easier to manage multiple tags at once. It uses a list of `TagMeta` objects and provides convenient methods for selecting, deselecting, and manipulating multiple tags. This widget is particularly useful for applications that need to handle multiple tag operations efficiently.

## Function signature

```python
InputTagList(
    tag_metas,
    max_width=300,
    max_height=50,
    multiple=False,
    widget_id=None,
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.7/widget-tagInputList.png)

## Parameters

|  Parameters  |                   Type                    |                                               Description                                               |
| :----------: | :---------------------------------------: | :-----------------------------------------------------------------------------------------------------: |
| `tag_metas`  | `Union[List[TagMeta], TagMetaCollection]` |                       List of `TagMeta` objects from which `Tags` will be managed                       |
| `max_width`  |                   `int`                   |                                           Max tag field width                                           |
| `max_height` |                   `int`                   |                                          Max tag field height                                           |
|  `multiple`  |                  `bool`                   | If set False, only one tag can be selected at a time, otherwise can select several tags (multi-select). |
| `widget_id`  |                   `str`                   |                                            ID of the widget                                             |

### tag_metas

Determine list of `TagMeta` objects from which `Tags` will be managed. Supports all possible `Tag` types: `any_number`, `any_string`, `one_of_string`, `none`.

**type:** `Union[List[TagMeta], TagMetaCollection]`

### max_width

Determine max tag field width.

**type:** `int`

**default value:** `300`

```python
input_tag_list = InputTagList(tag_metas=tag_metas, max_width=400)
```

### max_height

Determine max tag field height.

**type:** `int`

**default value:** `50`

```python
input_tag_list = InputTagList(tag_metas=tag_metas, max_height=100)
```

### multiple

If set False, only one tag can be selected at a time, otherwise can select several tags (multi-select).

**type:** `bool`

**default value:** `False`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|        Attributes and Methods        | Description                                                  |
| :----------------------------------: | ------------------------------------------------------------ |
|            `select_all()`            | Select all available tags in the list.                       |
|           `deselect_all()`           | Deselect all tags in the list.                               |
|    `select(tag_names: List[str])`    | Select specific tags by their names.                         |
|        `get_selected_tags()`         | Get list of currently selected tags with their values.       |
|      `get_selected_tag_metas()`      | Get list of `TagMeta` objects for currently selected tags.   |
| `set_values(values: Dict[str, Any])` | Set values for tags by tag name.                             |
|         `@selection_changed`         | Decorator function is handled when tag selection is changed. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/misc/input_tag_list/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/misc/input_tag_list/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, Button, Text, InputTagList
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Create different types of tag metas for demonstration

```python
# Create different types of tag metas
tag_meta_none = sly.TagMeta("none_tag", sly.TagValueType.NONE)
tag_meta_string = sly.TagMeta("string_tag", sly.TagValueType.ANY_STRING)
tag_meta_num = sly.TagMeta("number_tag", sly.TagValueType.ANY_NUMBER)
tag_meta_oneof = sly.TagMeta(
    "oneof_tag", sly.TagValueType.ONEOF_STRING, possible_values=["option1", "option2", "option3"]
)

# Additional tag metas for demonstration
tag_meta_quality = sly.TagMeta(
    "quality", sly.TagValueType.ONEOF_STRING, possible_values=["high", "medium", "low"]
)
tag_meta_category = sly.TagMeta("category", sly.TagValueType.ANY_STRING)
tag_meta_confidence = sly.TagMeta("confidence", sly.TagValueType.ANY_NUMBER)

# Create a list of tag metas
tag_metas = [
    tag_meta_none,
    tag_meta_string,
    tag_meta_num,
    tag_meta_oneof,
    tag_meta_quality,
    tag_meta_category,
    tag_meta_confidence,
]
```

### Initialize `InputTagList` widget

```python
input_tag_list = InputTagList(tag_metas=tag_metas, max_width=400, max_height=200)
```

### Initialize control buttons and text widget

```python
# Create buttons for demonstration
btn_select_all = Button("Select All")
btn_deselect_all = Button("Deselect All")
btn_select_specific = Button("Select random")
btn_get_selected = Button("Get Selected Tags")
btn_set_values = Button("Set Sample Values")

# Text widget to display results
result_text = Text("Select tags and click buttons to see the results", status="info")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widgets in containers.

```python
# Create layout
buttons_container = Container(
    [btn_select_all, btn_deselect_all, btn_select_specific, btn_get_selected, btn_set_values],
    direction="vertical",
    gap=10,
)

main_container = Container(
    [Container([input_tag_list, result_text], direction="vertical", gap=20), buttons_container],
    direction="horizontal",
    gap=20,
)

card = Card(
    title="InputTagList Widget Demo",
    content=main_container,
    description="This demo shows how to use the InputTagList widget with different tag types and operations.",
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Handle button clicks and selection changes

Use decorators to handle button clicks and tag selection changes:

```python
# Event handlers
@btn_select_all.click
def select_all():
    input_tag_list.select_all()
    result_text.text = "All tags selected"
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.7/screenshot-localhost-8000-1755869658195.png" alt=""><figcaption></figcaption></figure>

```python
@btn_deselect_all.click
def deselect_all():
    input_tag_list.deselect_all()
    result_text.text = "All tags deselected"

@btn_select_specific.click
def select_specific():
    input_tag_list.select(["quality", "category"])
    result_text.text = "Selected 'quality' and 'category' tags"
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.7/screenshot-localhost-8000-1755869682978.png" alt=""><figcaption></figcaption></figure>

```python
@btn_get_selected.click
def get_selected():
    selected_tags = input_tag_list.get_selected_tags()

    if selected_tags:
        result_info = f"Selected {len(selected_tags)} tags:\n"
        for tag in selected_tags:
            result_info += f"- {tag.meta.name}: {tag.value} (type: {tag.meta.value_type})\n"
    else:
        result_info = "No tags selected"

    result_text.text = result_info
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.7/screenshot-localhost-8000-1755869682978.png" alt=""><figcaption></figcaption></figure>

```python
@btn_set_values.click
def set_sample_values():
    sample_values = {
        "string_tag": "sample text",
        "number_tag": 42,
        "oneof_tag": "option2",
        "quality": "high",
        "category": "demo",
        "confidence": 0.95,
    }
    input_tag_list.set_values(sample_values)
    result_text.text = "Sample values set for all tags"
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.7/screenshot-localhost-8000-1755869702968.png" alt=""><figcaption></figcaption></figure>

```python
# Selection change handler
@input_tag_list.selection_changed
def on_selection_changed(selected_tag_metas):
    if selected_tag_metas:
        names = [tm.name for tm in selected_tag_metas]
        result_text.text = f"Selection changed: {', '.join(names)}"
    else:
        result_text.text = "No tags selected"
```
