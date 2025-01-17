# Images

## Introduction

In this tutorial we will focus on working with images using Supervisely SDK.

You will learn how to:

1. [upload images from local directory to Supervisely dataset.](image.md#upload-images-from-local-directory-to-supervisely)
2. [upload images to Supervisely as NumPy matrix.](image.md#upload-images-as-numpy-matrix)
3. [get information about images by id or name.](image.md#get-information-about-images)
4. [download images from Supervisely to local directory.](image.md#download-images-to-local-directory)
5. [download images from Supervisely as NumPy matrix.](image.md#download-images-as-rgb-numpy-matrix)
6. [get and update image metadata](image.md#get-and-update-image-metadata)
7. [remove images from Supervisely.](image.md#remove-images-from-supervisely)
8. [custom image sorting for Image Labeling Toolbox](image.md#custom-image-sorting-for-image-labeling-toolbox)

📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-image): source code and demo data.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../basics-of-authentication.md)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/tutorial-image) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/tutorial-image.git

cd tutorial-image

./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.

```
code -r .
```

**Step 4.** Change workspace ID in `local.env` file by copying the ID from the context menu of the workspace.

```
WORKSPACE_ID=654 # ⬅️ change value
```

<figure><img src="https://user-images.githubusercontent.com/79905215/209327856-e47fb82b-c207-48fc-bb36-1fe795d45f6f.png" alt=""><figcaption></figcaption></figure>

**Step 5.** Start debugging `src/main.py`.

### Import libraries

```python
import os
from dotenv import load_dotenv
import supervisely as sly
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance.

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Get variables from environment

In this tutorial, you will need an workspace ID that you can get from environment variables. [Learn more here](../../environment-variables.md#workspace\_id)

```python
workspace_id = sly.env.workspace_id()
```

### Create new project and dataset

Create new project.

**Source code:**

```python
project = api.project.create(workspace_id, "Fruits", change_name_if_conflict=True)

print(f"Project ID: {project.id}")
```

**Output:**

```python
# Project ID: 15599
```

Create new dataset.

**Source code:**

```python
dataset = api.dataset.create(project.id, "Fruits ds1")

print(f"Dataset ID: {dataset.id}")
```

**Output:**

```python
# Dataset ID: 53465
```

## Upload images from local directory to Supervisely

### Upload single image.

**Source code:**

```python
original_dir = "src/images/original"
path = os.path.join(original_dir, "lemons.jpg")
meta = {"my-field-1": "my-value-1", "my-field-2": "my-value-2"}

image = api.image.upload_path(
    dataset.id,
    name="Lemons",
    path=path,
    meta=meta # optional
)

print(f'Image "{image.name}" uploaded to Supervisely with ID:{image.id}')
```

**Output:**

```python
# Image "Lemons.jpeg" uploaded to Supervisely platform with ID:17539453
```

<figure><img src="https://user-images.githubusercontent.com/79905215/209367792-2bd43e87-453f-4cba-9f41-9648a964658d.png" alt=""><figcaption></figcaption></figure>

### Upload list of images.

✅ Supervisely API allows uploading multiple images in a single request. The code sample below sends fewer requests and it leads to a significant speed-up of our original code.

**Source code:**

```python
names = [
    "grapes-1.jpg",
    "grapes-2.jpg",
    "oranges-2.jpg",
    "oranges-1.jpg",
]
paths = [os.path.join(original_dir, name) for name in names]

upload_info = api.image.upload_paths(dataset.id, names, paths)

print(f"{len(upload_info)} images successfully uploaded to Supervisely platform")
```

**Output:**

```python
# 4 images successfully uploaded to Supervisely platform
```

<figure><img src="https://user-images.githubusercontent.com/79905215/209367771-ff6d5852-f153-4529-9092-f58bcb45a3cc.png" alt=""><figcaption></figcaption></figure>

## Upload images as NumPy matrix

### Single image

**Source code:**

```python
img_np = sly.image.read(path)

np_image_info = api.image.upload_np(dataset.id, name="Lemons-np.jpeg", img=img_np)

print(f"Image successfully uploaded as NumPy matrix to Supervisely (ID: {np_image_info.id})")
```

**Output:**

```python
# Image successfully uploaded as NumPy matrix to Supervisely (ID: 17539458)
```

### Upload list of images

**Source code:**

```python
names_np = [f"np-{name}" for name in names]
images_np = [sly.image.read(img_path) for img_path in paths]

np_images_info = api.image.upload_nps(dataset.id, names_np, images_np)

print(f"{len(images_np)} images successfully uploaded to platform as NumPy matrix")
```

**Output:**

```python
# 4 images successfully uploaded to platform as NumPy matrix
```

## Get information about images

### Single image

Get information about image from Supervisely by id.

**Source code:**

```python
image_info = api.image.get_info_by_id(image.id)

print(image_info)
```

**Output:**

```python
# ImageInfo(
#     id=17539453,
#     name='Lemons.jpeg',
#     link=None,
#     hash='0jirgXvKGTJ8Yi0I9nCdf9MllQ9jP3Les1fD7/dt+Zk=',
#     mime='image/jpeg',
#     ext='jpeg',
#     size=66066,
#     width=640,
#     height=427,
#     labels_count=0,
#     dataset_id=54008,
#     created_at='2022-12-23T15:57:35.707Z',
#     updated_at='2022-12-23T15:57:35.707Z',
#     meta={},
#     path_original='nAXBxaxQJRARr0Ljkj6FfREj1Fq89.jpg',
#     full_storage_url='https://dev.supervisely.com/h5unublic/images/original/3/e/OK/d8Y7NnEj1Fq89.jpg',
#     tags=[]
# )
```

You can also get information about image from Supervisely by name.

**Source code:**

```python
image_name = get_file_name(image.name)

image_info_by_name = api.image.get_info_by_name(dataset.id, image_name)

print(f"image name - {image_info_by_name.name}")
```

**Output:**

```python
# image name - Lemons.jpeg
```

### Get all images from dataset.

Get information about image from Supervisely by id.

**Source code:**

```python
image_info_list = api.image.get_list(dataset.id)

print(f"{len(image_info_list)} images information received.")
```

**Output:**

```python
# 10 images information received.
```

## Download images to local directory

### Single image

Download image from Supervisely to local directory by id.

**Source code:**

```python
save_path = os.path.join(result_dir, image_info.name)

api.image.download_path(image_info.id, save_path)

print(f"Image has been successfully downloaded to '{save_path}'")
```

**Output:**

```python
# Image has been successfully downloaded to 'src/images/result/Lemons.jpeg'
```

### Download list of images to local directory

Download list of images from Supervisely to local directory by ids.

**Source code:**

```python
image_ids = [img.id for img in image_info_list]
image_names = [img.name for img in image_info_list]
save_paths = [os.path.join(result_dir, img_name) for img_name in image_names]

api.image.download_paths(dataset.id, image_ids, save_paths)

print(f"{len(image_info_list)} images has been successfully downloaded.")
```

**Output:**

```python
# 10 images has been successfully downloaded.
```

<figure><img src="https://user-images.githubusercontent.com/79905215/209375238-9c6050f2-439f-4bac-a4b7-2b6ecbe03313.png" alt=""><figcaption></figcaption></figure>

## Download images as RGB NumPy matrix

### Single image

Download image from Supervisely to local directory by id.

**Source code:**

```python
image_np = api.image.download_np(image_info.id)

print(f"Image downloaded as RGB NumPy matrix. Image shape: {image_np.shape}")
```

**Output:**

```python
# Image downloaded as RGB NumPy matrix. Image shape: (427, 640, 3)
```

### Download list of images as RGB NumPy matrix

Download list of images from Supervisely to local directory by ids.

**Source code:**

```python
image_np = api.image.download_nps(dataset.id, image_ids)

print(f"{len(image_np)} images downloaded in RGB NumPy matrix.")
```

**Output:**

```python
# 10 images downloaded in RGB NumPy matrix.
```

## Get and update image metadata

### Get image metadata from server

**Source code:**

```python
image_info = api.image.get_info_by_id(image.id)
meta = image_info.meta

print(meta)
```

**Output:**

```python
# {'my-field-1': 'my-value-1', 'my-field-2': 'my-value-2'}
```

### Update image metadata

**Source code:**

```python
new_meta = {"my-field-3": "my-value-3", "my-field-4": "my-value-4"}

new_image_info = api.image.update_meta(id=image.id, meta=new_meta)

print(new_image_info["meta"])
```

**Output:**

```python
# {'my-field-3': 'my-value-3', 'my-field-4': 'my-value-4'}
```

### Get metadata in Image labeling toolbox

You can also get image metadata in Image labeling toolbox interface

<figure><img src="https://user-images.githubusercontent.com/79905215/209392054-4ceafec9-747b-4a26-8570-5ec52c0f23f0.gif" alt=""><figcaption></figcaption></figure>

## Remove images from Supervisely

### Remove one image.

Remove image from Supervisely by id

**Source code:**

```python
api.image.remove(image.id)

print(f"Image (ID: {image.id}) successfully removed")
```

**Output:**

```python
# Image (ID: 17539453) successfully removed
```

### Remove list of images.

Remove list of images from Supervisely by ids.

**Source code:**

```python
images_to_remove = api.image.get_list(dataset.id)
remove_ids = [img.id for img in images_to_remove]

api.image.remove_batch(remove_ids)

print(f"{len(remove_ids)} images successfully removed.")
```

**Output:**

```python
# 9 images successfully removed.
```

## Custom image sorting for Image Labeling Toolbox

To enhance the usability of working with images in the Image Labeling Toolbox, a custom sorting parameter can be added for project images. This parameter will define the order of images in the interface list.

<figure><img src="https://github.com/user-attachments/assets/c4e08ee7-97fc-4ec5-92ef-d0ba9c138c2e" alt=""><figcaption></figcaption></figure>

1. Sort button 
2. Sorting parameter


### Upload list of images with added custom sort parameter

The best and fastest way to accomplish this is to use context manager `ImageApi.add_custom_sort`
This context manager allows you to set the `sort_by` attribute of `ImageApi` object for the duration of the context, then delete it.
If nested functions support this functionality, each image they process will automatically receive a custom sorting parameter based on the available meta object.
<br>Currently, almost all image uploading methods support this functionality. Methods that support it have a corresponding description in the docstring.

**Source code:**

```python
original_dir = "src/images/original"
names = ["Oranges 1", "Oranges 2"]
paths = [os.path.join(original_dir, "oranges-1.jpg"), os.path.join(original_dir, "oranges-2.jpg")]
metas = [{"key-1": "a", "my-key": "b"}, {"key-1": "c", "my-key": "f"}]

with api.image.add_custom_sort(key="my-key"):
    image_infos = api.image.upload_paths(
        dataset.id,
        names=names,
        paths=paths,
        metas=metas
    )
for i in image_infos:
    print(f"{i.name}: {i.meta}")
```

**Output:**

```python
# Oranges 1.jpeg: {'key-1': 'a', 'my-key': 'b', 'customSort': 'b'}
# Oranges 2.jpeg: {'key-1': 'c', 'my-key': 'f', 'customSort': 'f'}
```

### Upload whole images project in Supervisely format with added custom sort parameter

It is also recommended to use a context manager for uploading the entire project. 
The only difference from the previous point is that there is no need to pass meta in dictionaries. 
It can be stored either in image info files or in meta files within the project structure. 
To learn more about the project structure and its files, see the [Project Structure](../../supervisely-annotation-format/project-structure.md) section.

**Source code:**

```python
from supervisely.project.upload import upload

project_dir = "src/images_project"
project_name = "Project with Sorting"

with api.image.add_custom_sort(key="my-key"):
    upload(project_dir, api, workspace_id, project_name)

project_info = api.project.get_info_by_name(workspace_id, project_name)
dataset_info = api.dataset.get_list(project_info.id)[0]
images_infos = api.image.get_list(dataset_info.id)
for i in images_infos:
    print(f"{i.name}: {i.meta}")
```

**Output:**

```python
# oranges-2.jpg: {'my-key': '5', 'customSort': '5'}
# oranges-1.jpg: {'my-key': '4', 'customSort': '4'}
# grapes-2.jpg: {'my-key': '1', 'customSort': '1'}
# lemons.jpg: {'my-key': '5', 'customSort': '5'}
# grapes-1.jpg: {'my-key': '2', 'customSort': '2'}
```

### Add custom sort parameter to meta object

Here are several ways to modify meta for images

#### 1. Add parameter to meta dict and update meta on server

**Source code:**

```python
meta = {"key-1": "a", "my-key": "b"}
new_meta = api.image.update_custom_sort(meta, "sort-value")
new_image_info = api.image.update_meta(id=images_infos[0].id, meta=new_meta)

print(new_image_info["meta"])
```

**Output:**

```python
# {'key-1': 'a', 'my-key': 'b', 'customSort': 'sort-value'}
```

#### 2. Set directly on server
**Source code:**

```python
api.image.set_custom_sort(new_image_info["id"], "new-sort-value")
updated_image_info = api.image.get_info_by_id(new_image_info["id"])

print(updated_image_info.meta)
```

**Output:**

```python
# {'key-1': 'a', 'my-key': 'b', 'customSort': 'new-sort-value'}
```

#### 3. Set directly on server in bulk

Same as the previous case, but for more than one image

**Source code:**

```python
image_ids = [image.id for image in images_infos]
sort_values = ["1st", "2nd", "3rd", "4th", "5th"]
api.image.set_custom_sort_bulk(image_ids, sort_values)
images_infos = api.image.get_list(dataset_info.id)
for i in images_infos:
    print(f"{i.name}: {i.meta}")
```

**Output:**

```python
# oranges-2.jpg: {'key-1': 'a', 'my-key': 'b', 'customSort': '1st'}
# oranges-1.jpg: {'my-key': '4', 'customSort': '2nd'}
# grapes-2.jpg: {'my-key': '1', 'customSort': '3rd'}
# lemons.jpg: {'my-key': '5', 'customSort': '4th'}
# grapes-1.jpg: {'my-key': '2', 'customSort': '5th'}
```