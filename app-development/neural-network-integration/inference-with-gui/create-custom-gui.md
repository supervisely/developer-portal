# How to create custom user interface

If the default GUI solution described in the previous articles do not cover your requests, you can create a completely custom GUI option.

## Implementation details

To implement your custom GUI, you need to create a new class derived from the `BaseInferenceGUI` class. This class contains required methods and properties, and you can implement as many additional functionalities as you need.

An instance of this class will be available in the user model class, inherited from the `Inference` class, as the `self.gui` property.

Let's consider some of the basic features that you need to implement.

### Create GUI class

First, declare the class derived from `BaseInferenceGUI` in your project. Alternatively, you can inherit from the `InferenceGUI` class, which provides you with our base functionality to be extended.


```python
import supervisely.nn.inference.gui as GUI
import supervisely.app.widgets as Widgets

class MyGUI(GUI.BaseInferenceGUI):
    pass
```

Next, you need to implement the required methods and properties:

1. `serve_button` property

This is a required property that is used to emit an event that starts model downloading and runs the `load_on_device()` method from the model class.

```python
@property
def serve_button(self) -> Widgets.Button:
    return self._serve_button
```

2. `download_progress` property

This required property is used to get an instance of the `Progress` widget to update it during model downloading in the `download(src, dst)` method in the model class.

```python
@property
def download_progress(self) -> Widgets.Progress:
    return self._download_progress
```

3. `get_device()` method

This method should return the device name, which will be provided as a parameter of the `load_on_device(model_dir, device)` method in the model class.

```python
def get_device(self) -> str:
    return "cuda:0"
```

4. `set_deployed()` method

This method will be called after the `load_on_device(model_dir, device)` method, and it is used to set the GUI in the state of a successfully deployed model.

```python
def set_deployed(self) -> None:
    self._serve_button.hide()
    self._success_notification_box.show()
```

5. `get_gui()` method

This method is required to provide the full container of your GUI widgets to display on the app's page.

```python
def get_ui(self) -> Widgets.Widget:
    return Widgets.Container([
        self._download_progress,
        self._serve_button,
        self._success_notification_box,
    ])
```

### Initialize new GUI in model

When your GUI class is implemented, you need to overwrite the GUI initialization in the model class inherited from the `Inference` class.

The main purpose of this method is to instantiate the `self._gui` field. If you use some of the parameters in the constructor, you have to provide them here.

```python
def initialize_gui(self) -> None:
    self._gui = MyGUI()
```

## Example

As an example of GUI customization, you can refer to the [gui.py](https://github.com/supervisely-ecosystem/mmdetection/blob/main/serve/src/gui.py) and [main.py](https://github.com/supervisely-ecosystem/mmdetection/blob/main/serve/src/main.py) files in [Serve MMDetection app](https://ecosystem.supervisely.com/apps/mmdetection/serve) repository.

In this example, the `MMDetectionGUI` class inherited from the `InferenceGUI` class to extend our base GUI template.