---
description: How to create Mask 3D and Mask 2D (Bitmap) on volumes in Python
---

# Spatial labels on volumes

## Introduction

In this tutorial, you will learn how to programmatically create classes, objects, figures, its geometries for volumes and upload them to Supervisely platform. 

Supervisely supports different types of shapes/geometries for volume annotation:

* Mask 3D
* Mask (also known as Bitmap)
* Bounding Box (rectangle)* - will be covered in other tutorials
* Polygon - will be covered in other tutorials* 

Learn more about [Supervisely Annotation in JSON format](https://developer.supervisely.com/api-references/supervisely-annotation-json-format/volumes-annotation) for volumes.

You can also use ready-made solutions for labeling. Read about how it works in our blog article [Best DICOM & NIfTI annotation tools for Medical Imaging AI](https://supervisely.com/blog/dicom-labeling-toolbox/) and take full advantage of our platform.

![Mask3D and Masks](https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/384c7ffc-5eb5-4920-8f02-022e59a136c8)

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/dicom-spatial-figures): source code, Visual Studio Code configuration, and a shell script for creating virtual env.
{% endhint %}

## How to debug this tutorial

**Step 1.** Prepare  `~/supervisely.env` file with credentials. [Learn more here.](../basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone the [repository](https://github.com/supervisely-ecosystem/dicom-spatial-figures) with source code and demo data and create a [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/dicom-spatial-figures
cd dicom-spatial-figures
./create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.&#x20;

```bash
code -r .
```

**Step 4.** Change ✅ workspace ID ✅ in `local.env` file by copying the ID from the context menu of the workspace. A new project with annotated videos will be created in the workspace you define:

```python
WORKSPACE_ID=696 # ⬅️ change value
```

![Copy the workspace ID from the context menu](https://user-images.githubusercontent.com/57998637/231221251-3dfc1a56-b851-4542-be5b-d82b2ef14176.gif)

**Step 5.** Start debugging `src/main.py`&#x20;


![Debug tutorial in Visual Studio Code](https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/00e3bad9-ed37-42ea-86f0-43eab647f994)

## Python Code

### &#x20;Import libraries

```python
import os

from dotenv import load_dotenv

import supervisely as sly
import numpy as np
```

### &#x20;Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials and workspace ID:

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

With the next lines, we will check that you did everything right - the API client initialized with the correct credentials and you defined the correct workspace ID in `local.env`.

```python
workspace_id = sly.env.workspace_id()
workspace = api.workspace.get_info_by_id(workspace_id)
if workspace is None:
    sly.logger.warning("You should put correct WORKSPACE_ID value to local.env")
    raise ValueError(f"Workspace with id={workspace_id} not found")
```

### Create project

Create an empty project with the name **"DICOM Volumes Demo"** with one dataset **"Lungs"** in your workspace on the server. If a project with the same name exists in your dataset, it will be automatically renamed (DICOM Volumes Demo\_001, DICOM Volumes Demo\_002, etc.) to avoid name collisions.&#x20;

```python
project = api.project.create(
    workspace.id,
    name="Volumes Demo",
    type=sly.ProjectType.VOLUMES,
    change_name_if_conflict=True,
)
dataset_dcm = api.dataset.create(project.id, name="CTChest_dcm")
dataset_nrrd = api.dataset.create(project.id, name="CTChest_nrrd")
sly.logger.info(f"Project has been sucessfully created, id={project.id}")
```

### Upload DICOM volume to the dataset on the server

Volumes can be imported from two sources:
 - set of `.dcm` files 
 - `.nrrd` file


Name for volume and example data dirs

```python
name = "CTChest.nrrd"
dicom_dir_name = "data/CTChest_dcm"
nrrd_dir_name = "data/CTChest_nrrd"
```

#### OPTION 1
Upload DICOM series to the dataset on server

```python
series_infos = sly.volume.inspect_dicom_series(root_dir=dicom_dir_name)

for _, files in series_infos.items():
    dicom_volume_np, dicom_volume_meta = sly.volume.read_dicom_serie_volume_np(
        files, anonymize=True
    )
    progress_cb = sly.tqdm_sly(
        desc=f"Upload DICOM volume as {name}", total=sum(dicom_volume_np.shape)
    ).update
    dicom_info = api.volume.upload_np(
        dataset_dcm.id, name, dicom_volume_np, dicom_volume_meta, progress_cb
    )
    sly.logger.info(
        f"DICOM volume has been uploaded to Supervisely with ID: {dicom_info.id}"
    )
```

#### OPTION 2
Upload NRRD volume as nparray

```python
nrrd_path = os.path.join(nrrd_dir_name, name)
nrrd_volume_np, nrrd_volume_meta = sly.volume.read_nrrd_serie_volume_np(nrrd_path)

progress_cb = sly.tqdm_sly(
    desc=f"Upload NRRD volume {name}", total=sum(nrrd_volume_np.shape)
).update
nrrd_info = api.volume.upload_np(
    dataset_id=dataset_nrrd.id,
    name=name,
    np_data=nrrd_volume_np,
    meta=nrrd_volume_meta,
    progress_cb=progress_cb,
)
sly.logger.info(f"NRRD volume has been uploaded to Supervisely with ID: {nrrd_info.id}")
```

Use `nrrd_info` instead of `dicom_info` further down the tutorial if needed

### Create annotation classes and update project meta

In this tutorial, classes are created without the `color` argument, the color will be generated automatically.
To set the color for a class, you need to add the `color` argument.

```python
lung_obj_class = sly.ObjClass("lung", sly.Mask3D)
body_obj_class = sly.ObjClass("body", sly.Bitmap)
```

The next step is to create ProjectMeta - a collection of annotation classes and tags that will be available for labeling in the project.

```python
project_meta = sly.ProjectMeta(obj_classes=[lung_obj_class, body_obj_class])
```

Finally, we need to send all this data to our project to the server:

```python
api.project.update_meta(project.id, project_meta.to_json())
```

### Prepare source data for Mask (Bitmap)

The mask data can be represented as:
 - `encoded string` 
 - `ndarray`

```python
mask_dir_name = "data/mask"
```

#### OPTION 1

Load raw geometry for 'body' as base64 `encoded string` (from `ndarray`) and convert back to `ndarray`

Full raw data for the Mask (Bitmap) can be found in `main.py`

```python
raw_bitmap_data = "eJwBDgjx94lQTkcNChoKAAAADUlIRFIAAAGgAAABRgEDAAAAFu..."
bitmap_data_decoded = sly.Bitmap.base64_2_data(raw_bitmap_data)
```

#### OPTION 2

Load `ndarray` from `.npy` file

```python
body_path = os.path.join(mask_dir_name, "body.npy")
bitmap_data = np.load(body_path)
```

Example of `ndarray`:

```python
bitmap_data = np.ndarray((4, 4), dtype='bool')
# Output bitmap_data:
# [[False False False False]
#  [False False False False]
#  [False False False False]
#  [False False False False]]
```

### Create a volume object for Mask named 'body'

The object must be bound to the corresponding class

```python
body = sly.VolumeObject(body_obj_class)
```

### Create Mask figure

Before creating the geometry, we need to convert the raw data into an `ndarray`. For this purpose will be used method `base64_2_data`.

After that, define the `PointLocation` to properly apply the mask to the image. This point indicates where the top-left corner of the mask is located, or in other words, the coordinates of the mask's initial position on the canvas or image.

⚠️  `PointLocation` uses only if the mask size is smaller than the image size othervise leave as `None`.

Once the geometry is created, it's time to prepare the `VolumeFigure`. To do this, we use a `VolumeObject` named **"body"**, and geometry and select the plane (`"axial"`) where the figure will be placed, the slice number which represents a certain image on this plane.

```python
bitmap_data = sly.Bitmap.base64_2_data(raw_bitmap_data)
bitmap_origin = sly.PointLocation(91, 36)
bitmap_geometry = sly.Bitmap(bitmap_data, bitmap_origin)
bitmap_figure = sly.VolumeFigure(body, bitmap_geometry, "axial", 69)
```

### Create `Slice` and `Plane` objects

```python
volume_slice = sly.Slice(69, figures=[bitmap_figure])
plane = sly.Plane(
    plane_name="axial", 
    items=[volume_slice], 
    volume_meta=dicom_info.meta
)
```

### Prepare source data for Mask 3D

For this type of shape, we will use the prepared `.nrrd` file to load geometry as bytes

```python
mask_dir_name = "data/mask"
mask_path = os.path.join(mask_dir_name, "lung.nrrd")
with open(mask_path, "rb") as file:
    geometry_bytes = file.read()
```

### Create a volume object for Mask 3D named 'lung'

```python
lung = sly.VolumeObject(lung_obj_class)
```

### Create an empty Mask 3D figure

This empty spatial figure will be uploaded as a dummy for future geometry loading and thus conversion to a normal spatial figure. 

```python
empty_figure = sly.VolumeFigure(
    lung, 
    sly.Mask3D(np.zeros((3, 3, 3), dtype=np.bool_)), 
    None, 
    None
)
```

### Create `VolumeObjectCollection`

This will allow us to add all the volume objects at once.

```python
objects = sly.VolumeObjectCollection([lung, body])
```

### Create volume annotation

For more information about the VolumeAnnotation in JSON format, please use the link at the beginning of this article

{% hint style="info" %}
💡**2D figures** will be added as common figures with planes, a **3D figures**  - always are spatial figures!
{% endhint %}

```python
volume_ann = sly.VolumeAnnotation(
    dicom_info.meta,
    objects,
    plane_axial=plane,
    spatial_figures=[empty_figure],
)
```

### Upload annotation to the volume on the server

```python
key_id_map = sly.KeyIdMap()
api.volume.annotation.append(
    dicom_info.id, 
    volume_ann, 
    key_id_map
)
sly.logger.info(
    f"Annotation has been sucessfully uploaded to the volume {dicom_info.name} in dataset with ID={dicom_info.dataset_id}"
)
```

### Upload geometry to Mask 3D spatial figure

```python
api.volume.figure.upload_sf_geometry(
    volume_ann.spatial_figures, 
    [geometry_bytes], 
    key_id_map
)
sp_figure_id = key_id_map.get_figure_id(volume_ann.spatial_figures[0].key())
sly.logger.info(
    f"Geometry has been sucessfully uploaded to the spatial figure with ID={sp_figure_id}"
)
```

In the [GitHub repository for this tutorial](https://github.com/supervisely-ecosystem/dicom-spatial-figures), you will find the [full Python script](https://github.com/supervisely-ecosystem/dicom-spatial-figures/blob/master/src/main.py).

## Recap

In this tutorial, we learned how to&#x20;

* Quickly configure Python development for Supervisely
* Create a project and dataset with classes of different shapes
* Initialize Mask, Mask 3D for volumes
* Construct Supervisely annotation and upload it with a volume to the server