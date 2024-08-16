# App Compatibility

## Introduction

When releasing a public app for Supervisely Ecosystem, you must ensure that your app is compatible with the version of Supervisely instance, which is provided in the [`config.json`](https://developer.supervisely.com/app-development/basics/app-json-config/config.json#instance_version) file. Otherwise, it may lead to an app doesn't work properly.<br>
ℹ️ If `instance_version` parameter is not specified in the `config.json` file, the release action will be blocked. So this parameter is mandatory.

## Upgrading the Supervisely Python SDK
If you releasing a new version of the Python SDK, which requires some features that are available only in specific versions of Supervisely, for example, you're adding a completely new feature or using new API endpoints, you must update the [`versions.json`](https://github.com/supervisely/supervisely/blob/master/versions.json) file in the SDK repo and the table in Documentation [here](https://developer.supervisely.com/getting-started/installation#compatibility-table).<br>
ℹ️ If you just creating a new release of the app, you don't need to update the compatibility files. They should be updated only when you release a new version of the SDK. So in this case, you can skip this section and jump to the [Creating a new release of the app](#creating-a-new-release-of-the-app) section.

### versions.json file
This file is used by release actions to check if the provided version of the SDK is compatible with the Supervisely instance. The file contains a dictionary with the following structure:

```json
{
  "6.9.11": "6.72.70",
  "6.9.13": "6.73.76",
  "6.9.18": "6.73.81",
  "6.9.22": "6.73.90",
  "6.9.31": "6.73.123",
  "6.10.0": "6.73.126"
}
```

Let's take a closer look at the example above. The key is the version of the Supervisely instance, and the value is the minimum version of the SDK that is compatible with the Supervisely instance. Here are some examples:

- instance version `6.9.10` is compatible with all Python SDK versions, `lower than 6.72.70`
- instance version `6.9.12` (which is not present in the file) is compatible with Python SDK versions `lower than 6.73.76` (the next value in the file) and `higher than 6.72.70` (the previous value in the file).
- instance version `6.9.13` is compatible with Python SDK versions `higher or equal to 6.73.76` and `lower than 6.73.81`. Please, pay attention to the `lower or equal` condition, which is different from the previous example.
- instance version `6.10.1` is compatible with all Python SDK versions `higher or equal to 6.73.126`

It may look a bit complicated, but in fact, it becomes very simple when you need to update the file. For example, imagine that you released a new version of the Supervisely Python SDK (e.g. `6.74.0`) and the feature that is required by this release was added in the Supervisely instance version `6.11.0`. So, you simply add this key-value pair to the file and that's it:

```json
{
  "6.9.11": "6.72.70",
  "6.9.13": "6.73.76",
  "6.9.18": "6.73.81",
  "6.9.22": "6.73.90",
  "6.9.31": "6.73.123",
  "6.11.0": "6.74.0"
}
```

Now, you can be sure that any newer release of the SDK will be compatible with the Supervisely instance version `6.11.0` (until new entry won't be added to the file) while older versions will not.<br>
ℹ️ Please, always add new entries the way, that the file remains sorted by the key in ascending order. It will be much easier for developers to find the required version. In the example above, the new entry should be added to the end of the file.

### Compatibility Table

This works in the same way as the `versions.json` file, but it's used in the Documentation and contains more user-friendly formatting. The table is located [here](https://developer.supervisely.com/getting-started/installation#compatibility-table). It's a simple table with the following structure:

|           Instance version          |        Python SDK version         |
| :---------------------------------: | :-------------------------------: |
|         >=6.10.0                    |       supervisely>=6.73.126       |
|         <=6.9.31                    |       supervisely<=6.73.123       |
|         <=6.9.22                    |       supervisely<=6.73.90        |
|         <=6.9.18                    |       supervisely<=6.73.81        |
|         <=6.9.13                    |       supervisely<=6.73.76        |
|         <=6.9.11                    |       supervisely<=6.72.70        |

As you can see, the order of the instance version is reversed compared to the `versions.json` file. It's done to make it easier for developers to find the required version, so you always see newer versions at the top of the table. And also it contains signs `>=` and `<=` to make it clear which versions are supported. They are used in the same way as in the `versions.json` file.

## Creating a new release of the app
Before creating a new release, please ensure that the compatibility files are up-to-date. After that, for a successful release of the app, the following conditions must be met:
- the repository must not contain any `requirements.txt` files in the directories with the corresponding `config.json` files
- the `config.json` file must contain the `instance_version` parameter
- the `config.json` file must contain the `docker_image` parameter
- the `instance_version` parameter must contain a version that is compatible with the Python SDK version, that is used in the docker image

ℹ️ Those conditions are applied TO ALL apps in the repository, so if at least one app doesn't meet the requirements, the release action will be blocked for the whole repository. If the repo contains multiple apps, you must update it for all of them.

Let's talk about each of these conditions in more detail.

### requirements.txt files
It's simple: they're prohibited. The application must use pre-built docker images, which contain all the necessary dependencies. If you need to install additional packages, you can do it in the Dockerfile. If you need to create a release for an app, that contains a `requirements.txt` file, you will need to put everything in the Dockerfile and remove the `requirements.txt` file. Otherwise, the release action will be blocked.

### instance_version and docker_image parameters
These parameters are mandatory. The `instance_version` parameter must contain the version of the Supervisely instance, which is required for the app to work properly. The `docker_image` parameter must contain the name of the docker image and its tag, which will be used to run the app. The docker image must contain the Python SDK version, which is compatible with the Supervisely instance version. If any of these parameters are missing, the release action will be blocked.

### instance_version and Python SDK version compatibility
This one is the trickiest since there are several different check cases and we'll cover them all.

#### Case 1: Standard docker images
So, many of the apps in the Supervisely Ecosystem use "default" docker images, which are built automatically on each release of Supervisely Python SDK and all of them contain the Python SDK version in the tag. For example:

```json
{
    "docker_image": "supervisely/labeling:6.72.70",
    "instance_version": "6.9.12"
}
```
You can find the list of standard docker images [here](https://github.com/supervisely/supervisely/tree/master/docker_images).<br>
So in this case, the tag of the docker image contains the Python SDK version, which is `6.73.132`. This tag will be used to check the compatibility with the Supervisely instance version. If you run the release action, you will see the following output:

```text
INFO: Image name: labeling, Image version: 6.72.70
INFO: Standard docker images: collaboration,data-operations,development,import-export,labeling,synthetic,system,visualization-stats
INFO: Docker image labeling is in the list of standard docker images.
INFO: Assuming that the version of the docker image (6.72.70) is a version of the supervisely Python SDK.
INFO: SDK version to check: 6.72.70
INFO: Version info: {
  "6.9.11": "6.72.70",
  "6.9.13": "6.73.76",
  "6.9.18": "6.73.81",
  "6.9.22": "6.73.90",
  "6.9.31": "6.73.123",
  "6.10.0": "6.73.126"
}
INFO: Minimum SDK version for server v6.9.12 is 6.72.70
INFO: Maximum SDK version for server v6.9.12 is 6.73.76
INFO: Server version 6.9.12 is compatible with SDK version 6.72.70
```

So the docker image was found in the list of standard docker images, the Python SDK version was extracted from the tag, and the compatibility check was successful. The app can be released.

#### Case 2: Custom docker images
If the app is using a custom docker image, which doesn't contain the Python SDK version in the tag, there are two main scenarios. The first one is if we're working with a new docker image, which contains a special label with the Python SDK version. When building a new docker image, the build action will automatically retrieve the Python SDK and save it as a label.<br>
In this case you don't need to do anything, just ensure that the `instance_version` parameter is correct and the release action will be successful.<br><br>

The second scenario is when the app is using an old custom docker image, which doesn't contain the Python SDK version in its labels. In this case, you must specify the Supervisely Python SDK version in the release description like this:

```text
python_sdk_version: 6.7.10
```

Otherwise, the release action would have no idea which Python SDK version is used in the docker image and the release will be blocked.<br>
Here's an example of the output if the Python SDK version was not specified in the release description, while the docker image is custom and doesn't contain the Python SDK version in the labels:

```text
INFO: Docker image yolo8 is not in the list of standard docker images.
INFO: Will read release description to find the appropriate SDK version.
ERROR: python_sdk_version not found in the release description.
ERROR: When using custom docker images, you must provide the python_sdk_version in the release description, example: python_sdk_version: 6.73.10
```

So, if you see this output, you must add the `python_sdk_version` parameter to the release description and run the release action again. Or you can build a new docker image with the Python SDK version in the labels and run the release action again.<br>