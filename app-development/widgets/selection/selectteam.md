# SelectTeam

## Introduction

**`SelectTeam`** widget in Supervisely is a dropdown menu that allows users to select a team from a list of available teams. Clicking on it can be processed from python code. This widget is particularly useful when working with multiple teams in Supervisely and allows to easily switch between team in applications.

## Function signature

```python
SelectTeam(
    default_id=None,
    show_label=True,
    size=None,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222352390-7631c1b5-30ce-4dc8-8924-c9c34c4cb6a1.png" alt=""><figcaption></figcaption></figure>

## Parameters

|  Parameters  |                    Type                   |            Description           |
| :----------: | :---------------------------------------: | :------------------------------: |
| `default_id` |                   `int`                   |    Default supervisely team ID   |
| `show_label` |                   `bool`                  |            Show label            |
|    `size`    | `Literal["large", "small", "mini", None]` | Selector size (large/small/mini) |
|  `widget_id` |                   `str`                   |         ID of the widget         |

### default\_id

Determine `Team` will be selected by default.

**type:** `int`

**default value:** `None`

```python
select_team = SelectTeam(default_id=team_id)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222352390-7631c1b5-30ce-4dc8-8924-c9c34c4cb6a1.png" alt=""><figcaption></figcaption></figure>

### show\_label

Determine show text `Team` on widget or not.

**type:** `bool`

**default value:** `True`

```python
select_team = SelectTeam(default_id=team_id, show_label=False)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222352402-803089e7-bbb8-4540-936c-4430a11f1626.png" alt=""><figcaption></figcaption></figure>

### size

Size of input.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
select_team = SelectTeam(default_id=team_id, show_label=False)
select_mini = SelectTeam(default_id=team_id, show_label=False, size="mini")
select_small = SelectTeam(default_id=team_id, show_label=False, size="small")
select_large = SelectTeam(default_id=team_id, show_label=False, size="large")
card = Card(
    title="Select Team",
    content=Container(widgets=[select_team, select_mini, select_small, select_large]),
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222354681-a4271d6b-1307-4886-a2be-fa98320b1568.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description              |
| :--------------------: | ------------------------ |
|   `get_selected_id()`  | Return selected team id. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/selection/002\_select\_team/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/002\_select\_team/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, NotificationBox, SelectTeam
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare `Team` ID

```python
team_id = sly.env.team_id()
```

### Initialize `SelectTeam` widget

```python
select_team = SelectTeam()
```

### Create button and notification box we will use in UI for demo

```python
ok_btn = Button("OK")

notification_box = NotificationBox()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select Team",
    content=Container(widgets=[select_team, ok_btn, notification_box]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to get `team ID` from widget

```python
@ok_btn.click
def show_team_members():
    team_id = select_team.get_selected_id()
    team = api.team.get_info_by_id(team_id)
    notification_box.set(
        title=f"Team '{team.name}'",
        description=f"Your role in the team: {team.role}",
    )
```

<figure><img src="https://user-images.githubusercontent.com/79905215/222133799-90b573a4-d1fa-4c8e-a337-9665cd8ae458.gif" alt=""><figcaption></figcaption></figure>
