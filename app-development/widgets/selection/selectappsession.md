# SelectAppSession

## Introduction

**`SelectAppSession`** widget in Supervisely is a dropdown menu that allows users to select an application session from a list of available sessions. `SelectAppSession` widget is particularly useful when working with multiple application sessions in Supervisely. It can be customized with various parameters, such as the size and label showing.

## Function signature

```python
SelectAppSession(
    team_id, tags,
    show_label=False,
    size="mini",
    operation="or",
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219646892-e064bd68-20f1-4ce3-9f1b-89650fe1dde0.gif" alt=""><figcaption></figcaption></figure>

## Parameters

|  Parameters  |                    Type                   |            Description           |
| :----------: | :---------------------------------------: | :------------------------------: |
|   `team_id`  |                   `int`                   |              Team ID             |
|    `tags`    |                `List[str]`                |       List of possible tags      |
| `show_label` |                   `bool`                  |          Show label text         |
|    `size`    | `Literal["large", "small", "mini", None]` |           Selector size          |
|  `operation` |           `Literal["or", "and"]`          | Operation type (`"or"`, `"and"`) |
|  `widget_id` |                   `str`                   |         ID of the widget         |

### team\_id

Determine `Team` from which run sessions will be selected.

**type:** `int`

### tags

Determines list of possible tags to select run sessions. Tags are set in `config.json` file of the application in `session_tags` field.

**type:** `List[str]`

### show\_label

Determine show text `App Session` on widget or not.

**type:** `bool`

**default value:** `false`

```python
select_app_session = SelectAppSession(team_id=team_id, tags=["deployed_nn"], show_label=True)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219651794-16c3d78d-d3fe-49c2-ada5-1c5039c1e761.png" alt=""><figcaption></figcaption></figure>

### size

Determine selector size (large/small/mini/None).

**type:** `Literal["large", "small", "mini", None]`

**default value:** `mini`

```python
select_app_session = SelectAppSession(team_id=team_id, tags=["deployed_nn"])
select_app_small = SelectAppSession(team_id=team_id, tags=["deployed_nn"], size="small")
select_app_large = SelectAppSession(team_id=team_id, tags=["deployed_nn"], size="large")
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219652377-cd8392d6-09b7-432b-94a4-ef91ca64f864.png" alt=""><figcaption></figcaption></figure>

### operation

Determine operation type in select. Can be one of `"or"`, `"and"`. Setting the `operation` parameter to `"or"` allows users to connect to apps with any of the provided tags, while setting it to `"and"` allows them to connect to apps that have all the selected tags specified.

**type:** `str`

**default value:** `Literal["or", "and"]`

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                         |
| :--------------------: | ----------------------------------- |
|   `get_selected_id()`  | Return current selected session ID. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/selection/009\_select\_app\_session/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/009\_select\_app\_session/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, SelectAppSession
```

### Get `team_id` from environment variables

```python
team_id = sly.env.team_id()
```

### Initialize `SelectAppSession` widget

```python
select_app_session = SelectAppSession(team_id=team_id, tags=["deployed_nn"])
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select App Session",
    content=Container(widgets=[select_app_session]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219653528-f8748e91-22ca-4cfb-b6cb-bb27a9997f1c.gif" alt=""><figcaption></figcaption></figure>
