# Select Tag

## Introduction

**`SelectTag`** widget in Supervisely is a compact dropdown menu that allows users to select tag metadata from a predefined list of `TagMeta` objects and create new tags on the fly. It supports features like filtering, multiple selection, and various tag value types (None, Any String, Any Number, One Of). This widget is commonly used in applications that need to work with annotation tags, such as filtering objects by tags, tag mapping, or configuring tagging workflows.

The `SelectTag` widget includes event handlers for tracking tag selection changes and new tag creation events.

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/widgets/selection/selecttag)

## Function signature

```python
SelectTag(
        tags: Optional[Union[List[TagMeta], TagMetaCollection]] = [],
        filterable: Optional[bool] = True,
        placeholder: Optional[str] = "Select tag",
        show_add_new_tag: Optional[bool] = True,
        size: Optional[Literal["large", "small", "mini"]] = None,
        multiple: bool = False,
        widget_id: Optional[str] = None,
)
```

![select-tag](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765895511654.png)

## Parameters

|     Parameters     |                     Type                      |                   Description                   |
| :----------------: | :-------------------------------------------: | :---------------------------------------------: |
|       `tags`       |   `Union[List[TagMeta], TagMetaCollection]`   |  List of TagMeta objects or TagMetaCollection.  |
|    `filterable`    |               `Optional[bool]`                | Enable search/filter functionality in dropdown. |
|   `placeholder`    |                `Optional[str]`                |    Placeholder text when no tag is selected.    |
| `show_add_new_tag` |               `Optional[bool]`                |   Whether to show button for adding new tags.   |
|       `size`       | `Optional[Literal["large", "small", "mini"]]` |          Size of the select dropdown.           |
|     `multiple`     |                    `bool`                     |     Whether multiple selection is enabled.      |
|    `widget_id`     |                `Optional[str]`                |            Unique widget identifier.            |

### tags

Initial list of TagMeta instances.

**type:** `Union[List[TagMeta], TagMetaCollection]`

**default value:** `[]`

```python
tag_weather = sly.TagMeta("weather", sly.TagValueType.ANY_STRING)
tag_count = sly.TagMeta("count", sly.TagValueType.ANY_NUMBER)

colors = ["red", "green", "blue"]
tag_color = sly.TagMeta("color", sly.TagValueType.ONEOF_STRING, possible_values=colors)

tag_verified = sly.TagMeta("verified", sly.TagValueType.NONE)

tags = [tag_weather, tag_count, tag_color, tag_verified]

select_tag = SelectTag(tags=tags)
```

### filterable

Enable or disable filtering/searching functionality in the dropdown.

**type:** `Optional[bool]`

**default value:** `True`

```python
select_tag = SelectTag(tags=tags, filterable=True)
```

### placeholder

Set placeholder text shown when no tag is selected.

**type:** `Optional[str]`

**default value:** `"Select tag"`

```python
select_tag = SelectTag(tags=tags, placeholder="Select a tag")
```

### multiple

Enable multiple tag selection.

**type:** `bool`

**default value:** `False`

```python
select_tag = SelectTag(tags, multiple=True)
```

### show_add_new_tag

Show or hide the "Add New Tag" button that allows users to create new tags directly from the widget with configurable value types.

**type:** `Optional[bool]`

**default value:** `True`

```python
select_tag = SelectTag(tags=tags, show_add_new_tag=True)
```

![add new tag button](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765895526731.png)

### size

Set the size of the widget.

**type:** `Optional[Literal["large", "small", "mini"]]`

**default value:** `None`

```python
select_tag = SelectTag(tags=tags, size="small")
```

## Methods and attributes

| Attributes and Methods | Description                                                  |
| :--------------------: | ------------------------------------------------------------ |
|  `get_selected_tag()`  | Get the currently selected TagMeta object(s).                |
|     `get_value()`      | Get the currently selected tag name(s).                      |
|     `set_value()`      | Sets the selected tag by name or list of names.              |
|    `get_all_tags()`    | Returns a copy of all available tags.                        |
|        `set()`         | Updates the list of available tags and refreshes the widget. |
|    `@value_changed`    | Decorator for value change event.                            |
|     `@tag_created`     | Decorator for new tag creation event.                        |

## Mini App Example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/selection/027_select_tag/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/027_select_tag/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, NotificationBox, SelectTag
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Create tag metadata

```python
# Create some initial tags with different value types
tag_weather = sly.TagMeta("weather", sly.TagValueType.ANY_STRING)
tag_count = sly.TagMeta("count", sly.TagValueType.ANY_NUMBER)

colors = ["red", "green", "blue"]
tag_color = sly.TagMeta("color", sly.TagValueType.ONEOF_STRING, possible_values=colors)

tag_verified = sly.TagMeta("verified", sly.TagValueType.NONE)

tags = [
    tag_weather,
    tag_count,
    tag_color,
    tag_verified,
]
```

### Initialize `SelectTag` widget

```python
select_tag = SelectTag(
    tags=tags,
    filterable=True,
    placeholder="Select a tag",
    multiple=True,
    show_add_new_tag=True,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
notification_box = NotificationBox()

card = Card(
    title="Select Tag",
    content=Container(widgets=[select_tag, notification_box]),
)

layout = Container(widgets=[card])

app = sly.Application(layout=layout)
```

### Handle tag selection changes

```python
@select_tag.value_changed
def on_tag_selected(selected_tags):
    if isinstance(selected_tags, list):
        tag_names = [tag.name for tag in selected_tags]
        notification_box.set(
            title="Tags selected",
            description=f"Selected tags: {', '.join(tag_names)}",
        )
    else:
        notification_box.set(
            title="Tag selected",
            description=f"Selected tag: {selected_tags.name}",
        )
```

### Handle new tag creation

```python
@select_tag.tag_created
def on_tag_created(new_tag: sly.TagMeta):
    notification_box.set(
        title="New tag created",
        description=f"You have created a new tag: {new_tag.name} (type: {new_tag.value_type})",
    )
```

![layout](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765895573820.png)
