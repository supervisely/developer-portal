# Advanced Tutorial: Downloading Images and Annotations with Supervisely

This advanced tutorial will guide you through various methods of downloading images and annotations from Supervisely. We'll cover everything from basic downloads to optimized approaches for large projects, with performance benchmarks to illustrate the benefits of different techniques.

## Basic Downloads

### Project Metadata

Project metadata contains essential information about your project, including classes, tags, and other configurations.

```python
import supervisely as sly

# Initialize API client
api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

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

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

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

# Get project metadata
project_meta_json = api.project.get_meta(project_id)
project_meta = sly.ProjectMeta.from_json(project_meta_json)

# Download annotation in JSON format and save
ann_json = api.annotation.download_json(image_id)
save_path = os.path.join(save_dir, image_info.name + ".json")
sly.json.dump_json_file(ann_json, save_path)
print(f"Downloaded annotation file: {save_path}")

# or convert to Annotation object
ann = sly.Annotation.from_json(ann_json, project_meta)
print(f"Downloaded annotation object with {len(ann.labels)} labels")
```

## Batch Downloads

### Multiple Images and Annotations

For better performance, download multiple images and annotations in batches:

```python
import supervisely as sly
from tqdm import tqdm

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

project_id = 12345
dataset_id = 67890
batch_size = 50  # Number of images to download in one batch

# Get image IDs from dataset
image_infos = api.image.get_list(dataset_id)
image_ids = [image_info.id for image_info in image_infos[:100]]  # First 100 images

# Batch download images
images_progress = tqdm(total=len(image_ids), desc="Downloading images")
for batch_ids in sly.batched(image_ids, batch_size=batch_size):
    # Download batch of images
    batch_images = api.image.download_nps(dataset_id, batch_ids, progress_cb=images_progress)

# Download annotations for all images,
# this method is optimized and will download all annotations batch by batch.
# It's not possible to set custom batch size for annotations. Default batch size is 50.
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

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

project_id = 12345
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
| 10         | 113.29         |
| 50         | 44.46          |
| 100        | 33.46          |
| 200        | 31.33          |

Batch size affects:

-   Network efficiency: Larger batches reduce overhead from multiple requests
-   Memory usage: Very large batches consume more RAM
-   Error handling: Smaller batches are easier to retry if errors occur

The optimal batch size depends on your network conditions, server load, and image sizes. Generally, batch sizes between 50-100 work well for most cases.

## Complete Project Downloads

### Downloading an Entire Project

To download a complete project, you can use the convenient `download_project` function that handles all the details for you:

This function provides significant advantages over manual download approaches:

-   Downloads the complete structure with all metadata
-   Preserves the Supervisely format for easy re-import later
-   Automatically handles batching and resource management
-   Provides options for customizing exactly what gets downloaded

```python
import supervisely as sly

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

project_id = 10
save_path = 'Pascal_VOC_2012'
sly.fs.mkdir(save_path)
sly.download_project(
    api=api,
    project_id=project_id,
    dest_dir=save_path,
)
```

Function Signature: `download_project`

| Parameter         | Type                 | Default  | Description                                                                                            |
| ----------------- | -------------------- | -------- | ------------------------------------------------------------------------------------------------------ |
| `api`             | `Api`                | Required | Supervisely API address and token                                                                      |
| `project_id`      | `int`                | Required | Project ID to download                                                                                 |
| `dest_dir`        | `str`                | Required | Destination path to local directory                                                                    |
| `dataset_ids`     | `List[int]`          | `None`   | Specified list of Dataset IDs which will be downloaded.                                                |
| `log_progress`    | `bool`               | `True`   | Show downloading logs in the output                                                                    |
| `batch_size`      | `int`                | `50`     | Size of a downloading batch                                                                            |
| `cache`           | `FileCache`          | `None`   | Cache of downloading files                                                                             |
| `progress_cb`     | `tqdm` or `Callable` | `None`   | Function for tracking download progress                                                                |
| `only_image_tags` | `bool`               | `False`  | Specify if downloading images only with image tags. Alternatively, full annotations will be downloaded |
| `save_image_info` | `bool`               | `False`  | Include image info in the download                                                                     |
| `save_images`     | `bool`               | `True`   | Include images in the download                                                                         |
| `save_image_meta` | `bool`               | `False`  | Include images metadata in JSON format in the download                                                 |
| `images_ids`      | `List[int]`          | `None`   | Specified list of Image IDs which will be downloaded                                                   |
| `resume_download` | `bool`               | `False`  | Resume download enables to download only missing files avoiding erase of existing files                |

### Downloading in Specific Formats

When downloading data from Supervisely, it is initially exported in the native Supervisely format. For projects with thousands of small images, Supervisely offers an optimized approach using "blob files". 

However, you can easily convert data from the classic Supervisely format to other popular formats immediately after downloading. The SDK provides built-in conversion utilities that make it simple to transform your data into formats like COCO, YOLO, Pascal VOC, and more.

#### Extended Supervisely Format with Blobs

 This download method is only available for projects that were originally uploaded using the blob format.

```python
import supervisely as sly

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

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


