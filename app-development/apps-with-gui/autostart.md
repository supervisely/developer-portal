---
description: Autostart for your app with GUI and more
---

# Autostart

## Motivation

In some cases you want your application to become active right after the start as if it has no GUI at all, but with a possibility to change some parameters of a launched app later. For example serving app deploys the default model which can be changed later or a labeling app starts processing some test dataset before some big project is selected. For this purpose, a simple decorator `@sly.app.call_on_autostart()` was created.

## Decorator functionality

### Main

The decorator wraps the given function so that it can only be called if the special flag in the environment is setted. This flag can be set in the modal window of an application with a simple checkbox or any part of your code using `sly.env.set_autostart("1")` and removed with `sly.env.set_autostart(None)`.

<!-- TODO: add screen with modal window -->

### Callback

The decorator can take as input any function and its keyword arguments. This function will be called if flag is not defined.

```python
import supervisely as sly

def callback(arg1: str):
    print(f"Found an argument: {arg1}")

@sly.app.call_on_autostart(callback, arg1="U R AWESOME")
def main_function():
    print("Supervisely was here")


sly.env.set_autostart("1")
main_function()
sly.env.set_autostart(None)
main_function()

>>> Supervisely was here
>>> Found an argument: U R AWESOME
```

### Inference

Every subclass of `Inference` class can already deploy some default model without interaction with GUI (if any). During such model deployment, all parameters will be taken from the default widget values. To override default parameters you can use widget functionality. The example below is taken from [ClickSeg](https://github.com/supervisely-ecosystem/serve-clickseg/blob/master/src/main.py#L159) application.

```python
if sly.is_production() and not os.environ.get("DEBUG_WITH_SLY_NET"):
    m = ClickSegModel(use_gui=True, custom_inference_settings=inference_settings_path)
    # select default model
    m.gui._models_table.select_row(ClickSegModel.DEFAULT_ROW_IDX)
    m.serve()
```