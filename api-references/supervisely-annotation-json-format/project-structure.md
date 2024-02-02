# Project Structure
In Supervisely all data and annotations are stored inside individual projects which themselves consist of datasets with files in them, and Project Meta file - a series of classes and tags. In Supervisely Project Meta is a very important part of the project, as it defines the classes and tags that are used to annotate the data. Read more about Project Meta in the [Project Meta](!!!PASTE URL HERE!!!) article.<br>
When downloaded, each project is converted into a folder that stores `meta.json` file containing Project Meta, and dataset folders with the individual annotation files in them. This allows you to seamlessly cycle data between Supervisely and local storage and upload it back to Supervisely if needed.<br>
Let's take a closer look at the structure of Supervisely project. Each project consists of the following elements:

![project structure system](../../.gitbook/assets/project\_structure.png)

**Project Folder**

On the top level, we have Project folders, these are the elements visible on the main Supervisely dashboard. Inside them they can contain only Datasets and Project Meta information, all other data has to be stored a level below in a Dataset. All datasets within a project have to contain content of the same category.

**Project Meta**

Project Meta contains the essential information about the project - Classes and Tags. These are defined project-wide and can be used for labeling every dataset inside the current project.

**Datasets**

Datasets are the second-level folders inside the project, they host the individual data files and their annotations.

**Items**

Every data file in the project has to be stored inside a dataset. Each file has its own set of annotations.

## Downloaded Project Structure

All projects downloaded from Supervisely maintain the same basic structure, with the contents varying based on which download option you choose.

**Download Archive**

When you select one of the download options, the system automatically creates an archive with the following name structure: `project_name.tar`

**Downloaded Project**

All projects downloaded from Supervisely have the following structure:

![project structure system](../../.gitbook/assets/project\_structure.png)

The root folder for the project named `project name`

* `meta.json` file&#x20;
* `obj_class_to_machine_color.json` file (optional, for image annotation projects)
* `key_id_map.json` file (optional)
* Dataset folders, each named `dataset_name`, which contains:
  * `ann` folder,  contains annotation files, each named `source_media_file_name.json` for the corresponding file
  * `img` (`video` or `pointcloud`) folder, contains source media
  * `meta` optional folder contains corresponding JSON files with metadata for images
  * `masks_human` optional folder for image annotation projects contains .png files with annotations marked on them
  * `masks_machine` optional folder for image annotation projects contains .png files with machine annotations

### Project structure example

The following structure is an example of a project with 3 datasets, each containing 3 images with annotations, and also a meta directory with metadata for each image.

```text
📦 project-name
 ┣ 📂 dataset-name-001
 ┃ ┣ 📂 ann
 ┃ ┃ ┣ 📄 pexels-photo-101063.png.json
 ┃ ┃ ┣ 📄 pexels-photo-103123.png.json
 ┃ ┃ ┗ 📄 pexels-photo-103127.png.json
 ┃ ┣ 📂 img
 ┃ ┃ ┣ 🏞️ pexels-photo-101063.png
 ┃ ┃ ┣ 🏞️ pexels-photo-103123.png
 ┃ ┃ ┗ 🏞️ pexels-photo-103127.png
 ┃ ┗ 📂 meta
 ┃ ┃ ┣ 📄 pexels-photo-101063.png.json
 ┃ ┃ ┣ 📄 pexels-photo-103123.png.json
 ┃ ┃ ┗ 📄 pexels-photo-103127.png.json
 ┣ 📂 dataset-name-002
 ┃ ┣ 📂 ann
 ┃ ┃ ┣ 📄 pexels-photo-100583.png.json
 ┃ ┃ ┣ 📄 pexels-photo-105472.png.json
 ┃ ┃ ┗ 📄 pexels-photo-106118.png.json
 ┃ ┣ 📂 img
 ┃ ┃ ┣ 🏞️ pexels-photo-100583.png
 ┃ ┃ ┣ 🏞️ pexels-photo-105472.png
 ┃ ┃ ┗ 🏞️ pexels-photo-106118.png
 ┃ ┗ 📂 meta
 ┃ ┃ ┣ 📄 pexels-photo-100583.png.json
 ┃ ┃ ┣ 📄 pexels-photo-105472.png.json
 ┃ ┃ ┗ 📄 pexels-photo-106118.png.json
 ┣ 📂 dataset-name-003
 ┃ ┣ 📂 ann
 ┃ ┃ ┣ 📄 pexels-photo-101647.png.json
 ┃ ┃ ┣ 📄 pexels-photo-103681.png.json
 ┃ ┃ ┗ 📄 pexels-photo-104328.png.json
 ┃ ┣ 📂 img
 ┃ ┃ ┣ 🏞️ pexels-photo-101647.png
 ┃ ┃ ┣ 🏞️ pexels-photo-103681.png
 ┃ ┃ ┗ 🏞️ pexels-photo-104328.png
 ┃ ┗ 📂 meta
 ┃ ┃ ┣ 📄 pexels-photo-101647.png.json
 ┃ ┃ ┣ 📄 pexels-photo-103681.png.json
 ┃ ┃ ┗ 📄 pexels-photo-104328.png.json
 ┗ 📄  meta.json
```

## Working with projects

<details>
<summary>Using the Python SDK</summary>

As we said before, it's possible to work with Supervisely projects both on the platform and locally. You can perform a lot of tasks locally even when you're offline, and then upload the results back to the platform if needed, most of the tools in Python SDK work for local projects as well. But for convenience, we'll split this article into two parts: working with projects on the platform and working with projects locally.

#### Working with projects on the platform
Let's imagine that we already have a project on the platform, we want to download it locally. Check this detailed tutorial on how to [Iterate over project](!!!PASTE URL HERE!!!) to learn more about projects on the platform.

```sh
pip install supervisely
```

```python
import supervisely as sly

...

# If we don't know the project ID, we can get it by name:
project_id = api.project.get_info_by_name("my_project").id

# Download the project to the specified directory:
sly.Project.download(api, project_id, "/path/to/project")
```

And that's it! Now you can work with the project locally.

#### Working with projects locally
Let's imagine that we've already prepared a folder with a correct file structure in Supervisely JSON format and now we want to read it and upload it to the platform. Check this detailed tutorial on how to [Iterate over local project](!!!PASTE URL HERE!!!) to learn more about local projects.

```sh
pip install supervisely
```

```python
import supervisely as sly

...

project = sly.Project("/path/to/project", sly.OpenMode.READ)

# Do something with the project...

sly.Project.upload("/path/to/project", api, workspace_id)
```

So we've read the project, did something with it, and uploaded it back to the platform.
</details>
<br>


<details>
<summary>Using CLI</summary>
Remember that Supervisely CLI is a command-line tool, so you need to run it from the terminal and install Supervisely Python SDK first.<br>
Let's try to download and upload a project using Supervisely CLI.

```bash
pip install supervisely
```

After that, you will need to specify the project ID and can download the project to the local directory:

```bash
supervisely project download -id 12345 -d /path/to/destination/dir
```
</details>

And now, when we do something with the project locally, we can upload it back to the platform:

!!!! Consider changing -id to -wid or something, since it's workspace ID.
```bash
supervisely -s /path/to/source/dir -id 12345 -n my_project
```