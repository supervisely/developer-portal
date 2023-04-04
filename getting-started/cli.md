---
description: >-
  Use Command Line Interface for easy and convenient usage of supervisely functional right inside your console locally and with shell scripts on instance!
---

# Introduction

In this tutorial, you will discover how to simplify certain basic functions of supervisely by automating them with easy-to-use Command Line Interface (CLI) commands.

{% hint style="info" %}

The list of commands currently available is not the final list and new commands may be added in the future. If you find that a certain functionality you need is not currently available, you can contact the developers and request that it be added.

{% endhint %}

# Prerequisites

To use CLI you first need to install latest supervisely package on your prefered Linux system:

```bash
pip3 install --upgrade supervisely
```

After that, you will be able to use CLI. Learn more about sdk installation [here](../getting-started/installation.md)

# Interact with Projects using CLI

## Download a Project
```bash
supervisely project download --id <project-id> --dst <local-destination>
```
In the following **required** arguments, replace:
- `<project-id>` with the ID of the supervisely project you want to download
- `<local-destination>` with the local directory where you want to save the project data

## Get project name
```bash
supervisely project get-name -id <project-id>
```
In the following **required** arguments, replace:  
- `<project-id>` with the ID of the supervisely project you want to get name

💡To export project name right in environmental variable, use the following trick in your shell script:
```shell
PROJECT_NAME=$(supervisely project get-name -id $PROJECT_ID)
```

# Interact with Team files using CLI

## Upload directory to Team files
```bash
supervisely teamfiles upload --id <team-id> --src <local-source> --dst <remote-destination>
```
In the following **required** arguments, replace:  
- `<team-id>` with the ID of the team you want to upload the files to
- `<local-source>` with the local directory where the files are located
- `<remote-destination>` with the remote directory in Team files where you want to upload the files

Note: to set link to Team files directory at workspace tasks interface, use [following command](#set-link-to-a-team-files-directory)

## Remove directory from Team files
```bash
supervisely teamfiles remove-directory --id <team-id> --path <remote-path>
```
Replace `<team-id>` with the ID of the team and `<remote-path>` with the path to the folder in Team files.

## Remove file from Team files
```bash
supervisely teamfiles remove-file --id <team-id> --path <remote-path>
```
Replace `<team-id>` with the ID of the team and `<remote-path>` with the path to the file in Team files.

# Interact with Workspace tasks using CLI

## Set link to a Team files directory
```bash
supervisely task set-output-dir --output-dir <output-path>
```
Replace `<output-path>` with the path to the output directory

# Release your Private Apps using CLI

See the full [tutorial](../app-development/basics/add-private-app.md) on how to add Private Apps using CLI.

Here, we will describe components of following command which releases a private app:

```bash
supervisely release --path <app-directory> --sub-app <sub-app-directory> --release-version <version> --release-description <description> --slug <slug> -y
```
In the following **optional** arguments, replace: 
- `<app-directory>` with the path to the directory containing the application. By default, it's a current working directory.
- `<sub-app-directory>` with the path to the sub-app relative to the application directory. By default, it's a current working directory.
- `<version>` with the version number in the format "vX.X.X". By default there will be a small increment "0.0.1" 
- `<description>` with the release description (max length is 64 symbols). You will be asked to enter description.
- `<slug>` with the slug for internal use. A term "slug" stands for a short label or ID that is used to identify a specific item or resource (ann app in our case).
- Add the `-y` flag to auto-confirm the release.

# Advanced

To install your own modification or specific version of supervisely, follow this steps:

## **Step 1.**

Make shell script `create_venv.sh` with instructions on virtual environment installation.

<details>

<summary>create_venv.sh</summary>

```shell
#!/bin/bash

# learn more in documentation
# Official python docs: https://docs.python.org/3/library/venv.html
# Superviely developer portal: https://developer.supervise.ly/getting-started/installation#venv

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

## **Step 2.**

Create text file `requirements.txt` with necessary dependecy:

```text
supervisely==<version in format X.X.X> # specific version
# path/to/sdk/supervisely  # alternative (only local debug)
# git+https://github.com/<your_name>/<your_supervisely_fork>.git@<your_branch> # alternative
```

## **Step 3.**

Run script and enter virtual environment. Then, activate your environment (you will see `(.venv)` appeared in your console):

```bash
cd your/directory/with/script/
./create_venv.sh
source .venv/bin/activate
```

{% hint style="info" %}

To use CLI on instance (for example, in your personal shell script), you need to include `requirements.txt` with supervisely in your Private App repository

{% endhint %}
