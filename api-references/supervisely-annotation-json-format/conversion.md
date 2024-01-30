---
description: >-
  How to convert Supervisely Annotation format to and from other annotation formats.
---

# Conversion to and from Supervisely Annotation Format

Supervisely Annotation format can be easily converted to and from other annotation formats with ready-to-use conversion tools. This article will overview the conversion process, highlight the options available for each format, and explain how to use the conversion tools.


## Conversion to Supervisely Annotation Format

#### Supervisely Import applications

There are plenty of import applications in Supervisely Ecosystem. Each of them is designed to convert a specific annotation format to Supervisely Annotation format. You can find the full list of import applications in the [Supervisely Import](https://ecosystem.supervisely.com/import) section of the Ecosystem. These applications are easy to use and can be launched directly from the Ecosystem with just a few clicks.


#### Python SDK

Supervisely Python SDK supports converting in both directions (to and from Supervisely Annotation format) the following annotation formats:

- COCO
- Pascal VOC
- YOLO
- !!!! ADD MORE HERE !!!!

For the following formats, only conversion to Supervisely Annotation format is supported:

- CVAT
- Darwin JSON (V7)
- !!!! ADD MORE HERE !!!!

In most cases, while converting data to Supervisely Annotation format, we recommend using the automatic method, which detects the source format and selects the appropriate conversion method. However, you can also specify the source format manually.

```python

import supervisely as sly

# Assuming the source dataset is located in the following directory:
source_dataset_path = "/path/to/source/dataset"

# Converted dataset will be saved to the following directory:
converted_dataset_path = "/path/to/converted/dataset"

# Automatic conversion.
sly_project = sly.convert.From.auto(source_dataset_path, converted_dataset_path)

# Manual conversion with specified source format.
sly_project = sly.convert.From.cvat(source_dataset_path, converted_dataset_path)
sly_project = sly.convert.From.coco(source_dataset_path, converted_dataset_path)

# The conversion will return a Supervisely Project object, which can be used to access the converted data.
# And for an immidiate upload to Supervisely platform with one line of code:
sly_project: sly.Project
sly_project.upload()

```

#### Supervisely CLI

Supervisely CLI supports converting all formats that are supported by Supervisely Python SDK. The conversion process is similar to the one described above. You need to specify the source and destination directories and the source format (if it cannot be detected automatically).<br>
Remember that Supervisely CLI is a command-line tool, so you need to run it from the terminal and install Supervisely Python SDK first.

```bash
pip install supervisely
```

```bash
# This is how converting our data from some format to Supervisely Annotation format
# without specifying the source format. It will be detected automatically.
supervisely convert_from /path/to/source/dir --dst_dir /path/to/destination/dir
# The dst_dir argument is optional. If it is not specified, the converted data will be saved to the
# in to_sly subdirectory of the working directory (where the command was launched from).

# If we know the format and want to specify it manually, we can do it like this:
supervisely convert_from --format coco /path/to/source/dir
# In this case, the converted data will be saved to the in to_sly subdirectory of the working directory
# and converter will assume that the source data is in COCO format.
```