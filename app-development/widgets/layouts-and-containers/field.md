# Field

## Introduction

**`Field`** widget within Supervisely is a type of form which has the ability to contain various other widgets. This feature enables users to organize multiple widgets within a layout. Additionally, users have the option to customize the widget by setting a `title`, `icon`, `title_url` and `description`.

## Function signature

```python
Field(
    content=Empty(),
    title="Field",
    description=None,
    title_url=None,
    description_url=None,
    icon=None,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218447575-b8a874dd-4110-4386-9677-4cb3c1ddbbbd.png" alt=""><figcaption></figcaption></figure>

## Parameters

|     Parameters    |     Type     |                 Description                |
| :---------------: | :----------: | :----------------------------------------: |
|     `content`     |   `Widget`   |        Widget to display on `Field`        |
|      `title`      |     `str`    |          Determines `Field` title          |
|   `description`   |     `str`    |       Determines `Field` description       |
|    `title_url`    |     `str`    |    Determines url used on `Field` title    |
| `description_url` |     `str`    | Determines url used on `Field` description |
|       `icon`      | `Field.Icon` |           Determines `Field` icon          |
|    `widget_id`    |     `str`    |              ID of the widget              |

### content

Determine `Widget` to display on `Field`.

**type:** `Widget`

```python
field = Field(
    content=Button(text="Button"),
    title="Field with button",
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218450019-10bde8fd-a4ad-4320-96c7-2c6ae828ecad.png" alt=""><figcaption></figcaption></figure>

### title

Determines `Field` title.

**type:** `str`

```python
field = Field(
    content=Empty(),
    title="Field title",
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218450471-b922323e-e00d-4981-ab8a-eab454e7679f.png" alt=""><figcaption></figcaption></figure>

### description

Determines `Field` description.

**type:** `str`

**default value:** `None`

```python
field = Field(
    content=Empty(),
    title="Field title",
    description="Field description",
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218450851-009b957b-9915-4451-b8b9-7f20e14ef048.png" alt=""><figcaption></figcaption></figure>

### title\_url

Determines url used on `Field` title.

**type:** `str`

**default value:** `None`

```python
field = Field(
    content=Empty(),
    title="Field title",
    title_url="https://i.imgur.com/0E8d8bB.png",
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218451615-4b6dadc5-0a78-407d-a3e6-ef2cb808a28c.png" alt=""><figcaption></figcaption></figure>

### description\_url

Determines url used on `Field` description.

**type:** `str`

**default value:** `None`

```python
field = Field(
    content=Empty(),
    title="Field title",
    description="Field description",
    description_url="https://i.imgur.com/0E8d8bB.png",
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218452085-13c262f2-373d-40f0-8167-6d27b391825a.png" alt=""><figcaption></figcaption></figure>

### icon

Determines `Field` icon. Icons can be found at [zavoloklom.github.io](http://zavoloklom.github.io/material-design-iconic-font/icons.html).

**type:** `Field.Icon`

**default value:** `None`

```python
icon = Field.Icon(image_url="https://i.imgur.com/0E8d8bB.png")
field = Field(content=Empty(), title="Field title", icon=icon)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218452703-a2801419-a910-4a33-848f-53b2589f50c0.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/layouts and containers/004\_field/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/004\_field/src/main.py)

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

<figure><img src="https://user-images.githubusercontent.com/79905215/223954574-02327395-0449-40ff-9479-5900e746243a.png" alt=""><figcaption></figcaption></figure>
