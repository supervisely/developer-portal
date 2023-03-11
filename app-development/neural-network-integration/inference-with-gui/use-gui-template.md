# How to use default GUI template

This tutorial demonstrates how to add a basic user interface to your integrated neural network using a default GUI template. As an example, we will consider a repository with an instance segmentation model and transform it to an app with GUI.

## interface appearance

Firstly, let's take a look at how the interface appears:

![](https://user-images.githubusercontent.com/97401023/224482058-def9bcad-c889-4532-8060-4ce6f574187d.png) 

### Table with models info
Developers can provide any model information in a simple form and display it on this table.

### Device selector
The template will automatically detect all available devices.

### "SERVE" button
After clicking the button, the model weights downloading process will begin, and you will see a progress bar. After successful downloading, the `SERVE` button will disappear, and the `STOP AND CHOOSE ANOTHER MODEL` button will appear instead. You can change the served model without rebooting the session.

## Tutorial

In this tutorial, we will show you how to add a user interface to your model app, and we will compare how it works with and without the GUI using the [Integrate instance segmentation model with GUI repository](https://github.com/supervisely-ecosystem/integrate-inst-seg-model-gui) and [Integrate instance segmentation model without GUI](https://github.com/supervisely-ecosystem/integrate-inst-seg-model) repositories, respectively.

Firstly, clone this repository and launch it on your computer to ensure that the code works correctly.

### Clone and launch

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/integrate-inst-seg-model-gui) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/integrate-inst-seg-model
cd integrate-inst-seg-model
./create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 4.** Run the debug configuration `Local debug with GUI (run model)` in VSCode.

![](https://user-images.githubusercontent.com/97401023/224483196-e4e9d3b2-bd0e-4bdb-b5ab-12c64c58317c.png)

**Step 5.** Open `http://0.0.0.0:8000` in your browser (or just click on appeared link in VSCode terminal).

![](https://user-images.githubusercontent.com/97401023/224483217-a39e0c8f-1c20-49f4-9178-7d0f6bf31e27.png)

**Step 6.** Choose and serve the model in the web browser on the selected device.

!()[https://user-images.githubusercontent.com/97401023/224483240-537c79ae-ca06-4578-be92-16e88e27c0c5.png]

**Step 7.** Run the next debug configuration `Local debug with GUI (query)` in VSCode.
For local run, you have to use one script for a model with GUI and another script to create a query with an image to run the inference.

![](https://user-images.githubusercontent.com/97401023/224483252-36affaf7-8827-40fe-b188-7578214ad9b2.png)

**Step 8.** Check result image `./demo_data/image_01_prediction.jpg` to ensure that everything works correctly.

![](https://user-images.githubusercontent.com/97401023/224483273-feb94282-89b3-4e12-afa8-c10ddf110038.png)

## Implementation details

Apps with an integrated model without and within GUI have a few differences.

1. To support GUI, you have to provide flag `use_gui=True` to constructor of your model.
```python
m = MyModel(model_dir=model_dir, custom_inference_settings=settings, use_gui=True)
```

2. If you want to place a list of pretrained weights to choose from on the app page, you have to implement the method `get_models()` which returns list of models to show in the table. 

```python
def get_models(self) -> List[Dict[str, str]]:
    mask_rcnn_R_50_C4_1x = {
        "Model": "R50-C4 (1x)",
        "Dataset": "COCO",
        "Train_time": "0.584",
        "Inference_time": "0.110",
        "Box AP score": "36.8",
        "Mask AP score": "32.2",
    }

    mask_rcnn_R_50_DC5_3x = {
        "Model": "R50-DC5 (3x)",
        "Dataset": "COCO",
        "Train_time": "0.470",
        "Inference_time": "0.076",
        "Box AP score": "40.0",
        "Mask AP score": "35.9",
    }
    return [mask_rcnn_R_50_C4_1x, mask_rcnn_R_50_DC5_3x]
```

3. Model method `load_on_device(model_dir, device)` will be called automatically in model with GUI with provided `model_dir` and `device` got from user selection in interface. You don't have to call this method manually.

In the example, we have to download weights of selected model. We do this with next strings in `load_on_device()` methods:

```python
selected_model = self.gui.get_checkpoint_info()
weights_path, config_path = self.download_pretrained_files(selected_model, model_dir)
```

`selected_model` gets `dict` of model data list from `get_models()` method, corresponding to selected model in GUI.

Method `download_pretrained_files()` implemented here to download selected model weights and get model config.

## Release

To add your serving app with a GUI to Supervisely, follow the same steps as [the tutorial without GUI](../inference/instance-segmentation.md)
The only difference is that you need to ensure that the `headless` parameter in your `config.json` file is set to `false` or removed.