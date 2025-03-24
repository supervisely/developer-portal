# Iterate over a project

In this article, we will learn how to iterate through a project with annotated data in python. It is one of the most frequent operations in Superviely Apps and python automation scripts.

# Dataset Types

In Supervisely, datasets can be organized in two ways: flat or nested. Understanding these structures can help you with efficient data organization and management.

## Flat Dataset Structure

A flat dataset is the simplest form of organization where all images and their annotations are stored in a single level. This structure is good for simple projects with straightforward organization.

You can add this dataset to your team via Supervisely Ecosystem - ⬇️[Lemons (Annotated)]()

**Example structure:**
```text
📦 Lemons (Annotated)           ← The project
 ┣ 📂 ds1                       ← The dataset
 ┃ ┣ 📂 ann                     ← Folder for annotations
 ┃ ┃ ┣ 📜 IMG_0748.jpeg.json    ← Annotation for image 0748
 ┃ ┃ ┣ 📜 IMG_1836.jpeg.json    ← Annotation for image 1836
 ┃ ┃ ┣ 📜 IMG_2084.jpeg.json
 ┃ ┃ ┣ 📜 IMG_3861.jpeg.json
 ┃ ┃ ┣ 📜 IMG_4451.jpeg.json
 ┃ ┃ ┗ 📜 IMG_8144.jpeg.json
 ┃ ┣ 📂 img                     ← Folder for images
 ┃ ┃ ┣ 🖼️ IMG_0748.jpeg         ← Image 0748
 ┃ ┃ ┣ 🖼️ IMG_1836.jpeg         ← Image 1836
 ┃ ┃ ┣ 🖼️ IMG_2084.jpeg
 ┃ ┃ ┣ 🖼️ IMG_3861.jpeg
 ┃ ┃ ┣ 🖼️ IMG_4451.jpeg
 ┃ ┃ ┗ 🖼️ IMG_8144.jpeg
 ┣ 📜 meta.json                 ← Project metadata
 ┗ 📜 README.md                 ← Optional readme file
```

## Nested Dataset Structure

A nested dataset structure is a bit more advanced. It lets you create datasets inside other datasets, forming a hierarchy—like tree for your data. Nested datasets are good for complex projects requiring hierarchical organization or when you need to group related data together.

You can add this dataset to your team via Supervisely Ecosystem - ⬇️[Fruits (Annotated)]()

{% hint style="info" %}

**Important Note about Nested Datasets:**

When working with nested datasets, keep in mind:
- Parent datasets (like "Temperate" or "Tropical") can be empty or non-empty themselves, but contain images inside nested datasets
- To get all parent dataset images including nested ones, you'll need to iterate through each nested dataset

{% endhint %}

**Example structure:**

- The main datasets ("Temperate" and "Tropical") don't hold images or annotations directly in ann and img folders.
- Instead, they have a datasets folder containing nested datasets (like "Apple", "Banana", etc.), and those hold the images and annotations.
- The main datasets can also contain images, but we removed them for this example

