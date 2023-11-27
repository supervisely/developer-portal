# Import multispectral images

## Introduction

In this tutorial, you will learn how to import multispectral images to Supervisely using Python SDK and get the advantage of the grouped view in the labeling interface, which allows you to synchronize zooming, panning and labeling of images in one group.

You will learn how to:
1. [Upload an RGB image as channels](#upload-an-rgb-image-as-channels)
2. [Upload a_multichannel tiff image as channels](#upload-a-multichannel-tiff-image-as-channels)
3. [Upload nrrd image as channels](#upload-nrrd-image-as-channels)
4. [Upload a pair of RGB and thermal images without splitting them into channels](#upload-a-pair-of-rgb-and-thermal-images-without-splitting-them-into-channels)
5. [Upload RGB image, its channels and depth image](#upload-rgb-image-its-channels-and-depth-image)
6. [Upload grayscale and UV images](#upload-grayscale-and-uv-images)
7. [Upload RGB image, thermal image and channels of thermal image](#upload-rgb-image-thermal-image-and-channels-of-thermal-image)
8. [Upload RGB image and two MRI images](#upload-rgb-image-and-two-mri-images)

📗 Everything you need to reproduce [this tutorial is on GitHub](!!!ADDRESS): source code and demo data.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../basics-of-authentication.md)

**Step 2.** Clone the [repository](!!!ADDRESS) with source code and demo data and create a [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone !!!ADDRESS

cd tutorial-image

./create_venv.sh
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

To enable grouping of images, including zooming, panning and labeling, you need to set multispectral settings for the project.
You can do it with just one line of code:

```python
api.project.set_multispectral_settings(project.id)
```

And now we're ready to upload images.

### Upload an RGB image as channels

**Input:** 1 RGB image in PNG format.
**Output:** 3 images in a group with the name `demo1.png` in Supervisely.


```python
image_name = "demo1.png"
image = cv2.imread(f"demo_data/{image_name}")

# Extract channels as 2d numpy arrays: channels = [a, b, c]
channels = [image[:, :, i] for i in range(image.shape[2])]

image_infos = api.image.upload_multispectral(dataset.id, image_name, channels)
print(f"Successfully uploaded {len(image_infos)} images (channels) from {image_name}")
```

We'll do this operation for other cases too, so let's take a closer look at the code:

1. The `image_name` will be used as a group name in the labeling interface and it can be any string.
2. Then we read the image from the disk using OpenCV.
3. Now we're splitting the image into channels, and preparing a list of channels as NumPy arrays.
4. Finally, we upload the channels to Supervisely using the `api.image.upload_multispectral` method.
5. The method returns a list of `ImageInfo` objects, which contain information about the uploaded images. Learn more about `ImageInfo` [here](https://supervisely.readthedocs.io/en/latest/sdk/supervisely.api.image_api.ImageInfo.html).

So, those are the steps we'll be doing for all the other cases.

### Upload a multichannel tiff image as channels

**Input:** 1 multichannel tiff image.
**Output:** 7 images in a group with the name `demo2.tif` in Supervisely.

```python
image_name = "demo2.tif"
image = tifffile.imread(f"demo_data/{image_name}")

# Extract channels as 2d numpy arrays: channels = [a, b, c, d, e, f]
channels = [image[:, :, i] for i in range(image.shape[2])]

image_infos = api.image.upload_multispectral(dataset.id, image_name, channels)
```