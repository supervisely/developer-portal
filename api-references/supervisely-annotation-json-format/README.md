---
description: >-
  Detailed explanation of how we store annotations in JSON format for images,
  videos, point clouds, DICOMs.
---

# Supervisely Annotation JSON format

Supervisely Annotation format is designed as a universal solution for storing annotations for different data types, such as images, videos, 3D data, etc., which also supports all types of annotations, such as polygons, bitmaps, tags, etc. To maximize compatibility with other annotation formats, Supervisely can also offer a lot of conversion options from and to other formats.<br>

Key features of the Supervisely Annotation format:
- **Universal** - supports all types of annotations and data types
- **Compatible** - can be converted to and from other annotation formats with ready-to-use conversion tools
- **Lightweight** - stores only the necessary information about annotations and uses compression algorithms to reduce the size of the annotation file
- **Versatile** - can even be used locally without the Supervisely platform with a dozens of features
- **Python SDK** - all features of the format are available through the Python SDK
- **CLI tools** - Supervisely Annotation format can be used with the command line interface

In this article, we will overview the structure of the Supervisely Annotation format and explain how to use it, for more details about specific parts of the format, usage examples and code snippets, please refer to the corresponding articles.

#### Table of contents
1. [Conversion to and from Supervisely Annotation Format](conversion.md)
2. [Project Structure in Supervisely Annotation Format](project-structure.md)
3. [ProjectMeta in Supervisely Annotation Format](project-meta.md)
...

ORIGINAL TABLE, DELETE LATER >>>>
1. [Project Structure in Supervisely Annotation Format](project-structure.md)
2. [Project Classes and Tags](project-classes-and-tags.md)
3. [Tags](tags.md)
4. [Objects](objects.md)
5. [Individual Image Annotations](individual-image-annotations.md)
6. [Individual Video Annotations](individual-video-annotations.md)
7. [Point Clouds](point-clouds.md)
7. [Point Cloud Episodes](point-cloud-episodes.md)
8. [Volumes Annotations](volumes-annotation.md)

<<<< END OF ORIGINAL TABLE

#### Supervisely Annotation Format overview

Supervisely Annotation Format is a JSON file that contains all the necessary information about the project, dataset, images, annotations, and tags. You can find detailed information about each part of the format in the corresponding articles above.<br>
Here we will give a short overview of the format and its main features:

**Strictly defined data type** - only one type of data can be stored in one project, for example, images or videos, but not both. This is done to simplify the format and make it more universal.

**Project structure** - the project is divided into datasets, where each dataset contains corresponding data: images, videos, point clouds, etc. Each dataset can contain any number of items and the project can contain any number of datasets.

**Project Meta file** - the `meta.json` file contains information about object classes, tags, and other project settings. The `meta.json` file is located in the root directory of the project and is required for the correct operation of the format.

**Entities folder** - depending on the type of data stored in the project, dataset folders will contain one of folders: `img`, `video`, `pointcloud`, `volume`. Each of these folders contains all the data of the corresponding type. For example, the `img` folder contains all images for the dataset.

**Annotations folder** - the `ann` folder contains all annotations for the dataset. Each annotation is stored in a separate file with the same name as the corresponding item. For example, the annotation for the image `img_001.jpg` will be stored in the file `img_001.jpg.json`.