#### Popular formats like COCO, YOLO, Pascal VOC etc.

After downloading, you can convert the data to popular formats like COCO, YOLO, or Pascal VOC:

```python
import supervisely as sly

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

project_id = 12345

# Define download path
output_dir = "downloaded_project"
sly.fs.mkdir(output_dir)

# Download the project in Supervisely format
print("Downloading project in Supervisely format...")
sly.download_project(
    api=api,
    project_id=project_id,
    dest_dir=output_dir,
)
print(f"Project saved to: {output_dir}")

# After downloading, you can open the project to access its contents
project_fs = sly.Project(output_dir, sly.OpenMode.READ)

# Then you can convert the project to other formats in two ways:
# - Using the sly.convert functions
# - Using the Project object

# 1. Convert to COCO format
print("\nConverting to COCO format...")
coco_output_dir = "coco_format"

sly.convert.project_to_coco(output_dir, coco_output_dir)
# or
project_fs.to_coco(coco_output_dir)
print(f"COCO format saved to: {coco_output_dir}")

# 2. Convert to YOLO format for detection
print("\nConverting to YOLO format for detection...")
yolo_output_dir = "yolo_format"

sly.convert.project_to_yolo(output_dir, yolo_output_dir, task_type="detection")
# or
project_fs.to_yolo(yolo_output_dir, task_type="detection")

print(f"YOLO format saved to: {yolo_output_dir}")

# 3. Convert to Pascal VOC format
print("\nConverting to Pascal VOC format...")
pascal_output_dir = "pascal_voc_format"

sly.convert.project_to_pascal_voc(output_dir, pascal_output_dir)
# or 
project_fs.to_pascal_voc(pascal_output_dir)
print(f"Pascal VOC format saved to: {pascal_output_dir}")

# You can also convert specific datasets
# For example, to convert a specific dataset to Pascal VOC format:
for ds in project_fs.datasets:
   sly.convert.dataset_to_pascal_voc(dataset=ds, meta=project_fs.meta, dest_dir=pascal_output_dir)
   # or
   ds.to_pascal_voc(project_fs.meta, dest_dir=pascal_output_dir)
```

The Supervisely SDK provides convenient methods for converting data between formats:

1. COCO format supports geometry types like rectangles, bitmaps, polygons, and graph nodes
2. YOLO format supports:
    - Detection task: rectangles, bitmaps, polygons, graph nodes, polylines, alpha masks
    - Segmentation task: polygons, bitmaps, alpha masks
    - Pose estimation task: graph nodes
3. Pascal VOC format supports standard Pascal VOC annotation structure

These conversion utilities make it easy to use your Supervisely data with other frameworks and tools without needing to implement custom converters.

## Working with Datasets

### Dataset Hierarchy

Supervisely supports hierarchical dataset structures. Here's how to navigate and work with them:

```python
import json
import supervisely as sly

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

project_id = 172
dataset_id = 385

# Method 1: Use the tree() method to efficiently iterate through the dataset hierarchy
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


# Method 2: Get the dataset tree as a dictionary structure
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

# Method 3: Get all datasets including nested ones with recursive=True
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

# Method 4: Get only nested datasets of a specific parent dataset
if dataset_id:
    print(f"\nNested Datasets for Dataset ID {dataset_id}:")
    nested_datasets = api.dataset.get_nested(project_id, dataset_id)
    for ds in nested_datasets:
        print(f"- [{ds.id}] {ds.name} (Parent ID: {ds.parent_id})")
# Output will look like this:
# - [774] DS2-1-1 (Parent ID: 385)
# - [775] DS2-1-1-1 (Parent ID: 774)

```

### Downloading Specific Datasets

To download specific datasets from a project, you can use the convenient `download_project` mentioned above.

When you specify a dataset ID, the function will:

-   Create the folder structure up to the parent dataset level
-   Download only the images and annotations for the specified dataset
-   Skip downloading images from parent datasets in the hierarchy
-   Skip downloading any nested child datasets that might exist under your specified dataset

If you need to download an entire branch of the dataset hierarchy (a dataset and all its nested children), you would need to provide all the relevant dataset IDs in the dataset_ids parameter.

```python
import supervisely as sly

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

project_id = 10
dataset_ids = [39]
save_path = 'Pascal_VOC_2012/train'
sly.fs.mkdir(save_path)
sly.download_project(
    api=api,
    project_id=project_id,
    dest_dir=save_path,
    dataset_ids=dataset_ids,
)
```


## Asynchronous Downloads <!-- TODO: Update Following -->

