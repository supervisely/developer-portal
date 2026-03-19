# Select User

## Introduction

**`SelectUser`** widget in Supervisely is a dropdown menu that allows users to select team members from a list of `UserInfo` objects. It supports features like filtering and multiple selection. This widget is commonly used in applications that need to assign tasks, filter data by user, or manage team member permissions.

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/widgets/selection/selectuser)

## Function signature

```python
SelectUser(
        users: Optional[List[UserInfo]] = None,
        team_id: Optional[int] = None,
        roles: Optional[List[str]] = None,
        filterable: Optional[bool] = True,
        placeholder: Optional[str] = "Select user",
        size: Optional[Literal["large", "small", "mini"]] = None,
        multiple: bool = False,
        widget_id: Optional[str] = None,
)
```

![select-user](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765979502349.png)

## Parameters

|  Parameters   |                     Type                      |                               Description                               |
| :-----------: | :-------------------------------------------: | :---------------------------------------------------------------------: |
|    `users`    |          `Optional[List[UserInfo]]`           |                   List of UserInfo objects for team.                    |
|   `team_id`   |                `Optional[int]`                |                 Team ID to fetch users from if needed.                  |
|    `roles`    |             `Optional[List[str]]`             | List of allowed user roles to filter by (e.g., ['admin', 'developer']). |
| `filterable`  |               `Optional[bool]`                |             Enable search/filter functionality in dropdown.             |
| `placeholder` |                `Optional[str]`                |               Placeholder text when no user is selected.                |
|    `size`     | `Optional[Literal["large", "small", "mini"]]` |                      Size of the select dropdown.                       |
|  `multiple`   |                    `bool`                     |                 Whether multiple selection is enabled.                  |
|  `widget_id`  |                `Optional[str]`                |                        Unique widget identifier.                        |

### users

Initial list of UserInfo instances from the team.

**type:** `Optional[List[UserInfo]]`

**default value:** `None`

```python
team_id = sly.env.team_id()
users = api.user.get_team_members(team_id)

select_user = SelectUser(users=users, team_id=team_id)
```

### team_id

Team ID to fetch users from if users list is not provided. When provided, users will be automatically loaded from the team.

**type:** `Optional[int]`

**default value:** `None`

```python
team_id = sly.env.team_id()
select_user = SelectUser(team_id=team_id)
```

### roles

List of allowed user roles to filter by. Only users with these roles will be shown in the dropdown.

**type:** `Optional[List[str]]`

**default value:** `None`

```python
# Show only admin and developer users
select_user = SelectUser(
    team_id=team_id,
    roles=['admin', 'developer']
)

# Show only annotators and reviewers
select_user = SelectUser(
    users=users,
    team_id=team_id,
    roles=['annotator', 'reviewer']
)
```

### filterable

Enable or disable filtering/searching functionality in the dropdown.

**type:** `Optional[bool]`

**default value:** `True`

```python
select_user = SelectUser(users=users, team_id=team_id, filterable=True)
```

### placeholder

Set placeholder text shown when no user is selected.

**type:** `Optional[str]`

**default value:** `"Select user"`

```python
select_user = SelectUser(users=users, team_id=team_id, placeholder="Select a user")
```

### multiple

Enable multiple user selection.

**type:** `bool`

**default value:** `False`

```python
select_user = SelectUser(users=users, team_id=team_id, multiple=True)
```

### size

Set the size of the widget.

**type:** `Optional[Literal["large", "small", "mini"]]`

**default value:** `None`

```python
select_user = SelectUser(users=users, team_id=team_id, size="small")
```

## Methods and attributes

|    Attributes and Methods     | Description                                                   |
| :---------------------------: | ------------------------------------------------------------- |
|     `get_selected_user()`     | Get the currently selected UserInfo object(s).                |
|         `get_value()`         | Get the currently selected user login(s).                     |
|         `set_value()`         | Sets the selected user by login or list of logins.            |
|       `get_all_users()`       | Returns a copy of all available users.                        |
|         `set_users()`         | Updates the list of available users and refreshes the widget. |
|        `set_team_id()`        | Load users from a team by team_id.                            |
| `set_selected_users_by_ids()` | Set the selected user(s) by user ID(s).                       |
|       `@value_changed`        | Decorator for value change event.                             |

## Mini App Example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/selection/028_select_user/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/028_select_user/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, NotificationBox, SelectUser
from supervisely.api.user_api import UserInfo
from typing import List
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get team members

```python
team_id = sly.env.team_id()
users = api.user.get_team_members(team_id)
```

### Initialize `SelectUser` widget

```python
select_user = SelectUser(
    users=users,
    team_id=team_id,
    filterable=True,
    placeholder="Select a user",
    multiple=True,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
notification_box = NotificationBox()

card = Card(
    title="Select User",
    content=Container(widgets=[select_user, notification_box]),
)

layout = Container(widgets=[card])

app = sly.Application(layout=layout)
```

### Handle user selection changes

```python
@select_user.value_changed
def on_user_selected(selected_users: List[UserInfo]):
    user_logins = [user.login for user in selected_users]
    print(f"Selected users: {user_logins}")
    notification_box.set(
        title="Users selected",
        description=f"Selected users: {', '.join(user_logins)}",
    )
```

![layout](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765979519403.png)
