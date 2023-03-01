# Field

## Introduction

In this tutorial you will learn how to use `Field`, `Field.Icon` widgets in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/field)

## Function signature

```python
Field(content, title, description=None, title_url=None, description_url=None, icon=None, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218447575-b8a874dd-4110-4386-9677-4cb3c1ddbbbd.png)

## Parameters

|    Parameters     |     Type     |                Description                 |
| :---------------: | :----------: | :----------------------------------------: |
|     `content`     |   `Widget`   |        Widget to display on `Field`        |
|      `title`      |    `str`     |          Determines `Field` title          |
|   `description`   |    `str`     |       Determines `Field` description       |
|    `title_url`    |    `str`     |    Determines url used on `Field` title    |
| `description_url` |    `str`     | Determines url used on `Field` description |
|      `icon`       | `Field.Icon` |          Determines `Field` icon           |
|    `widget_id`    |    `str`     |              Id of the widget              |

### content

Determine `Widget` to display on `Field`.

**type:** `Widget`

```python
button = Button(text="Button", icon="zmdi zmdi-plus-1")
field = Field(content=Button(), title="Field with button")
```

![content](https://user-images.githubusercontent.com/120389559/218450019-10bde8fd-a4ad-4320-96c7-2c6ae828ecad.png)

### title

Determines `Field` title.

**type:** `str`

```python
field = Field(content=Empty(), title="Field title")
```

![title](https://user-images.githubusercontent.com/120389559/218450471-b922323e-e00d-4981-ab8a-eab454e7679f.png)

### description

Determines `Field` description.

**type:** `str`

**default value:** `None`

```python
field = Field(content=Empty(), title="Field title", description="Field description")
```

![description](https://user-images.githubusercontent.com/120389559/218450851-009b957b-9915-4451-b8b9-7f20e14ef048.png)

### title_url

Determines url used on `Field` title.

**type:** `str`

**default value:** `None`

```python
field = Field(content=Empty(), title="Field title", title_url="https://i.imgur.com/0E8d8bB.png")
```

![title_url](https://user-images.githubusercontent.com/120389559/218451615-4b6dadc5-0a78-407d-a3e6-ef2cb808a28c.png)

### description_url

Determines url used on `Field` description.

**type:** `str`

**default value:** `None`

```python
field = Field(content=Empty(), title="Field title", description="Field description", description_url="https://i.imgur.com/0E8d8bB.png")
```

![description_url](https://user-images.githubusercontent.com/120389559/218452085-13c262f2-373d-40f0-8167-6d27b391825a.png)

### icon

Determines `Field` icon. Icons can be found at [zavoloklom.github.io](http://zavoloklom.github.io/material-design-iconic-font/icons.html).

**type:** `Field.Icon`

**default value:** `None`

```python
icon = Field.Icon(image_url="https://i.imgur.com/0E8d8bB.png")
field = Field(content=Empty(), title="Field title", icon=icon)
```

![icon](https://user-images.githubusercontent.com/120389559/218452703-a2801419-a910-4a33-848f-53b2589f50c0.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/036_field/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/036_field/src/main.py)

### Import libraries

```python
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Empty, Field
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Field` widget

```python
field = Field(content=Empty(), title="Field", description="Field description")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(title="Card", description="Card Description", content=field)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218455347-924a2423-f5f5-4770-8331-5007a7ddfa32.png)
