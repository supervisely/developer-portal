---
description: How to create Mask3D annotations on volumes in Python
---

# 3D annotations on volumes

## Introduction

In this tutorial, you will learn how to programmatically create 3D annotations for volumes and upload them to Supervisely platform. 

Supervisely supports different types of shapes/geometries for volume annotation:

* 🆕 Mask3D - ❤️‍🔥 Recommended Type
* Mask (also known as Bitmap) - 🗃️ Legacy Type
* Bounding Box (Rectangle) - 🗃️ Legacy Type
* Polygon - 🗃️ Legacy Type

Learn more about [Supervisely Annotation in JSON format](https://developer.supervisely.com/api-references/supervisely-annotation-json-format/volumes-annotation) for volumes.

You can also use ready-made solutions for labeling. Read about how it works in our blog article [Best DICOM & NIfTI annotation tools for Medical Imaging AI](https://supervisely.com/blog/dicom-labeling-toolbox/) and take full advantage of our platform.

![Mask3D and Masks](https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/384c7ffc-5eb5-4920-8f02-022e59a136c8)

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/dicom-spatial-figures): source code, Visual Studio Code configuration, and a shell script for creating virtual env.
{% endhint %}


### Prepare data for annotations

1. **NRRD files**

   The easiest way to create **Mask3D** annotation in Supervisely is to use NRRD file with 3D figure that corresponds to the dimensions of the Volume.

   <img width="960" alt="NRRD" src="https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/e420a798-d376-40fc-b118-44c62615aef2">

   This NRRD example you will find in `data/mask/lung.nrrd`


2. **Data Arrays**
   
   In the process, 3D objects can be created, to create **Mask3D** annotation from which it is sufficient to provide them as a `ndarray`, where 1 pixels of the object and 0 is empty space.

   <img width="728" alt="Data Array" src="https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/d4feb39e-a837-43c9-bbac-3c6d2ef4942d">


   ```python
   figure_array = np.zeros((3, 4, 2))
   figure_array[0, 1, 0] = 1
   figure_array[0, 2, 0] = 1
   figure_array[1, 1, 0] = 1
   figure_array[1, 2, 0] = 1
   ```

3. **Images**
   
    You can also use flat mask annotations, such as black and white pictures, and create **Mask3D** from them. You just need to know which plane and slice it refers to.

    <img width="950" alt="Image" src="https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/52070b6a-9f34-46e8-94c2-6736b3a9732d">
    
    This image file example you will find in `data/mask/body.png`



    If your flat annotation doesn't correspond to the dimensions of the plane, you also need to know its `PointLocation`. This will help to properly apply the mask to the image. This point indicates where the top-left corner of the mask is located, or in other words, the coordinates of the mask's initial position on the canvas or image.
    
    ```python
    plane = 'axial'
    slice_index = 69
    point_location = [36, 91]
    ```



## Python Code

### Import libraries and init API client

```python
import os
import numpy as np
from dotenv import load_dotenv
from PIL import Image
import supervisely as sly


# To init API for communicating with Supervisely Instance. 
# It needs to load environment variables with credentials and workspace ID
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()

# Check that you did everything right - the API client initialized with the correct credentials and you defined the correct workspace ID in `local.env`file
workspace_id = sly.env.workspace_id()
workspace = api.workspace.get_info_by_id(workspace_id)
if workspace is None:
    sly.logger.warning("You should put correct WORKSPACE_ID value to local.env")
    raise ValueError(f"Workspace with id={workspace_id} not found")
```

### Create project and upload volumes

Create an empty project with the name **"Volumes Demo"** with one dataset **"CTChest"** in your workspace on the server. If a project with the same name exists in your workspace, it will be automatically renamed (Volumes Demo\_001, Volumes Demo\_002, etc.) to avoid name collisions.

```python
# create empty project and dataset on server
project_info = api.project.create(
    workspace.id,
    name="Volumes Demo",
    type=sly.ProjectType.VOLUMES,
    change_name_if_conflict=True,
)
dataset_info = api.dataset.create(project_info.id, name="CTChest")

sly.logger.info(
    f"Project with id={project_info.id} and dataset with id={dataset_info.id} have been successfully created"
)

# upload NRRD volume as ndarray into dataset
volume_info = api.volume.upload_nrrd_serie_path(
    dataset_info.id,
    name="CTChest.nrrd",
    path="data/CTChest_nrrd/CTChest.nrrd",
)
```


### Create annotations and upload into the volume


```python

mask2d_path = "data/mask/body.png"
mask3d_path = "data/mask/lung.nrrd"

# create annotation classes
lung_class = sly.ObjClass("lung", sly.Mask3D, color=[111, 107, 151])
body_class = sly.ObjClass("body", sly.Mask3D, color=[209, 192, 129])

# update project meta with new classes
api.project.append_classes(project_info.id, [lung_class, body_class])

# create Mask3D object for 'body' annotation using 2D mask (image)
mask_2d = Image.open(mask2d_path)
mask_2d = np.array(mask_2d)
# create an empty mask with the same dimensions as the volume
body_mask_3d = sly.Mask3D(np.zeros(volume_info.file_meta["sizes"], np.bool_))
# fill this mask with the an image mask for the desired plane
body_mask_3d.add_mask_2d(mask_2d, plane_name="axial", slice_index=69, origin=[36, 91])

# create Mask3D object for 'lung' annotation using NRRD file with 3D Object
lung_mask_3d = sly.Mask3D.from_file(mask3d_path)

# create VolumeObjects with Mask3D
lung = sly.VolumeObject(lung_class, mask_3d=body_mask_3d)
body = sly.VolumeObject(body_class, mask_3d=lung_mask_3d)

# create volume annotation object
volume_ann = sly.VolumeAnnotation(
    volume_info.meta,
    objects=[lung, body],
    spatial_figures=[lung.figure, body.figure],
)

# upload VolumeAnnotation
api.volume.annotation.append(volume_info.id, volume_ann)
sly.logger.info(
    f"Annotation has been sucessfully uploaded to the volume {volume_info.name} in dataset with ID={volume_info.dataset_id}"
```

In the [GitHub repository for this tutorial](https://github.com/supervisely-ecosystem/dicom-spatial-figures), you will find the [full Python script](https://github.com/supervisely-ecosystem/dicom-spatial-figures/blob/master/src/main.py).


## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone the [repository](https://github.com/supervisely-ecosystem/dicom-spatial-figures) with source code and demo data and create a [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/dicom-spatial-figures
cd dicom-spatial-figures
./create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 4.** Change ✅ workspace ID ✅ in `local.env` file by copying the ID from the context menu of the workspace. A new project with annotated videos will be created in the workspace you define:

```python
WORKSPACE_ID=696 # ⬅️ change value
```

<img src="https://user-images.githubusercontent.com/57998637/231221251-3dfc1a56-b851-4542-be5b-d82b2ef14176.gif" width="550" alt="Copy the workspace ID from the context menu"/>

**Step 5.** Start debugging `src/main.py`


![Debug tutorial in Visual Studio Code](https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/f9f9f472-3fe1-420d-a6d3-46ed6a29027f)

## To sum up

In this tutorial, we learned:

* What are the types of annotations for Volumes
* How to create a project and dataset, upload volume
* How to create 3D annotations and upload into volume
* How to configure Python development for Supervisely