```text
📦 Fruits (Annotated)          ← The project
 ┣ 📂 Temperate                ← Main dataset #1
 ┃ ┣ 📂 ann                    ← Empty (no annotations here)
 ┃ ┣ 📂 img                    ← Empty (no images here)
 ┃ ┣ 📂 datasets               ← Where the nested datasets live
 ┃ ┃ ┣ 📂 Apple                ← Nested dataset for apples
 ┃ ┃ ┃ ┣ 📂 ann
 ┃ ┃ ┃ ┃ ┣ 📜 apple_1.jpg.json
 ┃ ┃ ┃ ┃ ┣ 📜 apple_2.jpg.json
 ┃ ┃ ┃ ┃ ┗ 📜 apple_3.jpg.json
 ┃ ┃ ┃ ┗ 📂 img
 ┃ ┃ ┃ ┃ ┣ 🖼️ apple_1.jpg
 ┃ ┃ ┃ ┃ ┣ 🖼️ apple_2.jpg
 ┃ ┃ ┃ ┃ ┗ 🖼️ apple_3.jpg
 ┃ ┃ ┗ 📂 Pear                  ← Nested dataset for pears
 ┃ ┃ ┃ ┣ 📂 ann
 ┃ ┃ ┃ ┃ ┣ 📜 pear_1.jpg.json
 ┃ ┃ ┃ ┃ ┣ 📜 pear_2.jpg.json
 ┃ ┃ ┃ ┃ ┗ 📜 pear_3.jpg.json
 ┃ ┃ ┃ ┗ 📂 img
 ┃ ┃ ┃ ┃ ┣ 🖼️ pear_1.jpg
 ┃ ┃ ┃ ┃ ┣ 🖼️ pear_2.jpg
 ┃ ┃ ┃ ┃ ┗ 🖼️ pear_3.jpg
 ┣ 📂 Tropical                 ← Main dataset #2
 ┃ ┣ 📂 ann                    ← Empty (no annotations here)
 ┃ ┣ 📂 img                    ← Empty (no images here)
 ┃ ┣ 📂 datasets               ← Where the nested datasets live
 ┃ ┃ ┣ 📂 Banana               ← Nested dataset for bananas
 ┃ ┃ ┃ ┣ 📂 ann
 ┃ ┃ ┃ ┃ ┣ 📜 banana_1.jpg.json
 ┃ ┃ ┃ ┃ ┣ 📜 banana_2.jpg.json
 ┃ ┃ ┃ ┃ ┗ 📜 banana_3.jpg.json
 ┃ ┃ ┃ ┗ 📂 img
 ┃ ┃ ┃ ┃ ┣ 🖼️ banana_1.jpg
 ┃ ┃ ┃ ┃ ┣ 🖼️ banana_2.jpg
 ┃ ┃ ┃ ┃ ┗ 🖼️ banana_3.jpg
 ┃ ┃ ┣ 📂 Lemon                ← Nested dataset for lemons
 ┃ ┃ ┃ ┣ 📂 ann
 ┃ ┃ ┃ ┃ ┣ 📜 lemon_1.jpg.json
 ┃ ┃ ┃ ┃ ┣ 📜 lemon_2.jpg.json
 ┃ ┃ ┃ ┃ ┗ 📜 lemon_3.jpg.json
 ┃ ┃ ┃ ┗ 📂 img
 ┃ ┃ ┃ ┃ ┣ 🖼️ lemon_1.jpg
 ┃ ┃ ┃ ┃ ┣ 🖼️ lemon_2.jpg
 ┃ ┃ ┃ ┃ ┗ 🖼️ lemon_3.jpg
 ┃ ┃ ┗ 📂 Mango                ← Nested dataset for mangoes
 ┃ ┃ ┃ ┣ 📂 ann
 ┃ ┃ ┃ ┃ ┣ 📜 mango_1.jpg.json
 ┃ ┃ ┃ ┃ ┣ 📜 mango_2.jpg.json
 ┃ ┃ ┃ ┃ ┗ 📜 mango_3.jpg.json
 ┃ ┃ ┃ ┗ 📂 img
 ┃ ┃ ┃ ┃ ┣ 🖼️ mango_1.jpg
 ┃ ┃ ┃ ┃ ┣ 🖼️ mango_2.jpg
 ┃ ┃ ┃ ┃ ┗ 🖼️ mango_3.jpg
 ┣ 📜 meta.json                ← Project metadata
 ┗ 📜 README.md                ← Optional readme file
```


{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/iterate-over-project): source code, Visual Studio code configuration, and a shell script for creating venv.
{% endhint %}

In this guide we will go through the following steps:

