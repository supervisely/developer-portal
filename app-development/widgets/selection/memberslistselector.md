# MembersListSelector

## Introduction

**`Members List Selector`** widget allows to display list of users with roles.

## Function signature

```python
members_list_selector = MembersListSelector(
    users=members,
    multiple=True,
    widget_id=None,
)
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/28a62429-88d8-4932-b70e-258fe3488855)

## Parameters

|  Parameters |       Type       |           Description           |
| :---------: | :--------------: | :-----------------------------: |
|   `users`   | `List[UserInfo]` |  Supervisely `UserInfo` objects |
|  `multiple` |      `bool`      | Enable multiple users selection |
| `widget_id` |       `str`      |         ID of the widget        |

### users

List of `UserInfo` objects.

**type:** `List[UserInfo]`

```python
members_list_selector = MembersListSelector(
    users=members,
)
```

### multiple

If `True` then multiple users can be selected. Otherwise, only one user can be selected.

**type:** `bool`

**default** `False`

```python
members_list_selector = MembersListSelector(
    users=members,
    multiple=True,
)
```

{% tabs %}
{% tab title="Multiple Enabled" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/5d6901a1-d509-4f95-8a8c-d6273778fd1e" alt=""><figcaption><p>Multiple Enabled</p></figcaption></figure>
{% endtab %}

{% tab title="Multiple disabled" %}
<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/3f6a04a3-d39f-4fe5-9e1f-02c4c94224f7" alt=""><figcaption><p>Multiple disabled</p></figcaption></figure>
{% endtab %}
{% endtabs %}

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|  Attributes and Methods  | Description                                  |
| :----------------------: | -------------------------------------------- |
|          `set()`         | Set members to widget.                       |
| `get_selected_members()` | Return list of selected members.             |
|      `select_all()`      | Select all members.                          |
|     `deselect_all()`     | Deselect all members.                        |
|        `select()`        | Select members by names.                     |
|       `deselect()`       | Deselect members by names.                   |
|     `set_multiple()`     | Set multiple flag.                           |
|    `get_all_members()`   | Return list of all members.                  |
|   `@selection_changed`   | Callback triggers when selection is changed. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/selection/018\_members\_list\_selector/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/018\_members\_list\_selector/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, MembersListSelector, Text
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get list of team members from API

```python
team_members = api.user.get_team_members(team_id)
```

### Initialize `MembersListSelector` widget and `Text` widget for displaying selected members count

```python
members_list_selector = MembersListSelector(team_members, multiple=True)
selected_members_cnt = Text(f"Selected Members: 0 / {len(team_members)}")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created into the `Container` widget.

```python
container = Container(widgets=[selected_members_cnt, members_list_selector])

card = Card(
    title="Members List Selector",
    content=members_list_selector,
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
    selected_members_cnt.set(
        f"Selected Members: {len(selected_members)} / {len(team_members)}", "text"
    )
```

![miniapp-min](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/63503ddb-36e1-4d18-9942-f45f51e4bab5)
