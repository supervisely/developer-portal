# Customize default GUI template

We provide some features which allow you customize our basic gui template for your case.
More detailed about using our GUI template [here](use-gui-template.md).

## Options

### Support Pretrained and Custom models

We support two ways how to download neural network weights: external URLs (Pretrained) and internal path from Team Files (Custom).
If your models got from your [Training App](), artifacts will be uploaded to Team Files and your Serving App can use the same files structure and be compatible with Training App. Your app just have to support Custom models.
If your model weights and configs saved outside the Supervisely, you can just use Pretrained models to provide detailed info in GUI about each model.
By default, both modes are active and shown in GUI as two tabs. If you don't want to use Pretrained models, just don't implement `get_models()` method in you Model class. If you don't want to use Custom models, override method `support_custom_models()` to return False like below:

```python
def support_custom_models(self) -> bool:
        return False
```

### Support link to File or Folder in Team File
For Custom Models you can set link type.
Default value is `file`.
If you want to use link to folder in Team File and download directory to use in your Serving App, override the method `get_custom_model_link_type()` to return `folder` value.

```python
def get_custom_model_link_type(self) -> Literal["file", "folder"]:
    return "folder"
```

### Custom UI content in Pretrained Models and Custom Models tab

Also default GUI template supports insertion of any Supervisely [Widgets] to Pretrained models and Custom models tabs. Custom widgets block placed at the bottom of tab.

You can use this for example to provide more info or media content about your models in Serving App.

If you want to add some additional info to Pretrained models tab, you can override the method `add_content_to_pretrained_tab()`:

```python
def add_content_to_pretrained_tab(self, gui: GUI.BaseInferenceGUI) -> sly.app.widgets.Widget:
    return sly.app.widgets.NotificationBox("some text")
```

If you want to add some additional info to Custom models tab, you can override the method `add_content_to_custom_tab()`:

```python
def add_content_to_custom_tab(self, gui: GUI.BaseInferenceGUI) -> sly.app.widgets.Widget:
    return sly.app.widgets.NotificationBox("some text")
```

You can use existing GUI content in your custom insertions because of `gui` paramerer provided here.
For example, you can subscribe on changing selection in models table to change somwthing in your custom widgets block.

As an example, you can see file [main.py](https://github.com/supervisely-ecosystem/vitpose/blob/master/serve/src/main.py) in repository of [Serve ViTPose app]().

### Nested model lists

If you have the case when you have some pretrained model architectures and also each architecture contain some of pretrained weights (for example, on different datasets or with different parameters), we support Nested models.

In this case Select field will be added to choose the architecture and model table will contain checkpoints of this architecture.

[screenshot]()

You can change method `get_models()` to use Nested Models.
Default format of method is `List[Dict[str, str]]`. For using Nested models required format is:
`Dict[str, Dict[str, List[Dict[str, str]]]]`.

How to looks?

```python
{
    "model_name_1": {
        "paper_from": "ICCV",
        "year": "2021",
        "checkpoints": [
            {
                "Name": "checkpoint_name_1",
                "Dataset": "COCO",
                ...
            },
            {
                "Name": "checkpoint_name_2",
                ...
            }
        ]
    },
    "model_name_2": {
        ...
    }
}
```

`checkpoints`, `paper_from` and `year` are reserved names in our GUI Template to show this as right text in Select field with models.

## GUI Template Methods

You can use some our methods in your model logic, for example, in `load_on_device()` method to get info from GUI from user.

### gui.get_checkpoint_info()

This method will return the dict of selected checkpoint from model table. It works only if Pretrained models supported in your App.

```python
checkpoint_info = self.gui.get_checkpoint_info()
```

### gui.get_model_info()

This method are useful only If you use Nested Models.
It return Dict in format:
{`selected_architecture_name`: `checkpoint_info`}
where `selected_architecture_name` is name of model from Select field 
`checkpoint_info` is result of method `gui.get_checkpoint_info()`.

```python
model_info = self.gui.get_model_info()
```

### gui.get_device()

Method will return name of selected device to run the model. For example, `cpu` or `cuda:0`.
The result of this method will be automatically provided to parameter `device` in `load_on_device(model_dir, device)` method.

```python
device = self.gui.get_device()
```

### gui.get_model_source()

Method will return the type of tab user selected - `Pretrained models` or `Custom models`.

```python
source_type = self.gui.get_model_source()
```

### gui.get_custom link()

If model source is `Custom models`, this method return the link to file or folder from Team Files.

```python
custom_model_link = self.gui.get_custom_link()
```

## Example

For more details you can see file [main.py](https://github.com/supervisely-ecosystem/mmsegmentation/blob/main/serve/src/main.py) in repository of Serve MMDetection app.