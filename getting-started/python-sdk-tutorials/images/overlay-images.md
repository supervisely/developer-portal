# Overlay images in labeling interface

## Introduction

In this tutorial, you will learn how to upload source images together with one or multiple overlay images for each source image using Python SDK. Overlay images are displayed as additional visual layers in the labeling interface and are linked to their parent (source) images.

In the Overlay labeling interface, overlay visibility is controlled by adjustable opacity.

## Recommended input structure

Both directory and archive are supported.

```text
📦input_folder
┣ 📂dataset_name
┃  ┣ 📂ann
┃  ┃  ┣ 📄scene_01.jpg.json
┃  ┃  ┗ 📄scene_02.jpg.json
┃  ┣ 📂img
┃  ┃  ┣ 🖼️scene_01.jpg
┃  ┃  ┗ 🖼️scene_02.jpg
┃  ┗ 📂overlay
┃     ┣ 📂scene_01.jpg
┃     ┃  ┣ 🖼️mask.png
┃     ┃  ┗ 🖼️heatmap.png
┃     ┗ 📂scene_02.jpg
┃        ┗ 🖼️mask.png
```

- Parent images are stored in `img`.
- Parent image annotations are stored in `ann` as `image_name.ext.json`.
- Overlay images are stored in `overlay/<parent_image_name_with_extension>/`.
- Overlay images do not require annotation files.

## How to upload overlay images with Python SDK

Use `api.image.upload_overlay_images`.

```python
def upload_overlay_images(
    dataset_id: int,
    names: List[str],
    paths: Optional[List[str]] = None,
    links: Optional[List[str]] = None,
    hashes: Optional[List[str]] = None,
    overlay_names: Optional[List[List[str]]] = None,
    overlay_paths: Optional[List[List[str]]] = None,
    overlay_links: Optional[List[List[str]]] = None,
    overlay_hashes: Optional[List[List[str]]] = None,
    batch_size: Optional[int] = 50,
    conflict_resolution: Optional[Literal["rename", "skip", "replace"]] = "rename",
    force_metadata_for_links: Optional[bool] = False,
) -> Tuple[List[ImageInfo], List[List[ImageInfo]]]:
```

|        Parameters        |                 Type                 |                                                Description                                                 |
| :----------------------: | :----------------------------------: | :--------------------------------------------------------------------------------------------------------: |
|        dataset_id        |                 int                  |                                        ID of the dataset to upload                                         |
|          names           |              List[str]               |                                             Parent image names                                             |
|          paths           |         Optional[List[str]]          |                              List of local paths to parent images (optional)                               |
|          links           |         Optional[List[str]]          |                              List of remote links to parent images (optional)                              |
|          hashes          |         Optional[List[str]]          |                       List of hashes for parent images already in storage (optional)                       |
|      overlay_names       |      Optional[List[List[str]]]       |            Overlay names grouped by parent index (`overlay_names[i]` corresponds to `names[i]`)            |
|      overlay_paths       |      Optional[List[List[str]]]       |                           Local overlay paths grouped by parent index (optional)                           |
|      overlay_links       |      Optional[List[List[str]]]       |                          Remote overlay links grouped by parent index (optional)                           |
|      overlay_hashes      |      Optional[List[List[str]]]       |                             Overlay hashes grouped by parent index (optional)                              |
|        batch_size        |            Optional[int]             |                          Number of items uploaded in one batch (for links/hashes)                          |
|   conflict_resolution    | Literal["rename", "skip", "replace"] |                                  Conflict resolution strategy (optional)                                   |
| force_metadata_for_links |            Optional[bool]            | Force metadata retrieval for images uploaded by links (if `False`, metadata can be temporarily incomplete) |

So, the method uploads parent images and linked overlay images to Supervisely and returns a tuple with parent `ImageInfo` list and grouped overlay `ImageInfo` lists.

### Important rules

- Exactly one source for parent images must be provided: `paths` or `links` or `hashes`.
- Exactly one source for overlays must be provided: `overlay_paths` or `overlay_links` or `overlay_hashes`.
- Parent and overlay lists must have consistent lengths.

## Example: upload from local paths

```python
import os
from dotenv import load_dotenv

import supervisely as sly

if sly.is_development():
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api.from_env()

workspace_id = 123
project = api.project.create(workspace_id, "Overlay demo", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, "ds0")

names = ["scene_01.jpg", "scene_02.jpg"]
paths = [
    "demo_data/img/scene_01.jpg",
    "demo_data/img/scene_02.jpg",
]

overlay_names = [
    ["scene_01_mask.png", "scene_01_heatmap.png"],
    ["scene_02_mask.png"],
]
overlay_paths = [
    ["demo_data/overlay/scene_01.jpg/scene_01_mask.png", "demo_data/overlay/scene_01.jpg/scene_01_heatmap.png"],
    ["demo_data/overlay/scene_02.jpg/scene_02_mask.png"],
]

parent_infos, overlay_infos_grouped = api.image.upload_overlay_images(
    dataset_id=dataset.id,
    names=names,
    paths=paths,
    overlay_names=overlay_names,
    overlay_paths=overlay_paths,
)
```

## Result in labeling interface

After upload, each parent image in the dataset has one or multiple linked overlays that can be displayed in the new Overlay labeling interface.