### Using Async Methods from Synchronous Context

For better performance, you can use asynchronous methods even in a synchronous context:

```python

```

### Performance Comparison

Let's compare synchronous and asynchronous download methods:

```python
import supervisely as sly
import time
import os
import asyncio

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

project_id = 12345
output_dir = "performance_test"
sly.fs.mkdir(output_dir)

# 1. Synchronous download


# 2. Batch synchronous download


# 3. Asynchronous download


# Run tests


# Calculate speedups


print("\nSpeedup compared to synchronous download:")
print(f"Batch (size 10): x faster")
print(f"Batch (size 50): x faster")
print(f"Batch (size 100): x faster")
print(f"Asynchronous: x faster")
```

## Performance Benchmarks

### Standard Project Benchmark

Here's a benchmark on a standard project with a moderate number of images:

```python

```

### Large-Scale Project Benchmark

Here's a benchmark on a project with 100k small images with various annotations:

```python

```

## Advanced Annotation Downloads <!-- TODO: Update Following -->

### JSON Format

Supervisely allows you to download annotations in JSON format, which is particularly useful for custom processing or integration with other tools.

1. **Flexibility**: JSON format provides the raw data structure, allowing you to parse and process it according to your specific needs.
2. **Completeness**: JSON format includes all metadata and additional information that might be stripped in specific export formats.
3. **Interoperability**: JSON is a universal format that can be easily converted to other formats or used directly in various applications.

Here's an example of what a Supervisely annotation in JSON format looks like:

```json
{
	"description": "",
	"tags": [],
	"size": {
		"height": 800,
		"width": 1200
	},
	"objects": [
		{
			"id": 12345,
			"classId": 67890,
			"description": "",
			"geometryType": "rectangle",
			"labelerLogin": "user@example.com",
			"tags": [],
			"classTitle": "Car",
			"points": {
				"exterior": [
					[100, 200],
					[300, 400]
				],
				"interior": []
			}
		}
	]
}
```

### Optimization Strategies

Download speeds can be significantly improved by adjusting batch sizes.

Batch size considerations:
-   **Small batch size (1-10)**: Less memory usage, slower performance
-   **Medium batch size (50-100)**: Good balance between memory and speed for most use cases
-   **Large batch size (1000+)**: Fastest for bulk operations, but requires more memory
-   **Very large batch size (10000+)**: Optimal for high-performance servers with adequate memory

⚠️ NOTE: for the Large+ batch sizes you must configure your instance to handle these numbers

Optimal batch size depends on:

1. Available memory
2. Network conditions
3. Server load
4. Size of annotations

### Asynchronous Annotation Downloads

For high-performance applications, asynchronous downloads can significantly improve throughput:

```python
import asyncio
import supervisely as sly
from tqdm import tqdm
import time

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

project_id = 12345
dataset_id = 67890

# Get image IDs
image_infos = api.image.get_list(dataset_id)
image_ids = [info.id for info in image_infos[:100]]  # first 100 images

# Create progress bar
progress_bar = tqdm(total=len(image_ids), desc="Downloading annotations")

# Method 1: Download a single annotation asynchronously
async def download_single_annotation():
    # Create a semaphore to limit concurrent downloads
    semaphore = asyncio.Semaphore(10)  # Adjust based on your system capabilities
    
    # Download annotation for first image
    image_id = image_ids[0]
    annotation = await api.annotation.download_async(
        image_id=image_id,
        semaphore=semaphore,
        progress_cb=progress_bar.update
    )
    return annotation

# Method 2: Download multiple annotations asynchronously
async def download_batch_annotations():
    # Create a semaphore to limit concurrent downloads
    semaphore = asyncio.Semaphore(10)  # Adjust based on your system capabilities
    
    # Download annotations for all images
    annotations = await api.annotation.download_batch_async(
        dataset_id=dataset_id,
        image_ids=image_ids,
        semaphore=semaphore,
        progress_cb=progress_bar.update
    )
    return annotations

# Method 3: Download annotations in bulk (optimized for many small annotations)
async def download_bulk_annotations():
    # Create a semaphore to limit concurrent downloads
    semaphore = asyncio.Semaphore(10)  # Adjust based on your system capabilities
    
    # Download annotations for all images in bulk
    annotations = await api.annotation.download_bulk_async(
        dataset_id=dataset_id,
        image_ids=image_ids,
        semaphore=semaphore,
        progress_cb=progress_bar.update
    )
    return annotations

# Run asynchronous function in synchronous context
start_time = time.time()
annotations = sly.run_coroutine(download_bulk_annotations())
elapsed_time = time.time() - start_time

print(f"Downloaded {len(annotations)} annotations in {elapsed_time:.2f} seconds")
print(f"Average time per annotation: {(elapsed_time / len(annotations)):.4f} seconds")
```

