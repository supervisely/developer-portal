# Multispectral images

## Introduction

In this tutorial, you will learn how to import multispectral images to Supervisely using Python SDK and get the advantage of the grouped view in the labeling interface, which allows you to synchronize the view, zooming, panning, and labeling of images in one group.

{% hint style="info" %}
You can also import multispectral images using [Import Multispectral Images](https://ecosystem.supervisely.com/apps/import-multispectral-images) app from Supervisely Ecosystem.
{% endhint %}

You will learn how to:

1. [Upload an image as channels](multispectral-images.md#upload-an-image-as-channels)
2. [Upload a multichannel tiff image as channels](multispectral-images.md#upload-a-multichannel-tiff-image-as-channels)
3. [Upload nrrd image as channels](multispectral-images.md#upload-nrrd-image-as-channels)
4. [Upload a pair of RGB and thermal images without splitting them into channels](multispectral-images.md#upload-a-pair-of-rgb-and-thermal-images-without-splitting-them-into-channels)
5. [Upload RGB image, its channels and depth image](multispectral-images.md#upload-rgb-image-its-channels-and-depth-image)
6. [Upload grayscale and UV images](multispectral-images.md#upload-grayscale-and-uv-images)
7. [Upload RGB image, thermal image and channels of thermal image](multispectral-images.md#upload-rgb-image-thermal-image-and-channels-of-thermal-image)
8. [Upload RGB image and two MRI images](multispectral-images.md#upload-rgb-image-and-two-mri-images)

For the [advanced level](multispectral-images.md#advanced-use-supervisely-format-for-multispectral-images), you can use supervisely annotation JSON format to download and upload projects with multispectral images.

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/import-multispectral-images-tutorial): source code and additional app files.
{% endhint %}

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../basics-of-authentication.md)

**Step 2.** Clone the [repository](https://github.com/supervisely-ecosystem/import-multispectral-images-tutorial) with source code and demo data and create a [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/import-multispectral-images-tutorial.git

cd import-multispectral-images-tutorial

sh create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.

```
code -r .
```

**Step 4.** Change the Team ID and Workspace ID in the `main.py` file by copying the ID from the context menu.

```python
team_id = 448
workspace_id = 690
```

**Step 5.** Start debugging `src/main.py`.

{% hint style="info" %}
Supervisely instance version >= 6.8.54\
Supervisely SDK version >= 6.72.201\\

In the tutorial, Supervisely Python SDK version is not directly defined in the requirements.txt. But when developing your app, we recommend defining the SDK version in the requirements.txt.
{% endhint %}

### Import libraries

```python
import cv2
import os
import tifffile
import supervisely as sly
import nrrd
from dotenv import load_dotenv
```

### Enter your Team ID and Workspace ID

```python
team_id = 448
workspace_id = 690
```

### Init API client

```python
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api.from_env()
```

### Create a new project and dataset

```python
project = api.project.create(workspace_id, "Multispectral images", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, "ds0")
```

### Set multispectral settings for the project

To enable grouping of images, including view, zooming, panning and labeling, you need to set multispectral settings for the project. You can do it with just one line of code:

```python
api.project.set_multispectral_settings(project.id)
```

And now we're ready to upload images.

## How to upload multispectral images

In this tutorial, we'll be using the `api.image.upload_multispectral` method to upload images to Supervisely.

```python
def upload_multispectral(
        dataset_id: int,
        image_name: str,
        channels: Optional[List[np.ndarray]] = None,
        rgb_images: Optional[List[str]] = None,
    ) -> List[ImageInfo]:
```

|  Parameters |                 Type                |                         Description                        |
| :---------: | :---------------------------------: | :--------------------------------------------------------: |
| dataset\_id |                 int                 |                 ID of the dataset to upload                |
| image\_name |                 str                 | Name of the image will be used as name of the images group |
|   channels  | Optional\[List\[np.ndarray]] = None |            List of channels as 2d numpy arrays.            |
| rgb\_images |     Optional\[List\[str]] = None    |                List of paths to RGB images.                |

So, the method uploads images (which can be passed as channels or RGB images) to Supervisely and returns a list of `ImageInfo` objects. RGB images as paths or channels as NumPy arrays can be passed to the method or both at the same time. The result will be a group of images in both cases.

### Upload an image as channels

**Input:** 1 RGB image in PNG format.\
**Output:** 3 images in a group with the name `demo1.png` in Supervisely.\\

![RGB image as channels](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/286217467-f29e3cf8-8a0e-467e-af14-4c89bf1638ea.png)

```python
image_name = "demo1.png"
image = cv2.imread(f"demo_data/{image_name}")

# Extract channels as 2d numpy arrays: channels = [a, b, c]
channels = [image[:, :, i] for i in range(image.shape[2])]

image_infos = api.image.upload_multispectral(dataset.id, image_name, channels)
```

We'll do this operation for other cases too, so let's take a closer look at the code:

1. The `image_name` will be used as a group name in the labeling interface and it can be any string.
2. Then we read the image from the disk using OpenCV.
3. Now we're splitting the image into channels, and preparing a list of channels as NumPy arrays.
4. Finally, we upload the channels to Supervisely using the `api.image.upload_multispectral` method.
5. The method returns a list of `ImageInfo` objects, which contain information about the uploaded images. Learn more about `ImageInfo` [here](https://supervisely.readthedocs.io/en/latest/sdk/supervisely.api.image\_api.ImageInfo.html).

So, those are the steps we'll be doing for all the other cases.

### Upload a multichannel tiff image as channels

**Input:** 1 multichannel tiff image.\
**Output:** 7 images in a group with the name `demo2.tif` in Supervisely.\\

![Multichannel tiff image as channels](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/286217497-34c4bf68-16f4-4475-82d1-2a3ae2b3adb9.png)

```python
image_name = "demo2.tif"
image = tifffile.imread(f"demo_data/{image_name}")

# Extract channels as 2d numpy arrays: channels = [a, b, c, d, e, f]
channels = [image[:, :, i] for i in range(image.shape[2])]

image_infos = api.image.upload_multispectral(dataset.id, image_name, channels)
```

### Upload nrrd image as channels

{% hint style="info" %}
In this tutorial, we'll be uploading the channels of nrrd image as separate images. But Supervisely supports high-dimensional nrrd images, so you can upload them as is without splitting them into channels.
{% endhint %}

**Input:** 1 nrrd image.\
**Output:** 7 images in a group with the name `demo3.nrrd` in Supervisely.\\

![Nrrd image as channels](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/286217509-06963544-b04d-481e-980e-98bba9d113b1.png)

```python
image_name = "demo3.nrrd"
image, header = nrrd.read(f"demo_data/{image_name}")

# Extract channels as 2d numpy arrays: channels = [a, b, c, d, e, f]
channels = [image[:, :, i] for i in range(image.shape[2])]

image_infos = api.image.upload_multispectral(dataset.id, image_name, channels)
```

### Upload a pair of RGB and thermal images without splitting them into channels

**Input:** 1 RGB image and 1 thermal image in PNG format.\
**Output:** 2 images in a group with the name `demo4.png` in Supervisely.\\

![RGB and thermal images without splitting them into channels](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/286217514-47b840a2-6d9f-4bda-9cc5-772bda3c891a.png)

```python
image_name = "demo4.png"
images = ["demo_data/demo4-rgb.png", "demo_data/demo4-thermal.png"]

image_infos = api.image.upload_multispectral(dataset.id, image_name, rgb_images=images)
```

As you can see, in this case, we don't extract any channels since we need to upload only images, not channels. So, we pass the list of image paths to the `rgb_images` parameter.

### Upload RGB image, its channels, and depth image

**Input:** 1 RGB image and 1 depth image in PNG format.\
**Output:** 5 images in a group with the name `demo5.png` in Supervisely.\\

![RGB image, its channels and depth image](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/286217523-fdf0f3e3-7ea1-49e4-a923-25b7f9339d5f.png)

```python
image_name = "demo5.png"
images = ["demo_data/demo5-rgb.png", "demo_data/demo5-depths.png"]

image = cv2.imread(images[0])

# Extract channels as 2d numpy arrays: channels = [a, b, c]
channels = [image[:, :, i] for i in range(image.shape[2])]

image_infos = api.image.upload_multispectral(dataset.id, image_name, channels, images)
```

Here, we uploaded one image both as channels and as an image. So, we pass the list of image paths to the `rgb_images` parameter and the list of channels to the `channels` parameter.

### Upload grayscale and UV images

**Input:** 1 grayscale image and 1 UV image in PNG format.\
**Output:** 2 images in a group with the name `demo6.png` in Supervisely.\\

![Grayscale and UV images](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/286217527-1c8d2fab-9b8d-4e1e-9225-6b2ae560f637.png)

```python
image_name = "demo6.png"
images = ["demo_data/demo6-grayscale.png", "demo_data/demo6-uv.png"]

image_infos = api.image.upload_multispectral(dataset.id, image_name, rgb_images=images)
```

### Upload RGB image, thermal image, and channels of thermal image

**Input:** 1 RGB image and 1 thermal image in PNG format.\
**Output:** 5 images in a group with the name `demo7.png` in Supervisely.\\

![RGB image, thermal image and channels of thermal image](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/286217532-c6551cd6-d207-4d13-a9ce-9b31d3e57b8f.png)

```python
image_name = "demo7.png"
images = ["demo_data/demo7-rgb.png", "demo_data/demo7-thermal.png"]

image = cv2.imread(images[1])

# Extract channels as 2d numpy arrays: channels = [a, b, c]
channels = [image[:, :, i] for i in range(image.shape[2])]

image_infos = api.image.upload_multispectral(dataset.id, image_name, channels, images)
```

### Upload RGB image and two MRI images

**Input:** 1 RGB image and 2 MRI images in PNG format.\
**Output:** 3 images in a group with the name `demo8.png` in Supervisely.\\

![RGB image and two MRI images](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/286217543-204c0631-7ade-475e-b8bf-68ebb143f00d.png)

```python
image_name = "demo8.png"
images = ["demo_data/demo8-rgb.png", "demo_data/demo8-mri1.png", "demo_data/demo8-mri2.png"]

image_infos = api.image.upload_multispectral(dataset.id, image_name, rgb_images=images)
```

## Grouped view in the labeling interface

So now, that we've uploaded all the images, let's take a look at the labeling interface.

![Grouped view in the labeling interface](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/286261123-a07db05b-a5fe-4f9b-891e-8db99179b1b9.gif)

As you can see, all the images are grouped by the name of the group, which is the name of the image we passed to the `image_name` parameter. We can zoom, pan, and label images in one group at the same time. So, whenever you create a label on one image, it will be automatically created on all the other images in the group. You can edit label on another image in the group, and it will be automatically updated on all the other images in the group. Just a reminder: we set the multispectral settings for the project at the beginning of the tutorial with the `api.project.set_multispectral_settings` method, which enables this grouped view.

## Summary

In this tutorial, you learned how to upload multispectral images to Supervisely using Python SDK and get the advantage of the grouped view in the labeling interface, which allows you to synchronize the view, zooming, panning, and labeling of images in one group. Let's recap the steps we did:

1. Create a new project and dataset.
2. Set multispectral settings for the project using the `api.project.set_multispectral_settings` method.
3. Upload images using the `api.image.upload_multispectral` method.

And that's it! Now you can upload your multispectral images to Supervisely using Python SDK.

## (Advanced) Use supervisely format for multispectral images

{% hint style="info" %}
You can always use [Export to Supervisely format](https://ecosystem.supervisely.com/apps/export-to-supervisely-format) to download to the local directory of your favorite multispectral project with the preserved multi-view settings and then easily upload it as a new project to the [Import images in Supervisely format](https://ecosystem.supervisely.com/apps/import-images-in-sly-format)
{% endhint %}

From the developer's point of view, the [Supervisely annotation JSON format](broken-reference) gives you easy access to the necessary parameters while grouping the images. To feel the power of this instrument, let's imagine the situation when you have already downloaded the project and opened the `meta.json` file:

```json
{
  "classes": [
    {
      "title": "leaf",
      "shape": "bitmap",
      "color": "#FB00ED",
      "geometry_config": {},
      "id": 6509759,
      "hotkey": ""
    }
  ],
  "tags": [
    {
      "name": "im_id",
      "value_type": "any_number",
      "color": "#0F8A2D",
      "id": 27855,
      "hotkey": "",
      "applicable_type": "all",
      "classes": []
    },
    {
      "name": "band",
      "value_type": "any_string",
      "color": "#9F7A2F",
      "id": 27856,
      "hotkey": "",
      "applicable_type": "all",
      "classes": []
    }    
  ],
  "projectType": "images",
  "projectSettings": {
    "multiView": {
      "enabled": true,
      "tagName": "im_id",
      "tagId": 27855,
      "isSynced": false
    }
  }
}
```

This is very easy to see, if you want to group your images by the `band` tag instead of `im_id` (sync mode on), simply change your `projectSettings` this way:

```json
  "projectSettings": {
    "multiView": {
      "enabled": true,
      "tagName": "band",
      "tagId": 27856,
      "isSynced": true
    }
  }
```

Moreover, you can additionally enhance your tags with `hotkey`, or specify the tag `classes`. See the explanation of every field [here](broken-reference).

To download and upload a project using Supervisely SDK, use the following code:

```python
import supervisely as sly
from tqdm import tqdm

load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api.from_env()
# id of your workspace and project from environmental variables
# see https://developer.supervisely.com/getting-started/environment-variables 
WORKSPACE_ID = sly.env.workspace_id()
PROJECT_ID = sly.env.project_id()

your_dir = "/your/multi-view/project/dir"

project_info = api.project.get_info_by_id(PROJECT_ID)
pbar = tqdm(desc="Download Project", total=project_info.items_count)
sly.download_project(api, PROJECT_ID, your_dir, progress_cb=pbar)

project_fs = sly.read_project(your_dir)
pbar = tqdm(desc="Upload Project", total=project_fs.total_items)
sly.upload_project(project_fs.directory, api, WORKSPACE_ID, progress_cb=pbar)
```
