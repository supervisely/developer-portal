# MembersListPreview

## Introduction

**`Members List Preview`** widget simply displays a list of members. It can be used to display members that were selected by the user in the [Members List Selector](https://developer.supervisely.com/app-development/apps-with-gui/members-list-selector) widget for example.

## Function signature

```python
members_list_preview = MembersListPreview(
    users=members,
    max_width=300,
    empty_text=None,
    widget_id=None,
)
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/d674558e-739d-4dd1-be66-a48f8f00adff)

## Parameters

|  Parameters  |       Type       |                           Description                           |
| :----------: | :--------------: | :-------------------------------------------------------------: |
|    `users`   | `List[UserInfo]` |                  Supervisely `UserInfo` objects                 |
|  `max_width` |       `int`      |                     Max width of the widget                     |
| `empty_text` |       `str`      | Text that will be displayed when there are no members in widget |
|  `widget_id` |       `str`      |                         ID of the widget                        |

### users

List of `UserInfo` objects.

**type:** `List[UserInfo]`

```python
members_list_preview = MembersListPreview(
    users=members
)
```

### max\_width

Set the maximum width of the widget in pixels.

**type:** `int`

**default** `300`

```python
members_list_preview = MembersListPreview(
    users=members,
    max_width=150
)
```

### empty\_text

Text that will be displayed when there are no members in the widget.

**type:** `str`

**default** `None`

```python
empty_text = "No members selected"
members_list_preview = MembersListPreview(
    users=members,
    empty_text=empty_text
)
```

![empty-text](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/45c8ab27-9466-4764-9796-d94f0c3b4fc2)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

### Methods and attributes

| Attributes and Methods | Description              |
| :--------------------: | ------------------------ |
|         `set()`        | Set users to the widget. |

## Mini App Example

In this example we will create a mini app with `MembersListPreview` widget. We will create a `MembersListSelector` widget and display selected members with `MembersListPreview` widget.

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/019\_members\_list\_preview/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/019\_members\_list\_preview/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import (
    Card,
    Container,
    MembersListPreview,
    MembersListSelector,
    Text,
)
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Define team ID

```python
team_id = 8  # Change this to your team ID
# team_id = sly.env.team_id() # Uncomment this line to use the team ID from the local.env file
```

### Get Team members and initialize `MembersListSelector` widget

```python
members = api.user.get_team_members(team_id)
members_list_selector = MembersListSelector(members, multiple=True)
```

### Initialize `MembersListPreview` widget and `Text` widget for displaying number of selected members

```python
empty_text = Text("No members selected", "text")
members_list_preview = MembersListPreview(empty_text=empty_text)
preview_text = Text(f"Selected Members: 0 / {len(members)}", "text")
preview_container = Container([preview_text, members_list_preview])
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widgets that we've previously created into the `Container` widget.

```python
container = Container(widgets=[members_list_selector, preview_container])

card = Card(
    title="Members List Preview",
    content=container,
)

layout = card
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from python code

```python
@members_list_selector.selection_changed
def on_selection_changed(selected_members):
    preview_text.set(f"Selected Members: {len(selected_members)} / {len(members)}", "text")
    members_list_preview.set(selected_members)
```

![mini-app-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/4368d2f1-1cd6-4d74-bcf2-5641ec4e00bb)
