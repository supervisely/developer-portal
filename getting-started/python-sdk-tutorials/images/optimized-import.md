# Advanced Tutorial: Optimized Import of Small Images

## Overview

This tutorial explains how to optimize the import of small images to Supervisely using blob files. This approach is more efficient for uploading and downloading numerous very small images compared to standard methods.

## Understanding the Blob Import Approach

When dealing with large quantities of small images (e.g., thousands of images under 100KB each), importing them individually is inefficient. The blob approach combines multiple images into a single archive file, making transfer and storage more efficient.

Benefits include:

-   Faster import and export speeds
-   Reduced server load
-   More efficient storage on disk

## Preparing Data for Blob Import

First, let's prepare our images and annotations:

```python
import os
import supervisely as sly
from pathlib import Path
from supervisely.project.project import TF_BLOB_DIR
from tqdm import tqdm

# Set paths
imgs_path = "your_local_images_dir"
anns_path = "your_local_annotations_dir"
blob_dir = "blob_archive_dir"
tar_name = "images.tar"
tar_path = f"{blob_dir}/{tar_name}"
sly.fs.mkdir(blob_dir)

# Get number of images in directory
# Count images in the directory
images_count = len([f for f in os.scandir(imgs_path) if f.is_file() and f.name.lower().endswith(sly.image.SUPPORTED_IMG_EXTS)])
print(f"Found {images_count} images in {imgs_path}")

# Create a tar archive from your images
sly.fs.archive_directory(imgs_path, tar_path)

# Create offsets file for the archive
# This is important for efficient blob operations
offsets_path = sly.fs.save_blob_offsets_pkl(
    blob_file_path=tar_path,
    output_dir=blob_dir
)
print(f"Created offsets file: {offsets_path}")

# You can also create offsets for specific images using filters
# For example, to include only JPEG files:
def filter_jpeg_only(filename):
    return filename.lower().endswith(('.jpg', '.jpeg'))

filtered_offsets_path = sly.fs.save_blob_offsets_pkl(
    blob_file_path=tar_path,
    output_dir=blob_dir,
    filter_func=filter_jpeg_only
)
print(f"Created filtered offsets file: {filtered_offsets_path}")

# Another example: filter by name pattern
def filter_by_pattern(filename):
    return "car" in filename.lower() or "vehicle" in filename.lower()

pattern_offsets_path = sly.fs.save_blob_offsets_pkl(
    blob_file_path=tar_path,
    output_dir=blob_dir,
    filter_func=filter_by_pattern
)
```

## Uploading to Team Files

After preparing the `.tar` and offsets `.pkl` files, upload it to Team Files:

```python
api = sly.Api.from_env()
team_id = 123  # Replace with your team ID

# Upload blob file to team files
remote_path = f"/{TF_BLOB_DIR}/{tar_name}"
blob_tf_info = api.file.upload(team_id, tar_path, remote_path)

# Upload file with offsets to team files
offsets_file_name = Path(offsets_path).name
remote_offsets_path = f"/{TF_BLOB_DIR}/{offsets_file_name}"
offsetst_tf_info = api.file.upload(team_id, offsets_path, remote_offsets_path)
```

## Creating a Project with Blob Images

Now create a project and dataset, then upload the blob images:

```python
# Create project and dataset
project = api.project.create(team_id, "Blob Images Project", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, "dataset_1")

# Upload images by offsets from the blob file to dataset
dataset_images = api.image.upload_blob_images(
    dataset=dataset,
    blob_file=blob_tf_info,
    progress_cb=tqdm(desc="Uploading images", total=images_count)
)
```

## Adding Annotations to Blob Images

After uploading images, add annotations:

```python
# Assuming annotations are already prepared in Supervisely format
# Upload annotations in batches to avoid high memory usage

# Define batch size
BATCH_SIZE = sly.fs.OFFSETS_PKL_BATCH_SIZE # 10 000

# Process images in batches
for i in range(0, len(dataset_images), BATCH_SIZE):
    batch_images = dataset_images[i:i + BATCH_SIZE]
    print(f"Processing batch {i//BATCH_SIZE + 1}, images {i+1} to {min(i+BATCH_SIZE, len(dataset_images))}")

    ids_batch = []
    ann_batch = []
    # Process each image in the current batch
    for img_info in batch_images:
        # Construct the path to the corresponding annotation file
        ann_file = os.path.join(anns_path, f"{img_info.name}.json")

        if os.path.exists(ann_file):
            # Load the annotation from file
            ids_batch.append(img_info.id)
            ann_batch.append(ann_file)
        else:
            print(f"Annotation file for image '{img_info.name}' is not found. Skipping.")

    # Upload batch of annotations
    ann_progress_cb=tqdm(desc=f"Uploading annotations batch {i//BATCH_SIZE + 1}", total=len(ids_batch))
    api.annotation.upload_paths(ids_batch, ann_batch, ann_progress_cb)
```

## Upload a Blob Project

If you already have a local Supervisely project with blob files, you can upload it directly to the platform:

