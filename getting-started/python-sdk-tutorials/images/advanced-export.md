# Advanced Tutorial: Downloading Images and Annotations with Supervisely

This advanced tutorial will guide you through various methods of downloading images and annotations from Supervisely. We'll cover everything from basic downloads to optimized approaches for large projects, with performance benchmarks to illustrate the benefits of different techniques.

## Basic Downloads

### Project Metadata

{% hint style="info" %}

The easiest way is to create a `.env` file that stores your **SERVER_ADDRESS** and **API_TOKEN**. This makes it simpler to initialize the `api` client as you work through code snippets in this tutorial. You can learn more about this in this [section](../../basics-of-authentication.md)

{% endhint %}

Project metadata contains essential information about your project, including classes, tags, and other configurations.

```python
import supervisely as sly

# Initialize API client
api = sly.Api.from_env()

# Get project metadata with project settings by ID
project_id = 12345
project_meta_json = api.project.get_meta(project_id, with_settings=True)
project_meta = sly.ProjectMeta.from_json(project_meta_json)

# Display project classes and tags
print("Project Classes:")
for obj_class in project_meta.obj_classes:
    print(f"- {obj_class.name} ({obj_class.geometry_type.name()})")

print("\nProject Tags:")
for tag_meta in project_meta.tag_metas:
    print(f"- {tag_meta.name}")
```

### Single Image and Annotation

Here's how to download a single image and its annotation:

```python
import os
import supervisely as sly

api = sly.Api.from_env()

# Define project and dataset IDs
project_id = 12345
dataset_id = 67890

# Get first image info
image_infos = api.image.get_list(dataset_id)
image_info = image_infos[0]
image_id = image_info.id

# Download numpy array
img_np = api.image.download_np(image_id)
print(f"Downloaded image numpy array: {image_info.name}, shape: {img_np.shape}")

# or download image and save locally if needed
save_dir = "downloaded_data"
sly.fs.mkdir(save_dir)
save_path = os.path.join(save_dir, image_info.name)
api.image.download(image_id, save_path)
print(f"Downloaded image file: {image_info.name}, path: {save_path}")

# Download annotation in JSON format and save
ann_json = api.annotation.download_json(image_id)
save_path = os.path.join(save_dir, image_info.name + ".json")
sly.json.dump_json_file(ann_json, save_path)
print(f"Downloaded annotation file: {save_path}")

# or convert to Annotation object
ann = sly.Annotation.from_json(ann_json, project_meta)
print(f"Downloaded annotation object with {len(ann.labels)} labels")
```

### Annotation JSON Format

Supervisely allows you to download annotations in JSON format, which is particularly useful for custom processing or integration with other tools.

1. **Flexibility**: JSON format provides the raw data structure, allowing you to parse and process it according to your specific needs.
2. **Completeness**: JSON format includes all metadata and additional information that might be stripped in specific export formats.
3. **Interoperability**: JSON is a universal format that can be easily converted to other formats or used directly in various applications.

To learn more about Supervisely image annotation format, read the [Image Annotation](../../supervisely-annotation-format/images.md) article.

## Batch Downloads

### Multiple Images and Annotations

For better performance, download multiple images and annotations in batches.
Almost all our methods that download multiple images or annotations at once use batches at a low level.
The batch size is optimized for efficient operation across different instances and is set to 50.

```python
import supervisely as sly
from tqdm import tqdm

api = sly.Api.from_env()

project_id = 12345
dataset_id = 67890

# Get image IDs from dataset
image_infos = api.image.get_list(dataset_id)
image_ids = [image_info.id for image_info in image_infos[:100]]  # First 100 images

# Download images. This method is optimized and will download all images batch by batch.
images_progress = tqdm(total=len(image_ids), desc="Downloading images")
images = api.image.download_nps(dataset_id, image_ids, progress_cb=images_progress)

# Download annotations for all images,
annotation_progress = tqdm(total=len(image_ids), desc="Downloading annotations")
batch_anns = api.annotation.download_batch(dataset_id, image_ids, progress_cb=annotation_progress)
```

### Setting Batch Size

Batch size significantly affects download performance. Here's how to set it and understand its impact:

