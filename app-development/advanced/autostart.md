---
description: Autostart for your app with GUI and more
---

# Autostart

## Motivation

There are situations when you want your application to launch immediately and not require any manipulation in GUI, but with a possibility to change some parameters of a launched app later. For example serving app deploys the default model which can be changed later or a labeling app starts processing some test dataset before some big project is selected. For this purpose, a simple decorator `@sly.app.call_on_autostart()` was created.

## Decorator functionality

### Main

The decorator wraps the given function so that it can only be called if the special flag in the environment is setted. This flag can be set in the modal window of an application with a simple checkbox or any part of your code using `sly.env.set_autostart("1")` and removed with `sly.env.set_autostart(None)`. If you want your application to always start with setted `autostart` flag, add this in `config.json`:

```json
"modal_template_state": {
    "autostart": true
}
```

<!-- TODO: add screen with modal window -->

### Simple example

Let's create a simple app with an activate-deactivate button. Imagine that button launch some other program. We want to indicate a successful launch by changing of button color from blue to green.

```python
import os
from dotenv import load_dotenv

import supervisely as sly
from supervisely.app.widgets import Container, Button, Card

load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

# blue button if not activated
activate_btn = Button("Activate")

# green button if activated successfully
deactivate_btn = Button("Deactivate", button_type="success")
deactivate_btn.hide()
```

Add some logic and create layout for app:

```python
# activation
@activate_btn.click
def activate_process():
    # launch some background task
    activate_btn.hide()
    deactivate_btn.show()

#deactivation
@deactivate_btn.click
def deactivate_process():
    # stop some background task
    deactivate_btn.hide()
    activate_btn.show()

btns = Container([activate_btn, deactivate_btn])
layout = Card("Activate-deactivate process", content=btns)
app = sly.Application(layout=layout)
```

At startup, we will see an inactive (blue) button which can be activated by clicking on it.

<figure><img src="https://github.com/supervisely/developer-portal/assets/87002239/aaefe954-9f48-40dc-b144-49437a3a40f0" alt=""><figcaption></figcaption></figure>

Now we would like to change the state of the button to active without interaction with GUI. Let's add `activate_on_autostart()` function, wrap it with `sly.app.call_on_autostart()` decorator and call at the end of our program. 

```python
@sly.app.call_on_autostart()
def activate_on_autostart():
    # launch some background task
    activate_btn.hide()
    deactivate_btn.show()

activate_on_autostart()
```

We will see that there are no changes in GUI.

![Not activated](https://github.com/supervisely/developer-portal/assets/87002239/75e06b6f-1602-4f37-bb08-51926aceefe7)

This is because the environment variable `autostart` is not set. Add the environment setting before the call of `activate_on_autostart()`.

```python
sly.env.set_autostart("1")
activate_on_autostart()
```

Now our button will become active (green) immediately after start and we can control this behavior with `autostart` environment.

![Activated](https://github.com/supervisely/developer-portal/assets/87002239/7e0173e5-04de-4b0a-8dfe-d64eeffa2bc5)

The final code:

```python
import os
from dotenv import load_dotenv

import supervisely as sly
from supervisely.app.widgets import Container, Button, Card

load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

# blue button if not activated
activate_btn = Button("Activate")

# green button if activated successfully
deactivate_btn = Button("Deactivate", button_type="success")
deactivate_btn.hide()


# activation
@activate_btn.click
def activate_process():
    # launch some background task
    activate_btn.hide()
    deactivate_btn.show()


#deactivation
@deactivate_btn.click
def deactivate_process():
    # stop some background task
    deactivate_btn.hide()
    activate_btn.show()


btns = Container([activate_btn, deactivate_btn])
layout = Card("Activate-deactivate process", content=btns)
app = sly.Application(layout=layout)


@sly.app.call_on_autostart()
def activate_on_autostart():
    # launch some background task
    activate_btn.hide()
    deactivate_btn.show()


sly.env.set_autostart("1")
activate_on_autostart()
```


### Inference
Every subclass of `Inference` class can already deploy some default model without interaction with GUI (if any). If you use default `InferenceGUI` (or your GUI inherits from it), all parameters will be taken from the default widget values while autostarting. To override default parameters you can use widget functionality. The example below is taken from [ClickSeg](https://github.com/supervisely-ecosystem/serve-clickseg/blob/master/src/main.py#L159) application: we override the default value of the `RadioTable` widget by calling `m.gui._models_table.select_row()`

```python
if sly.is_production() and not os.environ.get("DEBUG_WITH_SLY_NET"):
    m = ClickSegModel(use_gui=True, custom_inference_settings=inference_settings_path)
    # Change default parameter from 0 to DEFAULT_ROW_IDX
    m.gui._models_table.select_row(ClickSegModel.DEFAULT_ROW_IDX)
    m.serve()
```