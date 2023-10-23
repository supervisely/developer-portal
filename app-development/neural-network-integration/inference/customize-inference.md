# How to customize model inference

This document outlines some of the features of the `Inference` class that can be useful in customizing your model's behavior.

### Custom Inference Settings

Your neural network (NN) model can use various parameters that you may want to expose to the user. These parameters might include confidence threshold, intersection over union (IoU) threshold, and others. We have provided a way to allow users to set up some of these parameters when they connect to your model from the Inference dashboard or the Labeling Tool.

To support such parameters in your model, you should provide a dictionary or YAML file with default values to your model class, like this:

```python
settings = { 'confidence_threshold': 0.5 }
m = MyModel(model_dir=model_dir, custom_inference_settings=settings)
```

These parameters will be provided to the `predict(image_path, settings)` method as the `settings` parameter. In this method, you can use these parameters as needed, as shown in the following example from the [Integrate Instance Segmentation model repository](https://github.com/supervisely-ecosystem/integrate-inst-seg-model):

```python
def predict(self, image_path: str, settings: Dict[str, Any]) -> List[sly.nn.PredictionMask]:
    confidence_threshold = settings.get("confidence_threshold", 0.5)
    image = cv2.imread(image_path)  # BGR

    ####### CUSTOM CODE FOR MY MODEL STARTS (e.g. DETECTRON2) #######
    outputs = self.predictor(image)  # get predictions from Detectron2 model
    pred_classes = outputs["instances"].pred_classes.detach().cpu().numpy()
    pred_class_names = [self.class_names[pred_class] for pred_class in pred_classes]
    pred_scores = outputs["instances"].scores.detach().cpu().numpy().tolist()
    pred_masks = outputs["instances"].pred_masks.detach().cpu().numpy()
    ####### CUSTOM CODE FOR MY MODEL ENDS (e.g. DETECTRON2)  ########

    results = []
    for score, class_name, mask in zip(pred_scores, pred_class_names, pred_masks):
        # filter predictions by confidence
        if score >= confidence_threshold:
            results.append(sly.nn.PredictionMask(class_name, mask, score))
    return results
```

In this example, all predictions with a confidence score lower than the `confidence_threshold` parameter value will not be included in the result annotation.

If a user wants to change a parameter value before the inference, they can do so. For example, if they use the [Apply NN to Images Project](https://ecosystem.supervise.ly/apps/nn-image-labeling/project-dataset) app to label a project using your NN model, they will see these parameters in the `Inference settings` > `Additional settings` section.

![](https://user-images.githubusercontent.com/97401023/224495280-b3cf5b68-8120-4bb6-805f-fa7b13e3ded4.png)

After the user changes the parameter, the new value will be provided to your serving app in the request, and the `settings` dictionary will contain the new values in the `predict()` method.

If you want to add comments to your settings, as shown in the screenshot above, it is recommended to provide a path to a YAML file with comments to the `custom_inference_settings` parameter instead of a dictionary.

### Model Information

When users use your served model, they will connect to your app from another app, which sets up parameters to apply your neural network to data projects.

To ensure that users have chosen a suitable model for their task, it can be helpful to provide additional information about your served model. You can provide this information using the `get_info()` method, which returns a dictionary of parameters.

By default, these parameters are related to the chosen computer vision problem, but you can add additional information. We recommend overriding this method in your model class and adding such information as `model_name`, `checkpoint_name`, `pretrained_on_dataset`, and `device`. Of course, you can add any custom parameters.

```python
def get_info(self) -> dict:
    info = super().get_info()
    info["model_name"] = self.selected_model_name
    info["checkpoint_name"] = self.checkpoint_name
    info["pretrained_on_dataset"] = self.dataset_name
    info["device"] = self.device
    return info
```

It's important to understand that method `get_info()` of the `Inference` class calls method `get_classes()`, which is not implemented by default and must be declared explicitly for each model.

Also served model can provide additional info about its state through `model_meta` property. (e.g. description of dataset classes, type of predicted object). This data helps inference GUI and other supervisely applications to display correct model properties and visualize predictions. 

More information about model meta can be found [in this section](/app-development/neural-network-integration/inference-api-tutorial.md#model-meta-classes-and-tags). In [advanced tutorial](link-to-some-article) you can read about ways to define `model_meta` property of your custom model correctly.

### Sliding window mode

One problem with neural network model inference is that it can be challenging to apply them to large images with small objects. We provide tools to split the image into smaller parts, infer each part independently, and merge the results afterward.

This problem is significant for some computer vision tasks, but not for all. Therefore, it is crucial to consider this issue at the beginning.

We provide three modes to use sliding window:

- `none`
  
This means not to use sliding window and prevent users from setting up sliding window parameters from Inference interfaces. In this mode, you will get the path to the full image as the parameter `image_path` in the `predict(image_path, settings)` method.

- `basic`
  
In this mode, users can set up sliding window parameters, and you will get the path to a part of the image as the parameter `image_path` in the `predict(image_path, settings)` method.

In basic mode, all predictions are combined from all image parts into one result annotation.

- `advanced`
  
`basic` mode has the same problem as object detection neural networks without non-maximum suppression post-processing. Many labels from different image parts can collide, overlap and be split. We support the option to improve `basic` mode and add the [NMS post-processing](https://pytorch.org/vision/main/generated/torchvision.ops.nms.html) to predicted labels.

In `advanced` sliding window mode, you should implement the `predict_raw()` method which will predict the objects like `predict()` method, but they will be changed after by NMS algorithm. This approach is more appropriate for the object detection task.

You can refer to the [Serve YOLOv5 app repository](https://github.com/supervisely-ecosystem/yolov5/blob/master/supervisely/serve/src/main.py) as an example of using advanced sliding window mode.

To ensure that your app uses the required sliding window mode, see the class of the task from which your model class inherits (for example, `supervisely.nn.inference.InstanceSegmentation`) and check the `sliding_window_mode` parameter in the constructor of the class.

If you want to change this parameter in your model class, provide the correct mode value to the `sliding_window_mode` parameter of your model constructor:

```python
m = MyModel(sliding_window_mode="none")
```

### Model files storage

To simplify data manipulations, we support the `model_dir` parameter which is used as the location for all files needed for model inference. This folder must be provided to the `load_on_device(model_dir, device)` method to prepare your model.

You can also use the `download(src_path, dst_path)` method of the model class to download all the required files in the `load_on_device(model_dir, device)` method. Currently, you can provide external URLs or the path to a file or folder in Team Files as the src_path. By default, the destination path is `{model_dir}/{filename}`, but you can specify a different destination path to change the location or rename the downloaded file.

```python
model_name = 'MaskRCNN'
self.download(src_path=weights_url, dst_path=f'{model_dir}/{model_name}.pth')
```

It is recommended to provide the `model_dir` parameter in the constructor of your model to ensure that the `download()` method works correctly.