```python
import re
import time
from typing import List, Optional

from requests_toolbelt import MultipartDecoder
from tqdm import tqdm

import supervisely as sly
from supervisely.api.annotation_api import ApiField
from supervisely.imaging import image

api = sly.Api.from_env()

dataset_id = 67890

image_infos = api.image.get_list(dataset_id)
image_ids = [image_info.id for image_info in image_infos[:1000]]  # Test with 1000 images

# Define a function to download images using the API
def download_batch(dataset_id: int, image_ids: List[int], progress_cb: Optional[tqdm] = None):
    response = api.post(
        "images.bulk.download",
        {ApiField.DATASET_ID: dataset_id, ApiField.IMAGE_IDS: image_ids},
    )
    decoder = MultipartDecoder.from_response(response)
    for part in decoder.parts:
        content_utf8 = part.headers[b"Content-Disposition"].decode("utf-8")
        img_id = int(re.findall(r'(^|[\s;])name="(\d*)"', content_utf8)[0][1])

        if progress_cb is not None:
            progress_cb(1)
        bytes = part.content
        yield img_id, image.read_bytes(bytes)

# Test different batch sizes
batch_sizes = [10, 50, 100, 200]

for batch_size in batch_sizes:
    progress_cb = tqdm(total=len(image_ids), desc=f"Download images: {batch_size}")
    start_time = time.monotonic()
    for batch_ids in sly.batched(image_ids, batch_size=batch_size):
        for img_id, batch_anns in download_batch(dataset_id, batch_ids, progress_cb):
            pass
    elapsed_time = time.monotonic() - start_time
    print(f"Batch size: {batch_size}, Time: {elapsed_time:.2f} seconds")
```