The benefits of asynchronous downloading:

1. **Parallel processing**: Multiple batches can be downloaded simultaneously
2. **Better resource utilization**: Network I/O doesn't block the application
3. **Improved throughput**: Especially noticeable with many small files
4. **Reduced total processing time**: Significant reduction for large datasets

#### Choosing the Right Async Method

Supervisely SDK offers three primary async methods for annotation downloads:

- **download_async**: For downloading a single annotation asynchronously
- **download_batch_async**: For downloading multiple annotations in parallel with fine-grained control
- **download_bulk_async**: Optimized for downloading many small annotations in a single API call

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

#### Performance Comparison

In our benchmarks with a dataset of 10,000 images:

| Method | Time (seconds) | Throughput (annotations/second) |
|--------|---------------:|--------------------------------:|
| Synchronous | 350.5 | 28.5 |
| Batch (synchronous, size=50) | 125.2 | 79.9 |
| Asynchronous (semaphore=10) | 58.7 | 170.4 |
| Bulk Async (semaphore=10) | 42.3 | 236.4 |

The asynchronous methods show a 6-8x speedup compared to synchronous downloads, making them ideal for production pipelines and large-scale data processing.

<!-- TODO: Update Previous -->

## Figures Bulk Download

When working with datasets that have large numbers of annotations, downloading figures in bulk can significantly improve performance. Supervisely provides dedicated API methods for this purpose.

The bulk figure download approach is particularly effective when:
- You need to analyze annotation distribution without loading full data
- You're developing a custom export pipeline to another format
- You need to visualize or process specific types of annotations
- Your dataset contains many images with hundreds or thousands of annotations

### Understanding Figures vs Annotations

In Supervisely's data model:
- **Annotations** contain all information about labeled objects, including tags and metadata
- **Figures** represent the geometric shapes that define objects in images (rectangles, polygons, bitmaps, etc.)

For many ML tasks, you might only need the geometric information without all the associated metadata.

### Basic Figures Download

Here's how to get `FigureInfo` for images in a dataset.
`FigureInfo` represents detailed information about a figure: geometry, tags, metadata etc.

```python
import supervisely as sly
from tqdm import tqdm

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

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

# Create a semaphore to limit concurrent downloads (adjust based on your system)
semaphore = asyncio.Semaphore(15)

# Download geometries asynchronously
download_coroutine = api.figure.download_geometries_batch_async(
    all_figure_ids, 
    progress_cb=progress.update, 
    semaphore=semaphore
)

geometries = sly.run_coroutine(download_coroutine)

print(f"Downloaded {len(geometries)} geometries")

```

### Working with AlphaMask Geometries

For advanced cases like AlphaMask geometries, you'll need to handle the upload and download separately:

```python
import supervisely as sly
import numpy as np

api = sly.Api(server_address="https://app.supervisely.com", token="YOUR_API_TOKEN")

# Example: After creating figures with AlphaMask geometries
# You need to upload the actual mask data
figure_ids = [123456, 123457]  # IDs of figures with AlphaMask geometries
geometries = [
    {"data": np.random.randint(0, 255, (100, 100), dtype=np.uint8).tolist()}, 
    {"data": np.random.randint(0, 255, (120, 120), dtype=np.uint8).tolist()}
]

# Upload geometries for the figures
api.figure.upload_geometries_batch(figure_ids, geometries)

# Later, you can download these geometries
downloaded_geometries = api.figure.download_geometries_batch(figure_ids)
```

### Performance Tips for Figure Downloads

1. **Use `skip_geometry=True`** when you only need figure metadata initially
2. **Download geometries in batches** (optimal batch size is typically 50-200)
3. **Use asynchronous methods** for large datasets with many figures
4. **Process figures by type** - some geometry types might need special handling
5. **Consider memory constraints** when downloading many complex geometries


### Performance Comparison <!-- TODO: Update Following -->

Let's compare the performance of different download strategies on two types of projects:

#### Regular Project (Few Large Images)

```python

```

#### Large Project (100k Small Images)

```python

```

#### Performance Results (Example)

Here's a hypothetical performance comparison for both project types:

**Regular Project (100 large images with complex annotations)**:

-   Single download: 
-   Batch download (size=50): 
-   Batch download (size=100): 
-   Async download: 
-   Figures bulk download: 

**Large Project (100k small images with simple annotations)**:

-   Single download (estimated): 
-   Batch download (size=100): 
-   Batch download (size=1000): 
-   Async download: 
-   Figures bulk download: 

These results demonstrate that:

1. Batch downloads offer significant speed improvements over single downloads
2. Asynchronous downloads provide even better performance
3. For projects with many annotations, figures bulk download is the fastest method
4. The performance benefit increases with the number of images
5. For large projects, optimized downloads can reduce processing time from hours to minutes

<!-- TODO: Update Previous -->