```python

workspace_id = 345

# Path to your local project with blob structure
local_project_path = "/path/to/local/blob/project"

# Upload the project with progress tracking
project_id, project_name = sly.upload_project(
    dir=local_project_path,
    workspace_id=workspace_id,
    project_name="My Blob Project",
)

print(f"Project '{project_name}' has been uploaded. Project ID: {project_id}")
```

The `upload` method automatically handles blob files in your local project structure. During upload:

1. Blob archives (`.tar` files) are uploaded to Team Files
2. Offset files (`.pkl`) are uploaded alongside the archives
3. Images are registered in the platform using blob references instead of uploading each file
4. All annotations are preserved with their connections to the blob images

This approach is significantly faster than standard upload methods for projects with many small images.

## Downloading a Blob Project

To download a project that contains blob images:

```python
# Download the entire project efficiently
local_project_dir = "downloaded_project"
sly.download_fast(
    api=api,
    project_id=project.id,
    dest_dir=local_project_dir,
    download_blob_files=True  # Important for blob images
)
```

## Working with Local Blob Project

Access the downloaded project and iterate through it:

```python
project_fs = sly.Project(local_project_dir, sly.OpenMode.READ)
meta = project_fs.meta

# Iterate through datasets and images
for dataset_fs in project_fs:
    print(f"Dataset: {dataset_fs.name}")

    for item_name in dataset_fs.get_items_names():
        # Get image path and annotation
        item_path = dataset_fs.get_img_path(item_name)
        ann = dataset_fs.get_ann(item_name, project_fs.meta)

        # Process as needed
        print(f"  - Image: {item_name}, Labels: {len(ann.labels)}")
```

#

## BlobImageInfo

### Description

The `BlobImageInfo` class represents image metadata within a blob storage file. It contains information about where the image data is located in the blob file, defined by byte offsets. This class provides methods to manipulate and convert blob image information to formats suitable for storage and API interactions.

### Methods

| Method                                                                                    | Description                                                                                                        |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `from_image_info(cls, image_info: ImageInfo) -> BlobImageInfo`                            | Static method to create a BlobImageInfo instance from an ImageInfo object.                                         |
| `add_team_file_id(self, team_file_id: int)`                                               | Adds a team file ID to the BlobImageInfo instance. This ID links the image to the blob file in team storage.       |
| `to_dict(self, team_file_id: int = None) -> Dict`                                         | Converts BlobImageInfo to a dictionary format suitable for serialization. Includes team_file_id if provided.       |
| `from_dict(cls, data: Dict) -> BlobImageInfo`                                             | Static method to create a BlobImageInfo instance from a dictionary representation.                                 |
| `load_from_pickle_generator(cls, file_path: str) -> Generator[BlobImageInfo, None, None]` | Static method that creates a generator yielding BlobImageInfo instances from a pickle file containing offset data. |
| `dump_to_pickle(cls, blob_image_infos: List[BlobImageInfo], path: str) -> None`           | Static method that saves a list of BlobImageInfo instances to a pickle file.                                       |
| `offsets_dict(self) -> Dict[str, int]`                                                    | Property that returns a dictionary with the offset start and end values for the image data in the blob file.       |

## Blob Methods Reference

Here's a comprehensive table of methods related to blob operations in Supervisely:

| Method                               | Module         | Description                                                                  |
| ------------------------------------ | -------------- | ---------------------------------------------------------------------------- |
| `save_blob_offsets_pkl()`            | `fs.py`        | Generates a pickle file with image offsets in a blob archive                 |
| `get_file_offsets_batch_generator()` | `fs.py`        | Creates a generator that yields batches of image offsets from a blob archive |
| `upload_by_offsets()`                | `image_api.py` | Uploads images to Supervisely using offsets from a blob file                 |
| `upload_by_offsets_generator()`      | `image_api.py` | Generator version of upload_by_offsets for memory-efficient uploads          |
| `get_blob_offsets_file()`            | `image_api.py` | Downloads a blob offsets file from Team Files                                |
| `download_blob_file()`               | `image_api.py` | Downloads a blob file from Supervisely by project ID and download ID         |
| `upload_blob_images()`               | `image_api.py` | Uploads images from a blob file to a dataset using file offsets              |
| `download_blob_files_async()`        | `image_api.py` | Asynchronously downloads multiple blob files to specified paths              |
| `add_blob_file()`                    | `project.py`   | Adds a blob file to a local project structure                                |
| `create_blob_readme()`               | `project.py`   | Creates documentation for a blob-based project structure                     |

This table covers the core methods you'll use when working with blob files in Supervisely, from creating and uploading blobs to downloading and processing them.

## Performance Comparison

A blob project with 10000 small images (30KB each) can be:

-   Uploaded ~5x faster than individual uploads
-   Downloaded ~10x faster than individual downloads

## Best Practices

1. Use blob approach for collections with many small images
2. Batch operations in groups of 10000 images
3. Always save image metadata when downloading
4. Monitor memory usage when processing thousands of images
