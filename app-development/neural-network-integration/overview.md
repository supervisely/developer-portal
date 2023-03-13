# NN integration into Supervisely

In the upcoming tutorial series we will cover all the ways of how you can bring together your custom NN model and all the benefits of the Supervisely platform.

There are several ways how to do that with different tradeoffs of the amount of work needed to be done and the depth of integration with the Supervisely platform. We suggest the following roadmap:

## The levels of the NN integration:
**from simple python scripts to fully-featured training dashboards**

### Level 0. Run your model locally
Write a python script that tests your model locally on several images. It's a good point to start before actual integration. This initial step allows you to check the model predictions and make sure that your inference code is correct and you can visualize the predictions as well. As a result you will have just a python script.

### Level 1. Run the model on a Supervisely Project via API
Write a python script that downloads images from your Supervisely project via API, gets the model predictions for the images, converts it to Supervisely format and uploads it back to the platform into a new project. At this step you can modify the script from level 0. This way you'll get the basic experience of working with the Supervisely API in your python code. We have a number of tutorials for that:
- [Iterating projects and datasets](https://developer.supervise.ly/getting-started/python-sdk-tutorials/iterate-over-a-project)
- [Downloading and Uploading images to the platform](https://developer.supervise.ly/getting-started/python-sdk-tutorials/image)
- [Labeling an image from code](https://developer.supervise.ly/getting-started/python-sdk-tutorials/spatial-labels)

### Level 2. Integrate the model as a Serving App
Implement your model as a subclass of one of the Supervisely base classes for integration. At this point your model starts to be a regular Serving App that you've may already used in Supervisely. Generally speaking, your model will be compatible with the entire ecosystem of applications in Supervisely.
- Check the in-depth tutorial: [NN Integration Intro](https://developer.supervise.ly/app-development/neural-network-integration/inference/overview-nn-integration)
- Also you can check already existing serving apps. Their source code are publicly available and can be utilized as additional examples: [NN Integration Example](https://github.com/supervisely-ecosystem/integrate-inst-seg-model), [Serve Detectron2](https://github.com/supervisely-ecosystem/detectron2/tree/main/supervisely/instance_segmentation/serve)

### Level 3. Integrate the Inference along with the Training App
Implement both the Inference App and the Training App as a pair of Supervisely Apps with a user-friendly GUI and a flexible settings for all your needs.
- The in-depth tutorial: [Training dashboard](https://developer.supervise.ly/app-development/neural-network-integration/training/training-dashboard)
