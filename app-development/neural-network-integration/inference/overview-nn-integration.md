# Integrate your custom NN model

In this tutorial series you will learn how to integrate your custom model into Supervisely by creating a simple serving app.&#x20;

✅ **Integration process is simple** - the only thing you need is to implement a method of how your model gets prediction of an image. Supervisely SDK will handle the rest automatically.

{% hint style="info" %}
If you are using popular machine learning frameworks, you can skip integration and start using already existing apps in [Supervisely Ecosystem](https://ecosystem.supervise.ly/). Most popular neural network frameworks are already integrated into Supervisely. Users can train these models on their data and test them (inference) right in the platform in a few clicks.

We highly recommend to explore apps in [Supervisely Ecosystem](https://ecosystem.supervise.ly/), here are several examples of ready-to-use frameworks for instance segmentation:

* MMDetection [![GitHub Org's stars](https://camo.githubusercontent.com/bf25a249878d6417d7ab913069e1868e6e1c56baa2ec4f6dd4c5806e6d9c578f/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f6f70656e2d6d6d6c61622f6d6d646574656374696f6e3f7374796c653d736f6369616c)](https://camo.githubusercontent.com/bf25a249878d6417d7ab913069e1868e6e1c56baa2ec4f6dd4c5806e6d9c578f/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f6f70656e2d6d6d6c61622f6d6d646574656374696f6e3f7374796c653d736f6369616c) - apps for [training](https://ecosystem.supervise.ly/apps/mmdetection/train) and [inference](https://ecosystem.supervise.ly/apps/mmdetection/serve)
* Detectron2 [![GitHub Org's stars](https://camo.githubusercontent.com/709465743709c522feb07a94a3a9598a3585cc3e2b54324cb4f7bdce107a6506/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f66616365626f6f6b72657365617263682f646574656374726f6e323f7374796c653d736f6369616c)](https://camo.githubusercontent.com/709465743709c522feb07a94a3a9598a3585cc3e2b54324cb4f7bdce107a6506/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f66616365626f6f6b72657365617263682f646574656374726f6e323f7374796c653d736f6369616c) - apps for [training](https://ecosystem.supervise.ly/apps/detectron2/supervisely/train) and [inference](https://ecosystem.supervise.ly/apps/detectron2/supervisely/instance\_segmentation/serve)

If your favorite NN framework is not in our Ecosystem yet, you can send us a feature request in [Supervisely Ideas Exchange](https://ideas.supervise.ly/).
{% endhint %}

## Benefits

Once you implement a serving application for your NN architecture, you can do a lot of things, like inference on your data for pre-labeling to speed up annotation, perform active learning, analyze and debug your model with various data science tools, combine models into pipelines and many more.&#x20;

Find more use cases and video tutorials on [our youtube channel](https://www.youtube.com/c/Supervisely).

{% hint style="success" %}
Generally speaking, your model will be compatible with the entire ecosystem of applications in Supervisely.
{% endhint %}

&#x20;Here are the examples of apps you might be interested to use with your model:

* [`Apply NN to Images Project` app](https://ecosystem.supervise.ly/apps/nn-image-labeling/project-dataset) - apply NN to your images and save predictions&#x20;
* [`NN Image Labeling` app](https://ecosystem.supervise.ly/apps/nn-image-labeling/annotation-tool) - use NN right in labeling interface
* [`Apply Detection and Classification Models to Images Project` app](https://ecosystem.supervise.ly/apps/apply-det-and-cls-models-to-project) - combine models into pipelines
* [`Apply NN to Videos Project` app](https://ecosystem.supervise.ly/apps/apply-nn-to-videos-project) - predict and track objects on videos
* Analyze model performance metrics ([app1](https://ecosystem.supervise.ly/apps/review\_object\_detection\_metrics/supervisely), [app2](https://ecosystem.supervise.ly/apps/semantic-segmentation-metrics-dashboard))
* and so on ...


### Inference with Python API

You can also get the model inference in one line of code via our Python API with the help of `sly.nn.inference.Session` class. See our [Inference API Tutorial](https://developer.supervise.ly/app-development/neural-network-integration/inference-api-tutorial).

**So, let's integrate Detectron2 instance segmentation model in the next section.**