\*\*\*\* [**Step 1.**](iterate-over-a-project.md#1.-demo-project) Get a [demo project](https://ecosystem.supervisely.com/projects/lemons-annotated) with labeled lemons and kiwis.

\*\*\*\* [**Step 2.**](iterate-over-a-project.md#2.-.env-files) Prepare `.env` files with credentials and ID of a demo project.

\*\*\*\* [**Step 3.**](iterate-over-a-project.md#3.-python-script) Run [python script](https://github.com/supervisely-ecosystem/iterate-over-project/blob/master/main.py).

\*\*\*\* [**Step 4.**](iterate-over-a-project.md#4.-optimizations) Show possible optimizations.

### 1. Demo project

If you don't have any projects yet, go to the ecosystem and add the demo project 🍋 **`Lemons (Annotated)`**  or 🍍 **Fruits (Annotated)** to your current workspace.

![Add demo project "Lemons (Annotated)" to your workjspace](https://user-images.githubusercontent.com/12828725/180640631-8636ac88-a8f7-4f72-90bb-84438d12f247.png)

### 2. `.env` files

Create a file at `~/supervisely.env` with the credentials for your Supervisely account. Learn more about environment variables [here](../../environment-variables.md). The content should look like this:

```python
# your API credentials, learn more here: https://developer.supervisely.com/getting-started/basics-of-authentication
SERVER_ADDRESS="https://app.supervisely.com" # ⬅️ change it if use Enterprise Edition
API_TOKEN="4r47N.....blablabla......xaTatb" # ⬅️ change it
```

Create the second file `local.env` and place it in the same directory with the `main.py`. This file will contain values we are going to use in the python script.

```python
# change the Project ID to your value
PROJECT_ID=12208 # ⬅️ change it
```

### 3. Python script

{% hint style="info" %}
This script illustrates only the basics. If your project is huge and has **hundreds of thousands of images** then it is not so efficient to download annotations one by one. It is better to use batch (bulk) methods to reduce the number of API requests and significantly speed up your code. Learn more in [the optimizations section](iterate-over-a-project.md#optimizations) below.
{% endhint %}

To start debugging you need to

1. Clone the [repo](https://github.com/supervisely-ecosystem/iterate-over-project)
2. Create [venv](https://docs.python.org/3/library/venv.html) by running the script [`create_venv.sh`](https://github.com/supervisely-ecosystem/iterate-over-project/blob/master/create\_venv.sh)
3. Change value in [local.env](https://github.com/supervisely-ecosystem/iterate-over-project/blob/master/local.env)
4. Check that you have `~/supervisely.env` file with correct values

#### Source code:

```python
import os
import supervisely as sly
from dotenv import load_dotenv

if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api.from_env()

project_id = sly.env.project_id()
project = api.project.get_info_by_id(project_id)
if project is None:
    raise KeyError(f"Project with ID {project_id} not found in your account")
print(f"Project info: {project.name} (id={project.id})")

# get project meta - collection of annotation classes and tags
meta_json = api.project.get_meta(project.id)
project_meta = sly.ProjectMeta.from_json(meta_json)
print(project_meta)

# Set recursive to True if you want to include nested datasets
datasets = api.dataset.get_list(project.id, recursive=True) 
print(f"There are {len(datasets)} datasets in project")

for dataset in datasets:
    print(f"Dataset {dataset.name} has {dataset.items_count} images")
    images = api.image.get_list(dataset.id)
    for image in images:
        ann_json = api.annotation.download_json(image.id)
        ann = sly.Annotation.from_json(ann_json, project_meta)
        print(f"There are {len(ann.labels)} objects on image {image.name}")
```

#### Output

The script above produces the following output:

```
Project info: Lemons (Annotated) (id=12208)
ProjectMeta:
Object Classes
+-------+--------+----------------+--------+
|  Name | Shape  |     Color      | Hotkey |
+-------+--------+----------------+--------+
|  kiwi | Bitmap |  [255, 0, 0]   |        |
| lemon | Bitmap | [81, 198, 170] |        |
+-------+--------+----------------+--------+
Tags
+------+------------+-----------------+--------+---------------+--------------------+
| Name | Value type | Possible values | Hotkey | Applicable to | Applicable classes |
+------+------------+-----------------+--------+---------------+--------------------+
+------+------------+-----------------+--------+---------------+--------------------+

There are 1 datasets in project
Dataset ds1 has 6 images
There are 3 objects on image IMG_1836.jpeg
There are 4 objects on image IMG_8144.jpeg
There are 4 objects on image IMG_3861.jpeg
There are 3 objects on image IMG_0748.jpeg
There are 5 objects on image IMG_4451.jpeg
There are 7 objects on image IMG_2084.jpeg
```

### 4. Optimizations

The bottleneck of this script is in these lines (27-28):

```python
for image in images:
    ann_json = api.annotation.download_json(image.id)
```

If you have **1M** images in your project, your code will send 🟡 **1M** requests to download annotations. It is inefficient due to Round Trip Time (RTT) and a large number of similar tiny requests to a Supervisely database.

It can be optimized by using the batch API method:

```python
api.annotation.download_json_batch(dataset.id, image_ids) 
```

Supervisely API allows downloading annotations for multiple images in a single request. The code sample below sends ✅ **50x fewer** requests and it leads to a significant speed-up of our original code:

```python
for batch in sly.batched(images):
    image_ids = [image.id for image in batch]
    annotations = api.annotation.download_json_batch(dataset.id, image_ids)
    for image, ann_json in zip(batch, annotations):
        ann = sly.Annotation.from_json(ann_json, project_meta)
        print(f"There are {len(ann.labels)} objects on image {image.name}")
```

The optimized version of the original script is in [`main_optimized.py`](https://github.com/supervisely-ecosystem/iterate-over-project/blob/master/main\_optimized.py).
