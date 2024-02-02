---
description: >-
  How to convert Supervisely Annotation format to and from other annotation formats.
---

# Conversion to and from Supervisely Annotation Format

Supervisely Annotation format can be easily converted to and from other annotation formats with ready-to-use conversion tools. This article will overview the conversion process, highlight the options available for each format, and explain how to use the conversion tools.

## List of Supported Formats
Both directions (to and from Supervisely Annotation format) are supported for the following formats:
- [COCO](https://cocodataset.org/#format-data)
- [Pascal VOC](http://host.robots.ox.ac.uk/pascal/VOC/voc2012/htmldoc/index.html)
- YOLO
- !!!! ADD MORE HERE !!!!

For the following formats, only conversion to Supervisely Annotation format is supported:
- [CVAT](https://opencv.github.io/cvat/docs/manual/advanced/xml_format/)
- [V7 Darwin JSON](https://docs.v7labs.com/reference/darwin-json)
- !!!! ADD MORE HERE !!!!

All these formats are supported by Supervisely Appllications, Supervisely Python SDK and Supervisely CLI.<br>
If you can't find the format you need in this list, please email us at support@supervisely.com.


## Conversion to Supervisely Annotation Format

If you have your data in some annotation format and want to use it on Supervisely platform, you will need to convert it to Supervisely Annotation format. We can offer you several options for doing this:
- With ready-to-use [Supervisely Import Applications](#supervisely-import-applications)
- With [Supervisely Python SDK](#supervisely-python-sdk-to-supervisely-annotation-format)
- With [Supervisely CLI](#supervisely-cli-to-supervisely-annotation-format)

#### Supervisely Import Applications

There are plenty of import applications in Supervisely Ecosystem. Each of them is designed to convert a specific annotation format to Supervisely Annotation format. You can find the full list of import applications in the [Supervisely Import](https://ecosystem.supervisely.com/import) section of the Ecosystem. These applications are easy to use and can be launched directly from the Ecosystem with just a few clicks.


#### Supervisely Python SDK (to Supervisely Annotation Format)

In most cases, while converting data to Supervisely Annotation format, we recommend using the automatic method, which detects the source format and selects the appropriate conversion method. However, you can also specify the source format manually.

```bash
pip install supervisely
```

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

Now you have the converted data in Supervisely Annotation format and can upload it to Supervisely platform or work with it locally.

#### Supervisely CLI (to Supervisely Annotation Format)

The conversion process is similar to the one described above. You need to specify the source and destination directories and the source format (if it cannot be detected automatically).<br>
Remember that Supervisely CLI is a command-line tool, so you need to run it from the terminal and install Supervisely Python SDK first.

```bash
pip install supervisely
```

```bash
# This is how we converting our data from some format to Supervisely Annotation format
# without specifying the source format. It will be detected automatically.
supervisely convert_from /path/to/source/dir --dst_dir /path/to/destination/dir
# The dst_dir argument is optional. If it is not specified, the converted data will be saved to the
# in to_sly subdirectory of the working directory (where the command was launched from).

# If we know the format and want to specify it manually, we can do it like this:
supervisely convert_from --format coco /path/to/source/dir
# In this case, the converted data will be saved to the in to_sly subdirectory of the working directory
# and converter will assume that the source data is in COCO format.
```

Now in your destination directory (or in the to_sly subdirectory of the working directory) you have the converted data in Supervisely Annotation format and can use it for your tasks.

## Conversion to Supervisely Annotation Format

When you need to convert your data from Supervisely Annotation format to some other format, you can use the same options as described above. The only difference is that you need to use the appropriate conversion method (to instead of from) and specify the destination format. So, the options are:
- With ready-to-use [Supervisely Export Applications](#supervisely-export-applications)
- With [Supervisely Python SDK](#supervisely-python-sdk-from-supervisely-annotation-format)
- With [Supervisely CLI](#supervisely-cli-from-supervisely-annotation-format)

#### Supervisely Export Applications

As for the import applications, there are plenty of export applications in Supervisely Ecosystem. Each of them is designed to convert Supervisely Annotation format to a specific annotation format. You can find the full list of export applications in the [Supervisely Export](https://ecosystem.supervisely.com/export) section of the Ecosystem. These applications are easy to use and don't require any special knowledge.

#### Supervisely Python SDK (from Supervisely Annotation Format)

When converting from Supervisely Annotation format, you must specify the destination format.

```python
import supervisely as sly

# Assuming the source dataset is located in the following directory:
source_dataset_path = "/path/to/source/dataset"

# Converted dataset will be saved to the following directory:
converted_dataset_path = "/path/to/converted/dataset"

sly.convert.To.coco(source_dataset_path, converted_dataset_path)
sly.convert.To.pascal_voc(source_dataset_path, converted_dataset_path)
```

Now you have the converted data in the desired format and can use it for your tasks.

#### Supervisely CLI (from Supervisely Annotation Format)

The conversion process is similar to the one described above. You need to specify the source and destination directories and the destination format.<br>
Remember that Supervisely CLI is a command-line tool, so you need to run it from the terminal and install Supervisely Python SDK first.

```bash
pip install supervisely
```

```bash
# This is how we converting our data from Supervisely Annotation format to other formats.
supervisely convert_to /path/to/source/dir --dst_dir /path/to/destination/dir --format coco

# The dst_dir argument is optional. If it is not specified, the converted data will be saved to the
# in from_sly subdirectory of the working directory (where the command was launched from).
supervisely convert_to /path/to/source/dir --format pascal_voc
# But the --format argument is required. If it is not specified, the conversion will fail.
```

Now in your destination directory (or in the from_sly subdirectory of the working directory) you have the converted data in the desired format and can use it for your tasks.