Following results was obtained on [Pascal VOC 2012](https://datasetninja.com/pascal-voc-2012) dataset which you could download from [datasetninja.com](https://datasetninja.com/)

| Batch size | Time (seconds) |
| ---------- | -------------- |
| 10         | 210            |
| 50         | 44             |
| 100        | 33             |
| 200        | 31             |

Batch size affects:

-   Network efficiency: Larger batches reduce overhead from multiple requests
-   Memory usage: Very large batches consume more RAM
-   Error handling: Smaller batches are easier to retry if errors occur

The optimal batch size depends on your network conditions, server load, and image sizes. Generally, batch sizes between 50-100 work well for most cases.

## Complete Project Downloads

### Downloading an Entire Project

To download a complete project, you can use the convenient `download_fast` function that handles all the details for you.

This function provides significant advantages over manual download approaches:

-   Downloads the complete structure with all metadata
-   Preserves the Supervisely format for easy re-import later
-   Automatically handles batching and resource management
-   Provides options for customizing exactly what gets downloaded
-   Uses a smart approach to choose between asynchronous downloading or standard method

```python
import supervisely as sly

api = sly.Api.from_env()

project_id = 12345
save_path = 'Pascal_VOC_2012'
sly.fs.mkdir(save_path)
sly.download_fast(
    api=api,
    project_id=project_id,
    dest_dir=save_path,
)
```

Function Signature: `download_fast`

| Parameter             | Type                 | Default                | Description                                                                                            |
| --------------------- | -------------------- | ---------------------- | ------------------------------------------------------------------------------------------------------ |
| `api`                 | `Api`                | Required               | Supervisely API address and token                                                                      |
| `project_id`          | `int`                | Required               | Project ID to download                                                                                 |
| `dest_dir`            | `str`                | Required               | Destination path to local directory                                                                    |
| `dataset_ids`         | `List[int]`          | `None`                 | Specified list of Dataset IDs which will be downloaded.                                                |
| `log_progress`        | `bool`               | `True`                 | Show downloading logs in the output                                                                    |
| `cache`               | `FileCache`          | `None`                 | Cache of downloading files                                                                             |
| `progress_cb`         | `tqdm` or `Callable` | `None`                 | Function for tracking download progress                                                                |
| `only_image_tags`     | `bool`               | `False`                | Specify if downloading images only with image tags. Alternatively, full annotations will be downloaded |
| `save_image_info`     | `bool`               | `False`                | Include image info in the download                                                                     |
| `save_images`         | `bool`               | `True`                 | Include images in the download                                                                         |
| `save_image_meta`     | `bool`               | `False`                | Include images metadata in JSON format in the download                                                 |
| `images_ids`          | `List[int]`          | `None`                 | Specified list of Image IDs which will be downloaded                                                   |
| `resume_download`     | `bool`               | `False`                | Resume download enables to download only missing files avoiding erase of existing files                |
| `batch_size`          | `int`                | `50` sync, `100` async | Size of a downloading batch                                                                            |
| `download_blob_files` | `bool`               | `False`                | Flag indicating whether to download the project in Blob format or as a regular project                 |

### Downloading in Specific Formats

When downloading data from Supervisely, it is initially exported in the native Supervisely format. For projects with thousands of small images, Supervisely offers an optimized approach using "blob files".

However, you can easily convert data from the classic Supervisely format to other popular formats immediately after downloading. The SDK provides built-in conversion utilities that make it simple to transform your data into formats like COCO, YOLO, Pascal VOC, and more.

#### Extended Supervisely Format with Blobs

This download method is only available for projects that were originally uploaded using the blob format.

```python
import supervisely as sly

api = sly.Api.from_env()

project_id = 12345
output_dir = "blob_project"
sly.fs.mkdir(output_dir)

# Download project with blob files (much faster for projects with many small images)
sly.download_fast(
    api=api,
    project_id=project_id,
    dest_dir=output_dir,
    download_blob_files=True  # Important for blob images
)
```

The blob approach packages many small images into a single archive file, reducing filesystem operations and network requests. This can be up to `4-22x` faster than standard downloads for projects with thousands of small images.

For more detailed information about working with blob files, including how to upload and process blob-based projects, please refer to [documentation on working with blob files](./optimized-import).

You can also use the application from the ecosystem that will download a project of this format: [Export to Supervisely format: Blob](https://app.supervisely.com/ecosystem/apps/supervisely-ecosystem/export-to-supervisely-format-blob)

#### Popular formats like COCO, YOLO, Pascal VOC etc.

After downloading classic Supervisely format, you can convert the data to popular formats like COCO, YOLO, or Pascal VOC:

```python
import supervisely as sly

api = sly.Api.from_env()

project_id = 12345

# Define download path
output_dir = "downloaded_project"
sly.fs.mkdir(output_dir)

# Download the project in Supervisely format
print("Downloading project in Supervisely format...")
sly.download_fast(
    api=api,
    project_id=project_id,
    dest_dir=output_dir,
)
print(f"Project saved to: {output_dir}")

# After downloading, you can open the project to access its contents
project_fs = sly.Project(output_dir, sly.OpenMode.READ)
```

Then you can convert the project to other formats in two ways:

-   Using the sly.convert functions
-   Using the Project object

{% tabs %}

{% tab title="COCO" %}

COCO format supports geometry types like rectangles, bitmaps, polygons, and graph nodes

```python
# 1. Convert to COCO format
print("\nConverting to COCO format...")
coco_output_dir = "coco_format"

sly.convert.project_to_coco(output_dir, coco_output_dir)
# or
project_fs.to_coco(coco_output_dir)
print(f"COCO format saved to: {coco_output_dir}")
```

{% endtab %}

{% tab title="YOLO" %}

YOLO format supports:

-   Detection task: rectangles, bitmaps, polygons, graph nodes, polylines, alpha masks
-   Segmentation task: polygons, bitmaps, alpha masks - Pose estimation task: graph nodes

```python
# 2. Convert to YOLO format for detection
print("\nConverting to YOLO format for detection...")
yolo_output_dir = "yolo_format"

sly.convert.project_to_yolo(output_dir, yolo_output_dir, task_type="detect")
# or
project_fs.to_yolo(yolo_output_dir, task_type="detect")

print(f"YOLO format saved to: {yolo_output_dir}")
```

{% endtab %}

{% tab title="Pascal VOC" %}

Pascal VOC format supports standard Pascal VOC annotation structure

```python
# 3. Convert to Pascal VOC format
print("\nConverting to Pascal VOC format...")
pascal_output_dir = "pascal_voc_format"

sly.convert.project_to_pascal_voc(output_dir, pascal_output_dir)
# or
project_fs.to_pascal_voc(pascal_output_dir)
print(f"Pascal VOC format saved to: {pascal_output_dir}")
```

{% endtab %}

{% endtabs %}

You can also convert specific datasets

```python
# For example, to convert a specific dataset to Pascal VOC format:
for ds in project_fs.datasets:
    sly.convert.dataset_to_pascal_voc(dataset=ds, meta=project_fs.meta, dest_dir=pascal_output_dir)
    # or
    ds.to_pascal_voc(project_fs.meta, dest_dir=pascal_output_dir)
```

These conversion utilities make it easy to use your Supervisely data with other frameworks and tools without needing to implement custom converters.

## Working with Datasets

### Dataset Hierarchy

Supervisely supports hierarchical dataset structures.
See the special article that explains how to work with projects that have hierarchical datasets - [Iterate over a project](../common/iterate-over-a-project.md)

Here's how to navigate and work with them:

{% tabs %}

{% tab title="Iterate Through Dataset Tree" %}

```python
import supervisely as sly

api = sly.Api.from_env()

project_id = 172

# Use the tree() method to efficiently iterate through the dataset hierarchy
print("Dataset Hierarchy (using tree method):")
for parents, dataset in api.dataset.tree(project_id):
    # parents is a list of parent dataset names, empty list for root datasets
    indent = "  " * len(parents)
    if not parents:
        print(f"{indent}- {dataset.name} (ID: {dataset.id})")
    else:
        parent_path = " > ".join(parents)
        print(f"{indent}|- {dataset.name} (ID: {dataset.id}, Path: {parent_path})")

# Output will look like this:
# - DS1 (ID: 328)
#   |- DS1-1 (ID: 383, Path: DS1)
#     |- DS1-1-1 (ID: 384, Path: DS1 > DS1-1)
# - DS2 (ID: 382)
#   |- DS2-1 (ID: 385, Path: DS2)
#     |- DS2-1-1 (ID: 774, Path: DS2 > DS2-1)
#       |- DS2-1-1-1 (ID: 775, Path: DS2 > DS2-1 > DS2-1-1)
```

{% endtab %}

{% tab title="Dictionary Structure" %}

```python
import json
import supervisely as sly

api = sly.Api.from_env()

project_id = 172

# Get the dataset tree as a dictionary structure
original_tree = api.dataset.get_tree(project_id)
# Convert tree to use dataset `[ID] Name` as keys instead of DatasetInfo objects for better representation
def convert_tree_to_id_keys(tree: dict) -> dict:
    id_tree = {}
    for dataset_info, children in tree.items():
        id_tree[f"[{dataset_info.id}] {dataset_info.name}"] = {
            "children": convert_tree_to_id_keys(children) if children else {}
        }
    return id_tree
dataset_tree = convert_tree_to_id_keys(original_tree)
# You can now navigate this tree structure programmatically
print("\nDataset Tree Structure (using get_tree method): ")
print(json.dumps(dataset_tree, indent=2))

# Output will look like this:
# {
#   "[328] DS1": {
#     "children": {
#       "[383] DS1-1": {
#         "children": {
#           "[384] DS1-1-1": {
#             "children": {}
#           }
#         }
#       }
#     }
#   },
#   "[382] DS2": {
#     "children": {
#       "[385] DS2-1": {
#         "children": {
#           "[774] DS2-1-1": {
#             "children": {
#               "[775] DS2-1-1-1": {
#                 "children": {}
#               }
#             }
#           }
#         }
#       }
```

{% endtab %}

{% tab title="Flat List" %}

```python
import supervisely as sly

api = sly.Api.from_env()

project_id = 172

# Get all datasets including nested ones with recursive=True
print("\nAll Datasets (Flat List):")
all_datasets = api.dataset.get_list(project_id, recursive=True)
for ds in all_datasets:
    parent_info = f"(Parent ID: {ds.parent_id})" if ds.parent_id is not None else "(Root)"
    print(f"- [{ds.id}] {ds.name} {parent_info}")

# Output will look like this:
# - [328] DS1 (Root)
# - [382] DS2 (Root)
# - [383] DS1-1 (Parent ID: 328)
# - [384] DS1-1-1 (Parent ID: 383)
# - [385] DS2-1 (Parent ID: 382)
# - [774] DS2-1-1 (Parent ID: 385)
# - [775] DS2-1-1-1 (Parent ID: 774)
```

{% endtab %}

{% tab title="Nested of a Parent" %}

```python
import supervisely as sly

api = sly.Api.from_env()

project_id = 172
dataset_id = 385  # Specify parent dataset ID

# Get only nested datasets of a specific parent dataset
print(f"\nNested Datasets for Dataset ID {dataset_id}:")
nested_datasets = api.dataset.get_nested(project_id, dataset_id)
for ds in nested_datasets:
    print(f"- [{ds.id}] {ds.name} (Parent ID: {ds.parent_id})")

# Output will look like this:
# - [774] DS2-1-1 (Parent ID: 385)
# - [775] DS2-1-1-1 (Parent ID: 774)
```

{% endtab %}

{% endtabs %}

### Downloading Specific Datasets

To download specific datasets from a project, you can use the convenient `download_fast` mentioned above.

When you specify a dataset ID, the function will:

-   Create the folder structure up to the parent dataset level
-   Download only the images and annotations for the specified dataset
-   Skip downloading images from parent datasets in the hierarchy
-   Skip downloading any nested child datasets that might exist under your specified dataset

If you need to download an entire branch of the dataset hierarchy (a dataset and all its nested children), you would need to provide all the relevant dataset IDs in the `dataset_ids` parameter.

```python
import supervisely as sly

api = sly.Api.from_env()

project_id = 10
dataset_ids = [39]
save_path = 'Pascal_VOC_2012/train'
sly.fs.mkdir(save_path)
sly.download_fast(
    api=api,
    project_id=project_id,
    dest_dir=save_path,
    dataset_ids=dataset_ids,
)
```

## Dataset Images Asynchronous Downloads

### Download Methods

For better performance, you can use asynchronous methods even in a synchronous context:

```python
import supervisely as sly

coroutine = download_nps_async(img_ids)
img_nps = sly.run_coroutine(coroutine)
```

The table below lists various asynchronous methods available in the Supervisely SDK for downloading images in different formats and output types. These methods can significantly improve download performance compared to their synchronous counterparts, especially when working with large datasets.

| Method                           | Description                                                                                                  |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `download_np_async`              | Downloads a single image as numpy array                                                                      |
| `download_nps_async`             | Downloads multiple images as numpy arrays                                                                    |
| `download_path_async`            | Downloads a single image to a specified path                                                                 |
| `download_paths_async`           | Downloads multiple images to specified paths                                                                 |
| `download_bytes_single_async`    | Downloads a single image as bytes                                                                            |
| `download_bytes_many_async`      | Downloads multiple images as bytes in parallel (one request per image)                                       |
| `download_bytes_generator_async` | Downloads multiple images as bytes using a single batch request, yielding results through an async generator |

### Performance Comparison

Let's compare synchronous and asynchronous download methods, for example as numpy array:

{% tabs %}

{% tab title="One by one" %}

```python
import supervisely as sly
from time import monotonic
from tqdm import tqdm

api = sly.Api.from_env()

dataset_id = 1037458

# Get image IDs for testing
image_infos = api.image.get_list(dataset_id)
img_ids = [info.id for info in image_infos[:1000]]  # Using first 1000 images

print(f"Testing download performance with {len(img_ids)} images...")

# 1. Synchronous download (one by one)
start_time = monotonic()
img_nps = []
progress = tqdm(total=len(img_ids), desc="Sync download")
for img_id in img_ids:
    img_nps.append(api.image.download_np(img_id))
    progress.update(1)
sync_time = monotonic() - start_time
print(f"Synchronous download took {sync_time:.2f} seconds")
```

{% endtab %}

{% tab title="Batch by batch" %}

```python
import supervisely as sly
from time import monotonic
from tqdm import tqdm

api = sly.Api.from_env()

dataset_id = 1037458

# Get image IDs for testing
image_infos = api.image.get_list(dataset_id)
img_ids = [info.id for info in image_infos[:1000]]  # Using first 1000 images

print(f"Testing download performance with {len(img_ids)} images...")

# 2. Batch synchronous download with predefined batch sizes: 50
start_time = monotonic()
progress = tqdm(total=len(img_ids), desc=f"Batch download")
img_nps = api.image.download_nps(dataset_id, img_ids, progress_cb=progress)
batch_time = monotonic() - start_time
print(f"Batch download took {batch_time:.2f} seconds")
```

{% endtab %}

{% tab title="Asynchronous" %}

```python
import supervisely as sly
from time import monotonic
from tqdm import tqdm

api = sly.Api.from_env()

dataset_id = 1037458

# Get image IDs for testing
image_infos = api.image.get_list(dataset_id)
img_ids = [info.id for info in image_infos[:1000]]  # Using first 1000 images

print(f"Testing download performance with {len(img_ids)} images...")

# 3. Asynchronous download
start_time = monotonic()
progress = tqdm(total=len(img_ids), desc="Async download")
# Run async function in synchronous context
img_nps = sly.run_coroutine(api.image.download_nps_async(img_ids))
async_time = monotonic() - start_time
print(f"Asynchronous download took {async_time:.2f} seconds")
```

{% endtab %}

{% tab title="Calculate speedups" %}

```python
# Calculate speedups
print("\nSpeedup compared to synchronous download:")

batch_speedup = sync_time / batch_time
print(f"Batch: {batch_speedup:.2f}x faster")

async_speedup = sync_time / async_time
print(f"Asynchronous: {async_speedup:.2f}x faster")
```

{% endtab %}

{% endtabs %}

{% hint style="success" %}

**Images**

The performance improvement from synchronous to batch to asynchronous methods can be dramatic:

-   Batch: ~2.2x speedup
-   Asynchronous: **~14.6x** speedup

{% endhint %}

Results was obtained on [Pascal VOC 2012](https://datasetninja.com/pascal-voc-2012) dataset which you could download from [datasetninja.com](https://datasetninja.com/)

| Method                | Description                     | Pros                                          | Cons                                  | Best For                  |
| --------------------- | ------------------------------- | --------------------------------------------- | ------------------------------------- | ------------------------- |
| Single download       | Download one image at a time    | Simple to implement, minimal memory usage     | Very slow for many images             | Small projects, debugging |
| Batch download        | Download images in groups       | Better network utilization, simple API        | Blocking operation                    | Medium-sized projects     |
| Asynchronous download | Non-blocking parallel downloads | Highest performance, efficient resource usage | Limited by network/system performance | Large projects            |

Using asynchronous downloads with proper concurrency control (via semaphores) enables you to get the best possible performance while managing system resource usage.

## Advanced Annotation Downloads

### Synchronous Annotation Downloads

First, we'll compare what speed increase we get when downloading annotations in batches with a fixed size of 50. This size remains constant since an optimal value has been chosen that will work efficiently for any instance configuration.

{% tabs %}

{% tab title="One by one" %}

```python
import supervisely as sly
from time import monotonic
from tqdm import tqdm

api = sly.Api.from_env()

dataset_id = 1037458

# Get image IDs
image_infos = api.image.get_list(dataset_id)
img_ids = [info.id for info in image_infos[:1000]]  # Using first 1000 images

print(f"Testing annotation download performance with {len(img_ids)} images...")

# 1. Synchronous download (one by one)
start_time = monotonic()
anns = []
progress = tqdm(total=len(img_ids), desc="Sync download")
for img_id in img_ids:
    anns.append(api.annotation.download(img_id))
    progress.update(1)
sync_time = monotonic() - start_time
print(f"Synchronous annotations download took {sync_time:.2f} seconds")
```

{% endtab %}

{% tab title="Batch by batch" %}

```python
import supervisely as sly
from time import monotonic
from tqdm import tqdm

api = sly.Api.from_env()

dataset_id = 1037458

# Get image IDs
image_infos = api.image.get_list(dataset_id)
img_ids = [info.id for info in image_infos[:1000]]  # Using first 1000 images

print(f"Testing annotation download performance with {len(img_ids)} images...")

# 2. Batch synchronous download with predefined batch sizes: 50
start_time = monotonic()
progress = tqdm(total=len(img_ids), desc=f"Batch download")
anns = api.annotation.download_batch(dataset_id, img_ids, progress_cb=progress)
batch_time = monotonic() - start_time
print(f"Batch download took {batch_time:.2f} seconds")
```

{% endtab %}

{% tab title="Calculate speedups" %}

```python
# Calculate speedups
batch_speedup = sync_time / batch_time
print(f"Speedup of batch download: {batch_speedup:.2f}x faster")
```

{% endtab %}

{% endtabs %}

### Asynchronous Annotation Downloads

There are two methods for asynchronous annotation downloading that are used depending on the types of annotations in your dataset images.

| Method                 | Best For                                             |
| ---------------------- | ---------------------------------------------------- |
| `download_batch_async` | Standard annotations for normal-sized images         |
| `download_bulk_async`  | Small or simple annotations for smaller-sized images |

{% hint style="info" %}

To apply these methods effectively, you can separate images into different lists based on their size information.

{% endhint %}

{% tabs %}

{% tab title="Multiple in parallel" %}

```python
import supervisely as sly
from time import monotonic
from tqdm import tqdm
import asyncio

api = sly.Api.from_env()

dataset_id = 1037458

# Get image IDs
image_infos = api.image.get_list(dataset_id)
img_ids = [info.id for info in image_infos[:1000]]  # Using first 1000 images

print(f"Testing annotation download performance with {len(img_ids)} images...")

# Method 3: Download multiple annotations in parallel (one request per image)
# Adjust the number of concurrent requests depending on your instance limitations
semaphore = asyncio.Semaphore(4)

progress = tqdm(total=len(img_ids), desc=f"Async Batch download")
download_coroutine = api.annotation.download_batch_async(
    dataset_id,
    img_ids,
    progress_cb=progress,
    semaphore=semaphore,
)
start_time = monotonic()
anns = sly.run_coroutine(download_coroutine)
async_batch_time = monotonic() - start_time
print(f"Downloaded {len(anns)} annotations in {async_batch_time:.2f} seconds")
```

{% endtab %}

{% tab title="Multiple batches in parallel" %}

```python
import supervisely as sly
from time import monotonic
from tqdm import tqdm
import asyncio

api = sly.Api.from_env()

dataset_id = 1037458

# Get image IDs
image_infos = api.image.get_list(dataset_id)
img_ids = [info.id for info in image_infos[:1000]]  # Using first 1000 images

print(f"Testing annotation download performance with {len(img_ids)} images...")

# Method 4: Download multiple annotations in parallel in batches
tasks = []
progress = tqdm(total=len(img_ids), desc=f"Async Bilk download")
for batch in sly.batched(img_ids):
    download_coroutine = api.annotation.download_bulk_async(
        dataset_id,
        batch,
        progress_cb=progress,
    )
    tasks.append(download_coroutine)

start_time = monotonic()
anns_batches = sly.run_coroutine(asyncio.gather(*tasks))
anns = []
for batch_result in anns_batches:
    anns.extend(batch_result)
async_bulk_time = monotonic() - start_time
print(f"Downloaded {len(anns)} annotations in {async_bulk_time:.2f} seconds")
```

{% endtab %}

{% tab title="Calculate speedups" %}

```python
# Calculate speedups
async_batch_speedup = sync_time / async_batch_time
print(f"Speedup of async batch download: {async_batch_speedup:.2f}x faster")
async_bulk_speedup = sync_time / async_bulk_time
print(f"Speedup of async batch download: {async_bulk_speedup:.2f}x faster")
```

{% endtab %}

{% endtabs %}

Following results was obtained on [Pascal VOC 2012](https://datasetninja.com/pascal-voc-2012) dataset which you could download from [datasetninja.com](https://datasetninja.com/)

{% hint style="success" %}

**Annotations**

The performance improvement from synchronous to batch to asynchronous methods:

-   Batch: ~3.4x speedup
-   Asynchronous: ~8.1x speedup
-   Asynchronous batch: ~19x speedup

Your specific speedup may differ from these benchmarks depending on: number of annotations on images, complexity of annotations, image size (annotation size), network conditions, server load.

{% endhint %}

The benefits of asynchronous downloading:

1. **Parallel processing**: Multiple batches can be downloaded simultaneously
2. **Better resource utilization**: Network I/O doesn't block the application
3. **Improved throughput**: Especially noticeable with many small files
4. **Reduced total processing time**: Significant reduction for large datasets

### Choosing the Right Async Method

For best performance, consider these guidelines:

1. Use `semaphore` to control concurrency (typically 5-20 concurrent downloads)
2. The `download_bulk_async` method is generally fastest for datasets with many small annotations
3. For complex annotations with alpha masks or large bitmaps, `download_batch_async` with a smaller semaphore value may work better
4. When using `ApiContext`, the methods automatically use the project metadata to avoid redundant API calls:

```python
# Optimize downloads with ApiContext
project_id = api.dataset.get_info_by_id(dataset_id).project_id
project_meta = api.project.get_meta(project_id)

with sly.ApiContext(api, dataset_id=dataset_id, project_id=project_id, project_meta=project_meta):
    annotations = sly.run_coroutine(download_bulk_async())
```

## Figures Bulk Download

When working with datasets that have large numbers of annotations, downloading figures in bulk can significantly improve performance. Supervisely provides dedicated API methods for this purpose.

The bulk figure download approach is particularly effective when:

-   You need to analyze annotation distribution without loading full data
-   You're developing a custom export pipeline to another format
-   You need to visualize or process specific types of annotations
-   Your dataset contains many images with hundreds or thousands of annotations

### Understanding Figures vs Annotations

In Supervisely's data model:

-   **Annotations** contain all information about labeled objects, including tags and metadata
-   **Figures** represent the geometric shapes that define objects in images (rectangles, polygons, bitmaps, etc.)

For many ML tasks, you might only need the geometric information without all the associated metadata.

### Basic Figures Download

`FigureInfo` represents detailed information about a figure: geometry, tags, metadata etc.<br>
Here's how to get `FigureInfo` for images in a dataset.

```python
import supervisely as sly
from tqdm import tqdm

api = sly.Api.from_env()

# Define dataset ID
dataset_id = 254737

# Download all figures for a dataset
# Returns a dictionary where keys are image IDs and values are lists of figures
figures_dict = api.image.figure.download(dataset_id)
figure_ids = []
# Process each image's figures
for image_id, figures in figures_dict.items():
    print(f"Image ID: {image_id}, Number of figures: {len(figures)}")
    for figure in figures:
        print(f"  - Figure ID: {figure.id}, Class ID: {figure.class_id}, Type: {figure.geometry_type}")
        figure_ids.append(figure.id)
```

### Optimized Figures Download for Large Datasets

For large datasets, you can skip downloading the geometry data initially to speed up the process.

The bulk figures download offers several advantages:

1. **Reduced network overhead**: Only essential figure data is transferred
2. **Faster processing**: Server-side filtering minimizes data transfer
3. **Lower memory usage**: Only relevant geometry information is returned
4. **Simplified post-processing**: Data is already in the required format

```python

print(f"Found {len(figure_ids)} figures in the dataset")

# Then download only the geometries you need in batches
progress = tqdm(total=len(figure_ids), desc="Downloading geometries")

geometries = api.image.figure.download_geometries_batch(figure_ids, progress_cb=progress)

# Process geometries
for figure_id, geometry in zip(figure_ids, geometries):
    # Your processing code here
    pass
```

### Advanced: Asynchronous Geometry Download

For even better performance with large datasets, you can use asynchronous downloads:

```python
progress = tqdm(total=len(figure_ids), desc="Downloading geometries")

# Download geometries asynchronously
download_coroutine = api.figure.download_geometries_batch_async(
    all_figure_ids,
    progress_cb=progress.update
)

geometries = sly.run_coroutine(download_coroutine)

print(f"Downloaded {len(geometries)} geometries")

```

### Working with AlphaMask Geometries

For advanced cases like AlphaMask geometries, you'll need to handle the upload and download separately:

```python
import numpy as np
import supervisely as sly
from supervisely.geometry.constants import BITMAP

api = sly.Api.from_env()

# Example: After creating figures with AlphaMask geometries
# You need to upload the actual mask data
figure_ids = [123456, 123457]  # IDs of figures with AlphaMask geometries
geometries = [
    sly.AlphaMask(data=np.random.randint(0, 255, (100, 100), dtype=np.uint8)).to_json()[BITMAP],
    sly.AlphaMask(data=np.random.randint(0, 255, (120, 120), dtype=np.uint8)).to_json()[BITMAP],
]

# Upload geometries for the figures
api.image.figure.upload_geometries_batch(figure_ids, geometries)

# Later, you can download these geometries
downloaded_geometries = api.image.figure.download_geometries_batch(figure_ids)
```

### Performance Tips for Figure Downloads

1. **Use `skip_geometry=True`** when you only need figure metadata initially
2. **Download geometries in batches** (optimal batch size is typically 50-200)
3. **Use asynchronous methods** for large datasets with many figures
4. **Process figures by type** - some geometry types might need special handling
5. **Consider memory constraints** when downloading many complex geometries

## Conclusion

When downloading data from Supervisely, choosing the right method can dramatically impact performance. Single downloads are simple but inefficient for large datasets, suitable only for debugging or working with a few images. Batch downloads offer a good balance of simplicity and performance for medium-sized projects, improving network utilization while remaining easy to implement. For large-scale projects with thousands of images or annotations, asynchronous downloads deliver the best performance—up to 19x faster than sequential downloads—by efficiently utilizing network resources and processing multiple requests in parallel. Remember to use semaphores to control concurrency and consider the specific characteristics of your data (image sizes, annotation complexity) when selecting a download method. By implementing the appropriate download strategy for your project's scale, you can significantly reduce processing time and improve overall workflow efficiency.
