# CheckboxField

`CheckboxField` is similar to a [Checkbox](../002_checkbox/README.md) widget, but it is used in a form field. It allows to specify a title and a description for the checkbox.

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/widgets/controls/checkboxfield)

### Function signature

```python
CheckboxField(
    title="Title",
    description="Description",
    checked=False,
    widget_id=None,
)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/dd0f609d-06cf-4b35-a79c-4cda345dfffb" alt=""><figcaption></figcaption></figure>

### Parameters

|  Parameters   |       Type       |             Description              |
| :-----------: | :--------------: | :----------------------------------: |
|    `title`    |      `str`       |            Checkbox title            |
| `description` |      `str`       |         Checkbox description         |
|   `checked`   | `Optional[bool]` | Return `True` if checkbox is checked |
|  `widget_id`  | `Optional[str]`  |           ID of the widget           |

### title

Checkbox title.

**type:** `str`

### description

Checkbox description.

**type:** `str`

```python
checkbox = CheckboxField(
    title="Title",
    description="Description",
)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/dd0f609d-06cf-4b35-a79c-4cda345dfffb" alt=""><figcaption></figcaption></figure>

### checked

Whether Checkbox is checked.

**type:** `bool`

**default value:** `False`

```python
checkbox = CheckboxField(
    title="Title",
    description="Description",
    checked=True,
)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/5bbfe4d6-260a-4077-b539-dc97a37c97b9" alt=""><figcaption></figcaption></figure>

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

### Methods and attributes

| Attributes and Methods | Description                                                   |
| :--------------------: | ------------------------------------------------------------- |
|     `is_checked()`     | Return `True` if checked, else `False`.                       |
|        `set()`         | Set `title`, `description`, and `checked` properties.         |
|       `check()`        | Enable `checked` property.                                    |
|      `uncheck()`       | Disable `checked` property.                                   |
|    `@value_changed`    | Decorator function is handled when checkbox value is changed. |

### is_checked()

Return `True` if checked, else `False`.

```python
checkbox = CheckboxField(
    title="Title",
    description="Description",
)
checkbox.is_checked() # False
```

### set()

Set `title`, `description`, and `checked` properties.

```python
checkbox.set(title="New title", description="New description", checked=True)
```

### check()

Enable `checked` property.

```python
checkbox.check()
```

### uncheck()

Disable `checked` property.

```python
checkbox.uncheck()
```

### @value_changed

Decorator function is handled when checkbox value is changed.

```python
@checkbox.value_changed
def on_checkbox_changed(checked):
    print(f"Checkbox is checked: {checked}")
```

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/controls/010_checkbox_field/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/controls/010_checkbox_field/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, CheckboxField
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `CheckboxField` widget

```python
checkbox_field = CheckboxField(
    title="Title",
    description="Description",
)
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="CheckboxField",
    content=checkbox_field,
)
```

### Create the app using a layout

Create an app object with a layout parameter.

```python
app = sly.Application(layout=card)
```

### Add function to control widget from code

```python
@checkbox_field.value_changed
def get_value(is_checked):
    print(is_checked)
    if is_checked:
        checkbox_field.set(title="Is Checked", checked=is_checked)
    else:
        checkbox_field.set(title="Is Not Checked", checked=is_checked)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/2ce587c4-66ec-40d4-9b48-1f407641a6c4" alt=""><figcaption></figcaption></figure>
