# Multiview images in the labeling interface

## Introduction

This easy-to-follow tutorial will show you how to upload multiview images to Supervisely using Python SDK and get the advantage of the grouped view in the labeling interface, which allows you to label images quickly and efficiently on one screen.

{% hint style="info" %}
You can also import grouped images using [Import Images Groups](https://ecosystem.supervisely.com/apps/import-images-groups) app from Supervisely Ecosystem.
{% endhint %}

You will learn how to enable multiview in the project settings, upload grouped images and explore the grouped view in the labeling interface.

## How to debug this tutorial

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/import-multiview-images-tutorial): source code and additional app files.
{% endhint %}

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../basics-of-authentication.md)

**Step 2.** Clone the [repository](https://github.com/supervisely-ecosystem/import-multiview-images-tutorial) with source code and demo data and create a [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/import-multiview-images-tutorial.git

cd import-multiview-images-tutorial

sh create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.

```
code -r .
```

**Step 4.** Change the Workspace ID in the `local.env` file by copying the ID from the context menu.

```python
WORKSPACE_ID=942 # ⬅️ change value
```

**Step 5.** Start debugging `src/main.py`.

{% hint style="info" %}
Supervisely instance version >= 6.8.54\
Supervisely SDK version >= 6.72.214

In the tutorial, Supervisely Python SDK version is not directly defined in the requirements.txt. But when developing your app, we recommend defining the SDK version in the requirements.txt.
{% endhint %}

### Import libraries

```python
import os

from dotenv import load_dotenv

import supervisely as sly
```

### Load environment variables

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

workspace_id = sly.env.workspace_id()
```

### Init API client

```python
api = sly.Api.from_env()
```

### Explore the directory with images

here is the structure of the directory with images (`src/images`):

```text
 📂 images
 ┣ 📂 audi
 ┃ ┣ 🏞️ audi_01.jpg
 ┃ ┣ 🏞️ audi_02.jpg
 ┃ ┗ 🏞️ audi_03.jpg
 ┣ 📂 mercedes
 ┃ ┣ 🏞️ mercedes_01.jpg
 ┃ ┣ 🏞️ mercedes_02.jpg
 ┃ ┣ 🏞️ mercedes_03.jpg
 ┃ ┣ 🏞️ mercedes_04.jpg
 ┃ ┣ 🏞️ mercedes_05.jpg
 ┃ ┗ 🏞️ mercedes_06.jpg
 ┣ 📂 renault
 ┃ ┣ 🏞️ renault_01.jpg
 ┃ ┣ 🏞️ renault_02.jpg
 ┃ ┣ 🏞️ renault_03.jpg
 ┃ ┗ 🏞️ renault_04.jpg
 ┗ 📂 ford
   ┣ 🏞️ ford_01.jpg
   ┣ 🏞️ ford_02.jpg
   ┣ 🏞️ ford_03.jpg
   ┣ 🏞️ ford_04.jpg
   ┗ 🏞️ ford_05.jpg
```

### Create a new project and dataset

```python
project = api.project.create(workspace_id, "Grouped cars", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, "ds0")
```

## Enable multiview in the project settings

```python
api.project.set_multiview_settings(project.id)
```

You can also enable multiview in the Image Labeling Tool interface:

![enable_multiview](https://github.com/supervisely-ecosystem/import-multiview-images-tutorial/assets/79905215/6c45e0d4-a79d-4cac-a529-f1be25e4b058)

And now we're ready to upload images.

## How to upload multiview images

In this tutorial, we'll be using the `api.image.upload_multiview_images` method to upload grouped images to Supervisely.

```python
def upload_multiview_images(
    dataset_id: int,
    group_name: str,
    paths: List[str],
    metas: Optional[List[Dict]] = None,
    progress_cb: Optional[Union[tqdm, Callable]] = None,
) -> List[ImageInfo]:
```

| Parameters  |                Type                 |              Description               |
| :---------: | :---------------------------------: | :------------------------------------: |
| dataset_id  |                 int                 |      ID of the dataset to upload       |
| group_name  |                 str                 |     Name of the group (tag value)      |
|    paths    |  List\[str\] (paths to the images)  |      List of paths to the images       |
|    metas    | List\[Dict\] (metas of the images)  |     List of image metas (optional)     |
| progress_cb | Optional\[Union\[tqdm, Callable\]\] | Function for tracking upload progress. |

So, the method uploads images to Supervisely and returns a list of `ImageInfo` objects.

## Upload multiview images

```python
for group_dir in os.scandir("src/images"):
    if not group_dir.is_dir():
        continue
    images_paths = sly.fs.list_files(group_dir.path, valid_extensions=sly.image.SUPPORTED_IMG_EXTS)

    api.image.upload_multiview_images(dataset.id, group_dir.name, images_paths)
```

## Grouped view in the labeling interface

So now, that we've uploaded all the images, let's take a look at the labeling interface.

![Grouped view in the labeling interface](https://github.com/supervisely-ecosystem/import-multiview-images-tutorial/assets/79905215/f8b7203a-cfbd-4771-a76e-22086d7b0d18)

As you can see, the images in the Labeling tool are grouped in the same way as in your images in folders (images from one folder are combined into one group). When importing, each image from the folders will be assigned tags with the same values, which allows them to be grouped into one group.

Multiview labeling can be very useful when annotating objects of multiple classes simultaneously on several images. You don't need to shift your attention to find the necessary class every time you switch between images, allowing you to increase efficiency and save time and effort.

![Multiview labeling](https://github.com/supervisely-ecosystem/import-multiview-images-tutorial/assets/79905215/772d1ca4-763f-4c77-bbd8-422d8e50f9ad)

## Summary

In this tutorial, you learned how to upload multiview images to Supervisely using Python SDK and get the advantage of the grouped view in the labeling interface, which allows you to label images quickly and efficiently on one screen. Let's recap the steps we did:

1. Create a new project and dataset.
2. Set multiview settings for the project using the `api.project.set_multiview_settings` method.
3. Upload images using the `api.image.upload_multiview_images` method.

And that's it! Now you can upload your multiview images to Supervisely using Python SDK.


**!!!!!!!!!!!!!! Poster image**

## Intoduction

In the world of machine learning, the quest for more accurate and robust models continues to drive innovation. It is crucially important for models to recognize objects not just in standard views but in various real-world scenarios, capturing multiple perspectives to **tackle everyday challenges efficiently**.

To achieve this, multi-view image annotation is gaining considerable attention. It leverages multiple perspectives, incorporates diverse data modalities and feature spaces. This enables learning algorithms to better comprehend and generalize visual data, identifying specific object features for improved recognition accuracy.

However, the process of annotating multiple images can be time-consuming and tedious, especially when you have to switch between images to annotate them separately.

That's where **Supervisely Image Labeling Tool** comes in and solves this by providing a convenient feature to annotate multiple images on one screen without switching between tabs, saving you time and effort.

**!!!!!!!!!!!!!! GIF with multiple images on one screen**

There are two ways to annotate images in Supervisely Image Labeling Tool:

- using the SmartTool to annotate images with AI assistance ([here is the guide on how to use Smart Tool](https://supervisely.com/blog/smarttool-annotation/))
- using [Bounding Box](https://supervisely.com/blog/bounding-box-annotation-for-object-detection/), [Mask Pen](https://supervisely.com/blog/mask-pen-tool/), [Polygon](https://supervisely.com/blog/how-to-use-polygon-anotation-tool-for-image-segmentation/), [Brush](https://supervisely.com/blog/brush/), Polyline, and Graph (keypoint) for manual labeling purposes (or to easily correct some cases).

## Video tutorial

In this video tutorial, you will learn how to import images and annotate them in Supervisely Labeling Tool using grouped display. Here's what we'll cover:

1. Importing groups of images into Supervisely

2. Exploring the multi-view display functionality in the Image Labeling Tool.

3. Manually annotating groups of images.

4. Speeding up the labeling process with AI-assistance using the [Supervisely Smart Tool](https://supervisely.com/blog/smarttool-annotation/)

**!!!!!!!!!!!!!! Youtube video tutorial**

## The reason why you should use grouped display

**Highlight the problem**:
Let's say you have a dataset with 500 images (100 scenes, each with 5 images from various angles) showcasing several object classes. The task at hand – annotating all these scenes – poses a challenge in other solutions, with constant image switching and class selection proving both time-consuming and mentally taxing.

**Here's where our solution shines:**
Just group images by tag values; for our example, assign corresponding tags to all 5 images of each scene (e.g., `scene_1`, `scene_2`, `scene_3`, etc.).
By activating the Multiple Image View Mode in Supervisely Image Labeling Tool, you can annotate multiple images simultaneously on one screen.
That's it! Now it is only 100 multi-view scenes instead of 500 separate images, and you don't need to switch between images and select the desired class for each object to annotate, keeping in mind all the details about the objects you are annotating.

Look how convenient and intuitive it can be, and in this tutorial we'll learn how to use it.

**!!!!!!!!!!!!!! GIF with grouped display**

## About Tags in Supervisely

If you need more than a bunch of marked pixels on an image and associate some extra information with annotations or files, you can use [tags](https://docs.supervisely.com/data-organization/projects/tags). Tags are key-value pairs that can be assigned to any object or image. Tags can be used to store any information about the object, such as its name, type, or any other properties of objects or images that you want to highlight.

In this use case, **string-type tags are required** to group images by tag values and annotate them simultaneously on one screen.

![tags](https://github.com/almazgimaev/sly-blog-multi-image-annotation-/assets/79905215/ecac6444-31c9-4e8b-b6a6-770c350161bb) <!-- !!! change this image -->

## How to work with Multi-view images in Supervisely

Supervisely provides a convenient workflow, from import using the Application or Python SDK to annotation with the Labeling Tool, enabling viewing and annotation of images as groups. Follow the brief guide below to import grouped images and multi-view annotate them in Supervisely.

## Step 1. Prepare Images for Import

- Organize your images into a simple project structure according to the application's [overview description](https://ecosystem.supervisely.com/apps/import-images-groups#Overview):

```text
📦 archive.zip
 ┗ 📂 project_name
   ┗ 📂 dataset_name
     ┣ 📂 group_name_1
     ┃ ┣ 🏞️ demo1.png
     ┃ ┣ 🏞️ demo2.png
     ┃ ┗ 🏞️ demo3.png
     ┣ 📂 group_name_2
     ┃ ┣ 🏞️ demo4.png
     ┃ ┣ 🏞️ demo5.png
     ┃ ┣ 🏞️ demo6.png
     ┃ ┣ 🏞️ demo7.png
     ┃ ┣ 🏞️ demo8.png
     ┃ ┗ 🏞️ demo9.png
     ┣ 🏞️ demo10.png
     ┣ 🏞️ demo11.png
     ┣ 🏞️ demo12.png
     ┗ 🏞️ demo13.png
```

In this example, we have 2 groups of images: `group_name_1` (3 images) and `group_name_2` (6 images). When you import this project, the application will automatically assign predefined tag to these images with `group_name_1` and `group_name_2` values respectively. Then, the application will group images by these tag values. The remaining 4 images are not grouped and will be imported without any tags.

- We have prepared 🔗 [demo data](https://github.com/supervisely-ecosystem/import-images-groups/releases/download/v0.0.1/cars.catalog.zip) for you, so it will help you to quickly reproduce the tutorial without a headache and get an experience and clear understanding of all the steps in this tutorial.

## Step 2. Import Images

After preparing your images for import, follow these steps to easily import image groups into Supervisely:

<blog-app github="import-images-groups/master"></blog-app>

1. Run the [Import images groups](https://ecosystem.supervisely.com/apps/import-images-groups?utm_source=blog) application.

2. Drag and drop the archive with your Project into the [application](https://ecosystem.supervisely.com/apps/import-images-groups?utm_source=blog) or upload it into the Team Files.

3. Click the `Run` button to start the import process.

![Running the import-images-groups application](https://github.com/almazgimaev/sly-blog-multi-image-annotation-/assets/79905215/9d6996c2-c994-447d-863d-72fb020c92d0)

Alternatively, you can also enable multiview in the Image Labeling Tool interface (corresponding tags should be assigned to images before):

![enable_multiview](https://github.com/supervisely-ecosystem/import-multiview-images-tutorial/assets/79905215/6c45e0d4-a79d-4cac-a529-f1be25e4b058)

## Step 3. Explore Multi-View Display

After importing images using the "Import images groups" app, you will see that images are grouped on the screen by tag values. Use `LEFT` and `RIGHT` arrow keys to navigate between groups.

![Grouped view in the labeling interface](https://github.com/supervisely-ecosystem/import-multiview-images-tutorial/assets/79905215/f8b7203a-cfbd-4771-a76e-22086d7b0d18)

As you can see, the images in the Labeling tool are grouped in the same way as in your images in folders (images from one folder are combined into one group). When importing, each image from the folders will be assigned tags with the same values, which allows them to be grouped into one group.

## Step 4. Multi-View Annotation

**⚡ Fast labeling with interactive AI assistance.**

Combine the power of AI and grouped displaying to annotate images faster and more efficiently. [Connect your computer with GPU](https://docs.supervisely.com/agents/connect-your-computer) and utilize popular pre-trained models for the Smart Labeling tool to improve efficiency

<!-- ![](./ai.gif) -->

![ai](https://github.com/almazgimaev/sly-blog-multi-image-annotation-/assets/79905215/950f60aa-901e-4760-97f2-0bc083c2be22)

The Smart Tool is a powerful tool that allows you to annotate images with AI assistance. It offers users the opportunity to utilize a variety of neural network algorithms integrated within the Supervisely platform. This encompasses robust models like [RITM](https://ecosystem.supervisely.com/apps/ritm-interactive-segmentation/supervisely?utm_source=blog), [Segment Anything](https://ecosystem.supervisely.com/apps/serve-segment-anything-hq/supervisely_integration/serve?utm_source=blog), and more, with ongoing efforts to enhance our [Ecosystem](https://ecosystem.supervisely.com/) through the integration of new models. It's essential to emphasize that the effectiveness, precision, and speed of segmentation are strongly influenced by the selection of the model. Therefore, we recommend that you try out different models to find the one that best suits your needs.

Read the guide on [how to use the Smart Tool](https://supervisely.com/blog/smarttool-annotation/) to annotate images with AI assistance.

![models](https://github.com/almazgimaev/sly-blog-multi-image-annotation-/assets/79905215/3610503d-5979-4359-96c1-67117fa798a5)

You can also train your model and use it in the Smart Tool. Explore blog posts dedicated to this topic:

- [How to Train Smart Tool for Precise Cracks Segmentation in Industrial Inspection](https://supervisely.com/blog/industrial-inspection-cracks-segmentation/)
- [Automate manual labeling with custom interactive segmentation model for agricultural images](https://supervisely.com/blog/custom-smarttool-wheat/)
- [Unleash The Power of Domain Adaptation - How to Train Perfect Segmentation Model on Synthetic Data with HRDA](https://supervisely.com/blog/unleash-the-power-of-domain-adaptation-with-HRDA-synthetic-cracks-segmentation/)
- [Lessons Learned From Training a Segmentation Model On Synthetic Data](https://supervisely.com/blog/lessons-learned-from-training-a-segmentation-model-on-synthetic-data/)

**Manual annotation**

Use different tools such as [Bounding Box](https://supervisely.com/blog/bounding-box-annotation-for-object-detection/), [Mask Pen](https://supervisely.com/blog/mask-pen-tool/), [Polygon](https://supervisely.com/blog/how-to-use-polygon-anotation-tool-for-image-segmentation/), [Brush](https://supervisely.com/blog/brush/), Polyline, and Graph (keypoint) for manual labeling purposes (✔️ or to easily correct some cases).

![manual](https://github.com/almazgimaev/sly-blog-multi-image-annotation-/assets/79905215/c9b4fba2-799e-4eba-ad5f-e4f657caaf9f)

## Collaborate teamwork

How else can you speed up the annotation process?

✅ Create a team and invite your colleagues to [labeling jobs](https://docs.supervisely.com/labeling/jobs), and work together on the same project.

Check out our blog [posts](https://supervisely.com/blog/tags/collaboration/) on how to effectively perform annotation at scale using Labeling Jobs, Labeling Queues and Labeling Consensus approaches.

Labeling Jobs and other collaboration tools in Supervisely helps to organize efficient work and complete tasks like:

1. Job management - the need to describe a particular task: what kind of objects to annotate and how
2. Progress monitoring - tracking annotation status and reviewing submitted results
3. Access permissions - limiting access only to specific datasets, classes, and **tags** within a single job
4. And what's more, you can take a screenshot for urgent tasks without using additional apps and quickly share the link.

## Automate workflow with Python SDK

You can also automate the process of working with image groups using Supervisely Python SDK.

```sh
pip install supervisely
```

You can learn more about it in our [Developer Portal](https://developer.supervisely.com/getting-started/python-sdk-tutorials/images/multiview-images), but here we'll just show how you can upload your image groups with just a few lines of code.

```python
project_id = 123456
dataset_id = 654321

# enable multi-view display in project settings
api.project.set_multiview_settings(project_id)

images_paths = ['path/to/audi_01.png', 'path/to/audi_02.png']

# upload group of images
api.image.upload_multiview_images(dataset_id, "audi", images_paths)
```

In the example above we uploaded two groups of images.
Before or after uploading images, we also need to enable image grouping in the project settings.

There's so much you can do with them using our Python SDK! You can find a set of Python SDK tutorials to work with images on our [Developer Portal](https://developer.supervisely.com/getting-started/python-sdk-tutorials/images).

### To sum up

Supervisely Image Labeling Tool is remarkably user-friendly, requiring minimal setup to get started. Its potential for further enhancements makes it stand out among competitors, providing a truly convenient solution for diverse use cases, including multi-view image annotation.
