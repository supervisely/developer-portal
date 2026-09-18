---
description: >-
  Use Command Line Interface for easy and convenient usage of supervisely functional right inside your console locally and with shell scripts on instance!
---

# Introduction

In this tutorial, you will discover how to simplify certain basic functions of Supervisely by automating them with easy-to-use Command Line Interface (CLI) commands.

{% hint style="info" %}

The list of commands currently available is not the final list and new commands may be added in the future. If you find that a certain functionality you need is not currently available, you can contact the developers and request that it be added.

{% endhint %}

# Prerequisites

To use CLI you first need to install latest Supervisely package on your preferred Linux system:

```bash
pip3 install --upgrade supervisely
```

After that, you will be able to use CLI. Learn more about SDK installation [here](../getting-started/installation.md)

# Interact with Projects using CLI

## Download a Project
```bash
supervisely project download -id <project-id> -d <local-destination>
```
In the following **required** arguments, replace:
- `<project-id>` with the ID of the Supervisely project you want to download. Prefixes: `-id`, `--id` 
- `<local-destination>` with the local directory where you want to save the project data. Prefixes: `-d`, `--dst` 

## Get project name
```bash
supervisely project get-name -id <project-id>
```
Replace: `<project-id>` with the ID of the Supervisely project you want to get name. Prefixes: `-id`, `--id` 

To export project name right in environmental variable, use the following trick in your shell script:
```shell
PROJECT_NAME=$(supervisely project get-name -id $PROJECT_ID)
```

## Upload a Project
```bash
supervisely project upload -s <source-local> -id <workspace-id> -n <project-name>
```
In the following **required** arguments, replace:
- `<local-source>` with the local directory where your project is stored. Prefixes: `-s`, `--src` 
- `<workspace-id>` with the ID of the target Supervisely workspace. Prefixes: `-id`, `--id` 

In the following **optional** arguments, replace: 
- `<project-name>` with the name of the project. By default, it takes the name of the source directory. Prefixes: `-n`, `--name` 

# Import Data using CLI

Use `supervisely import` to import a local dataset into an existing Supervisely project. The command runs the Auto Import application in Docker and automatically detects the annotation format.

Before running the command, make sure Docker is installed and your Supervisely credentials are available in `~/supervisely.env`:

```text
SERVER_ADDRESS=<server-address>
API_TOKEN=<api-token>
```

## Import a local directory

```bash
supervisely import <local-source> --project-id <project-id>
```

In the following **required** arguments, replace:
- `<local-source>` with the local directory or file you want to import.
- `<project-id>` with the ID of the destination Supervisely project. Prefix: `--project-id`

In the following **optional** arguments, replace:
- `<dataset-id>` with the ID of an existing dataset in the destination project. Prefix: `--dataset-id`
- `<dataset-name>` with the name of the dataset that will be created if `--dataset-id` is not provided. Prefix: `--dataset-name`
- `<docker-image>` with a custom Auto Import CLI Docker image. Prefix: `--image`
- `<env-file>` with a custom path to the Supervisely credentials file. By default, `~/supervisely.env` is used. Prefix: `--env-file`
- Add the `--import-as-links` flag to import supported link-based datasets without uploading binary files.
- Add the `--dry-run` flag to print the Docker command without running the import.

For example:

```bash
supervisely import ./dataset --project-id 6911 --dataset-name "my dataset"
```

The source path is mounted into the Docker container as read-only. If you pass a directory, it is mounted as `/input`. If you pass a file, its parent directory is mounted and the file is passed to Auto Import inside `/input`.

For large archives or project structures with many files, make sure Docker has enough temporary disk space. Auto Import may unpack archives and prepare temporary files before uploading data. If the container runs out of disk space, run the same import on a machine with a larger Docker storage volume.

{% hint style="info" %}
`--import-as-links` is intended for link-based formats, for example CSV or TXT files with URLs. It is not a replacement for importing arbitrary local image files without uploading them.
{% endhint %}

## Advanced mode

For most imports, use the short command shown above. If you need to debug the Docker command, use a custom Auto Import image, or provide a separate credentials file, use the optional arguments:

```bash
supervisely import ./dataset \
  --project-id 6911 \
  --dataset-name "my dataset" \
  --env-file ~/supervisely.env \
  --image supervisely/main-import-cli:latest
```

Add `--dry-run` to print the `docker run` command without starting the import:

```bash
supervisely import ./dataset --project-id 6911 --dry-run
```

This is useful when you want to inspect the mounted paths and environment variables before running a large import.

### Use a custom Docker work directory

By default, Auto Import uses a temporary directory inside the container for staging data. This is convenient and does not require extra arguments, but large archives may require more Docker disk space.

If you need to control where temporary data is stored, run the Docker command manually and mount a work directory:

```bash
mkdir -p .sly-import-work

docker run --rm \
  --env-file ~/supervisely.env \
  -e PROJECT_ID=6911 \
  -e DATASET_NAME="my dataset" \
  -e SLY_APP_DATA_DIR=/work \
  -v "$PWD/dataset:/input:ro" \
  -v "$PWD/.sly-import-work:/work" \
  supervisely/main-import-cli:latest \
  --input /input \
  --work-dir /work
```

Keep the source dataset mounted as read-only (`/input:ro`). Auto Import may unpack archives, remove temporary junk files, and create intermediate files while preparing the upload, so all writable data should go to the work directory.

