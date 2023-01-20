# Table of contents

* [💻 Supervisely Developer Portal](README.md)

## 🎉 Getting Started

* [Installation](getting-started/installation.md)
* [Basics of authentication](getting-started/basics-of-authentication.md)
* [Intro to Python SDK](getting-started/intro-to-python-sdk.md)
* [Environment variables](getting-started/environment-variables.md)
* [Python SDK tutorials](getting-started/python-sdk-tutorials/README.md)
  * [Iterate over a project](getting-started/python-sdk-tutorials/iterate-over-a-project.md)
  * [Spatial labels](getting-started/python-sdk-tutorials/spatial-labels.md)
  * [Keypoints (skeletons)](getting-started/python-sdk-tutorials/keypoints.md)
  * [Image and object tags](getting-started/python-sdk-tutorials/image-and-object-tags.md)
  * [Images](getting-started/python-sdk-tutorials/image.md)
  * [Videos](getting-started/python-sdk-tutorials/video.md)
  * [Pointclouds (LiDAR)](getting-started/python-sdk-tutorials/point-clouds-and-episodes.md)
  * [Volumes (DICOM)](getting-started/python-sdk-tutorials/volumes.md)

## 🔥 App development

* [Basics](app-development/basics/README.md)
  * [Create app from any py-script](app-development/basics/from-script-to-supervisely-app.md)
  * [Configuration file](app-development/basics/app-json-config/README.md)
    * [Example 1. Headless](app-development/basics/app-json-config/example-1.-headless.md)
    * [Example 2. App with GUI](app-development/basics/app-json-config/example-2.-app-with-gui.md)
    * [v1 - Legacy](app-development/basics/app-json-config/v1-legacy/README.md)
      * [Example 1. v1 Modal Window](app-development/basics/app-json-config/v1-legacy/example-1.-v1-modal-window.md)
      * [Example 2. v1 app with GUI](app-development/basics/app-json-config/v1-legacy/example-2.-v1-app-with-gui.md)
  * [Add private app](app-development/basics/add-private-app.md)
* [Apps with GUI](app-development/apps-with-gui/README.md)
  * [Hello World!](app-development/apps-with-gui/hello-world.md)
  * [Progress Bar](app-development/apps-with-gui/progress-bar.md)
* [Neural Network integration](app-development/neural-network-integration/README.md)
  * [Instance segmentation](app-development/neural-network-integration/instance-segmentation.md)
* [Advanced](app-development/advanced/README.md)
  * [How to make your own widget](app-development/advanced/how-to-make-your-own-widget.md)
  * [Legacy tutorial](app-development/advanced/in-depth-app-development/README.md)
    * [Chapter 1 Headless](app-development/advanced/in-depth-app-development/chapter-1-headless/README.md)
      * [Part 1 — Hello world! \[From your Python script to Supervisely APP\]](app-development/advanced/in-depth-app-development/chapter-1-headless/part-1-hello-world-from-your-python-script-to-supervisely-app.md)
      * [Part 2 — Errors handling \[Catching all bugs\]](app-development/advanced/in-depth-app-development/chapter-1-headless/part-2-errors-handling-catching-all-bugs.md)
      * [Part 3 — Site Packages \[Customize your app\]](app-development/advanced/in-depth-app-development/chapter-1-headless/part-3-site-packages-customize-your-app.md)
      * [Part 4 — SDK Preview \[Lemons counter app\]](app-development/advanced/in-depth-app-development/chapter-1-headless/part-4-sdk-preview-lemons-counter-app.md)
      * [Part 5 — Integrate custom tracker into Videos Annotator tool \[OpenCV Tracker\]](app-development/advanced/in-depth-app-development/chapter-1-headless/part-5-integrate-custom-tracker-into-videos-annotator-tool-opencv-tracker.md)
    * [Chapter 2 Modal Window](app-development/advanced/in-depth-app-development/chapter-2-modal-window/README.md)
      * [Part 1 — Modal window \[What is it?\]](app-development/advanced/in-depth-app-development/chapter-2-modal-window/part-1-modal-window-what-is-it.md)
      * [Part 2 — States and Widgets \[Customize modal window\]](app-development/advanced/in-depth-app-development/chapter-2-modal-window/part-2-states-and-widgets-customize-modal-window.md)
    * [Chapter 3 UI](app-development/advanced/in-depth-app-development/chapter-3-ui/README.md)
      * [Part 1 — While True Script \[It's all what you need\]](app-development/advanced/in-depth-app-development/chapter-3-ui/part-1-while-true-script-its-all-what-you-need.md)
      * [Part 2 — UI Rendering \[Simplest UI Application\]](app-development/advanced/in-depth-app-development/chapter-3-ui/part-2-ui-rendering-simplest-ui-application.md)
      * [Part 3 — APP Handlers \[Handle Events and Errors\]](app-development/advanced/in-depth-app-development/chapter-3-ui/part-3-app-handlers-handle-events-and-errors.md)
      * [Part 4 — State and Data \[Mutable Fields\]](app-development/advanced/in-depth-app-development/chapter-3-ui/part-4-state-and-data-mutable-fields.md)
      * [Part 5 — Styling your app \[Customizing the UI\]](app-development/advanced/in-depth-app-development/chapter-3-ui/part-5-styling-your-app-customizing-the-ui.md)
    * [Chapter 4 Additionals](app-development/advanced/in-depth-app-development/chapter-4-additionals/README.md)
      * [Part 1 — Remote Developing with PyCharm \[Docker SSH Server\]](app-development/advanced/in-depth-app-development/chapter-4-additionals/part-1-remote-developing-with-pycharm-docker-ssh-server.md)

## 😎 Advanced user guide

* [Objects binding](advanced-user-guide/objects-binding.md)
* [Automate with Python SDK & API](advanced-user-guide/automate-with-python-sdk-and-api/README.md)
  * [Start and stop app](advanced-user-guide/automate-with-python-sdk-and-api/start-and-stop-app.md)
  * [User management](advanced-user-guide/automate-with-python-sdk-and-api/user-management.md)
  * [Labeling Jobs](advanced-user-guide/automate-with-python-sdk-and-api/labeling-jobs.md)

## 🖥 UI widgets

* [Element UI library](https://element.eleme.io/1.4/#/en-US/component/button)
* [Supervisely UI widgets](https://ecosystem.supervise.ly/docs/table)
* [Apexcharts - modern & interactive charts](https://apexcharts.com/)
* [Plotly graphing library](https://plotly.com/python/)

## 📚 API References

* [Supervisely annotation JSON format](api-references/supervisely-annotation-json-format/README.md)
  * [Project Structure](api-references/supervisely-annotation-json-format/project-structure.md)
  * [Project Classes and Tags](api-references/supervisely-annotation-json-format/project-classes-and-tags.md)
  * [Tags](api-references/supervisely-annotation-json-format/tags.md)
  * [Objects](api-references/supervisely-annotation-json-format/objects.md)
  * [Individual Image Annotations](api-references/supervisely-annotation-json-format/individual-image-annotations.md)
  * [Individual Video Annotations](api-references/supervisely-annotation-json-format/individual-video-annotations.md)
  * [Point Clouds](api-references/supervisely-annotation-json-format/point-clouds.md)
  * [Point Cloud Episodes](api-references/supervisely-annotation-json-format/point-cloud-episodes.md)
  * [Volumes Annotation](api-references/supervisely-annotation-json-format/volumes-annotation.md)
* [REST API Reference](https://api.docs.supervise.ly/)
* [Python SDK Reference](https://supervisely.readthedocs.io/en/latest/sdk\_packages.html)
