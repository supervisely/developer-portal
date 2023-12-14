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
Supervisely SDK version >= 6.72.214\

In the tutorial, Supervisely Python SDK version is not directly defined in the requirements.txt. But when developing your app, we recommend defining the SDK version in the requirements.txt.
{% endhint %}

### Import libraries

```python
import os

from dotenv import load_dotenv

import supervisely as sly
```

### Load variables from environment

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

There are 2 more ways to enable multiview in the project settings:

- in the project settings (create tag and enable the `Group Images mode` option and select the tag you created)
- in the Image Laleling Tool (enable `Group Images by Tag` mode in the `More` menu and select the tag you created before)

And now we're ready to upload images.

## How to upload grouped images

In this tutorial, we'll be using the `api.image.upload_multiview_images` method to upload images groups to Supervisely.

```python
for group_dir in os.scandir("src/images"):
    if not group_dir.is_dir():
        continue
    images_paths = sly.fs.list_files(group_dir.path, valid_extensions=sly.image.SUPPORTED_IMG_EXTS)

    api.image.upload_multiview_images(dataset.id, group_dir.name, images_paths)
```

| Parameters  |                Type                 |               Description               |
| :---------: | :---------------------------------: | :-------------------------------------: |
| dataset_id  |                 int                 |       ID of the dataset to upload       |
| group_name  |                 str                 |      Name of the group (tag value)      |
|    paths    |  List\[str\] (paths to the images)  |       List of paths to the images       |
|    metas    | List\[Dict\] (metas of the images)  |     List of image metas (optional)      |
| progress_cb | Optional\[Union\[tqdm, Callable\]\] | Function for tracking upload progress.  |

So, the method uploads images to Supervisely and returns a list of `ImageInfo` objects.

## Grouped view in the labeling interface

So now, that we've uploaded all the images, let's take a look at the labeling interface.

![Grouped view in the labeling interface](https://github.com/supervisely-ecosystem/import-multiview-images-tutorial/assets/79905215/772d1ca4-763f-4c77-bbd8-422d8e50f9ad)

As you can see, all the images are grouped by the name of the group, which is the name of the image we passed to the `group_name` parameter.
Note that the images are grouped by the tag value, not by the tag name.

## Summary

In this tutorial, you learned how to upload multiview images to Supervisely using Python SDK and get the advantage of the grouped view in the labeling interface, which allows you to label images quickly and efficiently on one screen. Let's recap the steps we did:

1. Create a new project and dataset.
2. Set multiview settings for the project using the `api.project.set_multiview_settings` method.
3. Upload images using the `api.image.upload_multiview_images` method.

And that's it! Now you can upload your multiview images to Supervisely using Python SDK.