For a single archive or file, mount the parent directory and pass the file path inside `/input`:

```bash
docker run --rm \
  --env-file ~/supervisely.env \
  -e PROJECT_ID=6911 \
  -e SLY_APP_DATA_DIR=/work \
  -v "$PWD:/input:ro" \
  -v "$PWD/.sly-import-work:/work" \
  supervisely/main-import-cli:latest \
  --input /input/dataset.zip \
  --work-dir /work
```

# Interact with Team files using CLI

## Download directory from Team files
```bash
supervisely teamfiles download -id <team-id> -s <remote-source> -d <local-destination> -f "<filter-text>" -i
```
In the following **required** arguments, replace:  
- `<team-id>` with the ID of the actual team. Prefixes: `-id`, `--id`
- `<remote-source>` with the local directory where the files are located. Prefixes: `-s`, `--src`
- `<local-destination>` with the remote directory in Team files where you want to upload the files. Prefixes: `-d`, `--dst`

In the following **optional** arguments, replace:  
- `"<filter-text>"` with the regular expression (f.e. `".jpg$"`) which will filter files in directory. Then, only filtered files will be downloaded.  Prefixes: `-f`, `--filter`
- Add the `-i` flag to ignore and skip if source directory not exists.


## Upload directory to Team files
```bash
supervisely teamfiles upload -id <team-id> -s <local-source> -d <remote-destination>
```
In the following **required** arguments, replace:  
- `<team-id>` with the ID of the actual team. Prefixes: `-id`, `--id`
- `<local-source>` with the local directory where the files are located. Prefixes: `-s`, `--src`
- `<remote-destination>` with the remote directory in Team files where you want to upload the files. Prefixes: `-d`, `--dst`

Note: to set link to Team files directory at workspace tasks interface, use [following command](#set-link-to-a-team-files-directory)

## Remove directory from Team files
```bash
supervisely teamfiles remove-directory -id <team-id> -p <remote-path>
```
In the following **required** arguments, replace:  
- `<team-id>` with the ID of the team. Prefixes: `-id`, `--id`
- `<remote-path>` with the path to the folder in Team files. Prefixes: `-p`, `--path`

## Remove file from Team files
```bash
supervisely teamfiles remove-file -id <team-id> -p <remote-path>
```
In the following **required** arguments, replace:  
- `<team-id>` with the ID of the team. Prefixes: `-id`, `--id`
- `<remote-path>` with the path to the file in Team files. Prefixes: `-p`, `--path`

# Interact with Workspace tasks using CLI

## Set link to a Team files directory
```bash
supervisely task set-output-dir -d <output-path>
```
Replace `<output-path>` with the path to the output directory. Prefixes: `-d`, `--dir`

# Release your Private Apps using CLI

See the full [tutorial](../app-development/basics/add-private-app.md) on how to add Private Apps using CLI.

Here, we will describe components of following command which releases a private app:

```bash
supervisely release -p <app-directory> -a <sub-app-directory> --release-version <version> --release-description <description> -s <slug> -y
```
In the following **optional** arguments, replace: 
- `<app-directory>` with the path to the directory containing the application. By default, it's a current working directory. Prefixes: `-p`, `--path`
- `<sub-app-directory>` with the path to the sub-app relative to the application directory. By default, it's a current working directory. Prefixes: `-a`, `--sub-app`
- `<version>` with the version number in the format "vX.X.X". By default, there will be a small increment "0.0.1". Prefix: `--release-version`
- `<description>` with the release description (max length is 64 symbols). You will be asked to enter description. Prefix: `--release-description`
- `<slug>` with the slug for internal use. A term "slug" stands for a short label or ID that is used to identify a specific item or resource (ann app in our case). Prefixes: `-s`, `--slug`
- Add the `-y` flag to auto-confirm the release.

# Advanced

To install your own modification or specific version of Supervisely, follow these steps:

## **Create file `requirements.txt`**

Create file `requirements.txt` with necessary dependency:

```text
supervisely==<version in format X.X.X> # specific version
# path/to/sdk/supervisely  # alternative (only local debug)
# git+https://github.com/<your_name>/<your_supervisely_fork>.git@<your_branch> # alternative
```
## **Make shell script `create_venv.sh`**

Make shell script `create_venv.sh` with instructions on virtual environment installation.

<details>

<summary>create_venv.sh</summary>

```shell
#!/bin/bash

# learn more in documentation
# Official python docs: https://docs.python.org/3/library/venv.html
# Superviely developer portal: https://developer.supervisely.com/getting-started/installation#venv

if [ -d ".venv" ]; then
    echo "VENV already exists, will be removed"
    rm -rf .venv
fi

echo "VENV will be created" && \
python3 -m venv .venv && \
source .venv/bin/activate && \

echo "Install requirements..." && \
pip3 install -r requirements.txt && \
echo "Requirements have been successfully installed" && \
echo "Testing imports, please wait a minute ..." && \
python -c "import supervisely as sly" && \
echo "Success!" && \
deactivate
```

</details>



## **Activate virtual environment**

Run script and choose directory with virtual environment. Then, activate your environment (you will see `(.venv)` appeared in your console):

```bash
cd your/directory/with/script/
./create_venv.sh
source .venv/bin/activate
```

{% hint style="info" %}

To use CLI on instance (for example, in your personal shell script), you need to include `requirements.txt` with Supervisely in your Private App repository

{% endhint %}
