---
description: How to create Mask3D annotations on volumes in Python
---

# 3D annotations on volumes

## Introduction

In this tutorial, you will learn how to programmatically create 3D annotations for volumes and upload them to Supervisely platform. 

Supervisely supports different types of shapes/geometries for volume annotation, and now we will consider the primary type - **Mask3D**.

You can explore other types as well, like Mask (also known as Bitmap), Bounding Box (Rectangle), and Polygon. However, you'll find more information about them in other articles.

Learn more about [Supervisely Annotation in JSON format](https://developer.supervisely.com/getting-started/supervisely-annotation-format) for volumes.

Read about our enterprise-grade DICOM labeling toolbox in blog post [Best DICOM & NIfTI annotation tools for Medical Imaging AI](https://supervisely.com/blog/dicom-labeling-toolbox/) to be informed about all the advantages of our platform.

![Labeling toolbox](https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/bf904c67-f7e8-4d5e-a10d-0eaae8ff7c28)

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/dicom-spatial-figures): source code, Visual Studio Code configuration, and a shell script for creating virtual env.
{% endhint %}


### Prepare data for annotations

During your work, you can create 3D annotation shapes, and here are a few ways you can do that:

1. **NRRD files**

    The easiest way to create **Mask3D** annotation in Supervisely is to use NRRD file with 3D figure that corresponds to the dimensions of the Volume.

    <img width="960" alt="NRRD" src="https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/e420a798-d376-40fc-b118-44c62615aef2">

    You can find an example NRRD file at [data/mask/lung.nrrd](https://github.com/supervisely-ecosystem/dicom-spatial-figures/tree/master/data/mask) in the GitHub repository for this tutorial.


2. **NumPy Arrays**
   
    Another simple way to create **Mask3D** annotation is to use NumPy arrays, where values of 1 represent the object and values of 0 represent empty space.

    <img width="728" alt="NumPy Array" src="https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/0f842a17-ddb1-4fb1-891d-ba4aa2ee4796">

    On the right side, you can see a volume with a pink cuboid. Let's represent this volume as an NumPy array.

    ```python   
    figure_array = np.zeros((3, 4, 2))
     ```

    To draw a pink cuboid on it, you need to assign a value of 1 to the necessary cells. In the code below, each cell is indicated by three axes [`axis_0`, `axis_1`, `axis_2`].

    ```python
    figure_array[0, 1, 0] = 1 
    figure_array[0, 2, 0] = 1
    figure_array[1, 1, 0] = 1
    figure_array[1, 2, 0] = 1
    ```
    In the Python code example section, we will create a NumPy array that represents a foreign body in the lung as a sphere. 

3. **Images**
   
    You can also use flat mask annotations, such as black and white pictures, to create **Mask3D** from them. You just need to know which plane and slice it refers to.

    <img width="950" alt="Image" src="https://github.com/supervisely-ecosystem/dicom-spatial-figures/assets/57998637/52070b6a-9f34-46e8-94c2-6736b3a9732d">
    
    You can find an example image file at [data/mask/body.png](https://github.com/supervisely-ecosystem/dicom-spatial-figures/tree/master/data/mask) in the GitHub repository for this tutorial.

    If your flat annotation doesn't correspond to the dimensions of the plane, you also need to know its `PointLocation`. This will help to properly apply the mask to the image. This point indicates where the top-left corner of the mask is located, or in other words, the coordinates of the mask's initial position on the canvas or image.
    
    ```python
    plane = 'axial'
    slice_index = 69
    point_location = [36, 91]
    ```



## Python code example

### Import libraries and init API client

```python
import os
import numpy as np
import cv2
from dotenv import load_dotenv
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

# create annotation classes
lung_class = sly.ObjClass("lung", sly.Mask3D, color=[111, 107, 151])
body_class = sly.ObjClass("body", sly.Mask3D, color=[209, 192, 129])
tumor_class = sly.ObjClass("tumor", sly.Mask3D, color=[255, 153, 204])

# update project meta with new classes
api.project.append_classes(project_info.id, [lung_class, tumor_class, body_class])

################################  1  NRRD file    ######################################

mask3d_path = "data/mask/lung.nrrd"

# create 3D Mask annotation for 'lung' using NRRD file with 3D object
lung_mask = sly.Mask3D.create_from_file(mask3d_path)
lung = sly.VolumeObject(lung_class, mask_3d=lung_mask)

###############################  2  NumPy array    #####################################

# create 3D Mask annotation for 'tumor' using NumPy array
tumor_mask = sly.Mask3D(generate_tumor_array())
tumor = sly.VolumeObject(tumor_class, mask_3d=tumor_mask)

##################################  3  Image    ########################################

image_path = "data/mask/body.png"

# create 3D Mask annotation for 'body' using image file
mask = cv2.imread(image_path, cv2.IMREAD_GRAYSCALE)
# create an empty mask with the same dimensions as the volume
body_mask = sly.Mask3D(np.zeros(volume_info.file_meta["sizes"], np.bool_))
# fill this mask with the an image mask for the desired plane. 
# to avoid errors, use constants: Plane.AXIAL, Plane.CORONAL, Plane.SAGITTAL
body_mask.add_mask_2d(mask, plane_name=sly.Plane.AXIAL, slice_index=69, origin=[36, 91])
body = sly.VolumeObject(body_class, mask_3d=body_mask)

# create volume annotation object
volume_ann = sly.VolumeAnnotation(
    volume_info.meta,
    objects=[lung, tumor, body],
    spatial_figures=[lung.figure, tumor.figure, body.figure],
)

# upload VolumeAnnotation
api.volume.annotation.append(volume_info.id, volume_ann)
sly.logger.info(
    f"Annotation has been sucessfully uploaded to the volume {volume_info.name} in dataset with ID={volume_info.dataset_id}"
)
```

**Auxiliary function for generating tumor NumPy array:**

```python

def generate_tumor_array():
    """
    Generate a NumPy array representing the tumor as a sphere
    """
    width, height, depth = (512, 512, 139)  # volume shape
    center = np.array([128, 242, 69])  # sphere center in the volume
    radius = 25
    x, y, z = np.ogrid[:width, :height, :depth]
    # Calculate the squared distances from each point to the center
    squared_distances = (x - center[0]) ** 2 + (y - center[1]) ** 2 + (z - center[2]) ** 2
    # Create a boolean mask by checking if squared distances are less than or equal to the square of the radius
    tumor_array = squared_distances <= radius**2
    tumor_array = tumor_array.astype(np.uint8)
    return tumor_array

```

### Download existing annotations, manipulate the geometry & upload the result

```python
volume_id = os.getenv("VOLUME_ID")
project_id = sly.env.project_id()
project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))
key_id_map = sly.KeyIdMap()

# * Download the annotation

ann_json = api.volume.annotation.download(volume_id)
ann = sly.VolumeAnnotation.from_json(ann_json, project_meta, key_id_map)

# load spatial geometries
for figure in ann.spatial_figures:
    api.volume.figure.load_sf_geometry(figure, key_id_map)

# * Manipulate the geometry

new_sfs = []
for figure in ann.spatial_figures:
    # In this example, we will invert the mask of each spatial figure,
    # but you can perform any manipulation you need.
    inverted_mask_array = np.invert(figure.geometry.data)

    # create a new object with the inverted mask
    new_geometry: sly.Mask3D = figure.geometry.clone()
    new_geometry.data = inverted_mask_array

    # add the new figure to the list of spatial figures
    new_figure = sly.VolumeFigure.clone(figure, geometry=new_geometry)
    new_sfs.append(new_figure)

# clone the annotation with the new spatial figures
new_ann = sly.VolumeAnnotation.clone(ann, spatial_figures=new_sfs)

# * Upload the new annotation
api.volume.annotation.append(volume_id, new_ann, key_id_map)
```

### Convert Mask3D geometries into meshes

Spatial figures can be easily converted into meshes:

```python
ann_json = api.volume.annotation.download(volume_id)
ann = sly.VolumeAnnotation.from_json(ann_json, project_meta, key_id_map)

for figure in ann.spatial_figures:
    # load the spatial geometry first, if not already loaded
    api.volume.figure.load_sf_geometry(figure, key_id_map)

    # Option 1: python Trimesh object
    mesh = sly.volume.volume.convert_3d_geometry_to_mesh(figure.geometry)

    # Option 2: export to STL/OBJ file
    out_path = figure.geometry.sly_id + ".stl"  # or ".obj"

    # two latter arguments are optional and passed to the convert_3d_geometry_to_mesh function
    sly.volume.volume.export_3d_as_mesh(
        figure.geometry, out_path, apply_decimation=True, decimation_fraction=0.4
    )
```

In the [GitHub repository for this tutorial](https://github.com/supervisely-ecosystem/dicom-spatial-figures), you will find the [full Python script](https://github.com/supervisely-ecosystem/dicom-spatial-figures/blob/master/src/main.py).


## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](https://developer.supervisely.com/getting-started/basics-of-authentication)

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
* How to download and manipulate spatial geometries