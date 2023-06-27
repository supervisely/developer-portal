# Add public app

## Introduction

Supervisely supports both private and public apps.

🔒 **Private apps** are those that are available only on your private Supervisely Instance (Enterprise Edition) in your account. The guidelines for adding public apps is [this tutorial](add-private-app.md).

🌎 **Public apps** are available on all private Supervisely Instances and in Community Edition. 

This tutorial covers the case of adding a public app. It means that this app is open-sourced and is available on all Supervisely instances (Community Edition and all private customer's instances with Enterprise Edition license).

{% hint style="info" %}
Only for Supervisely Team.
{% endhint %}

### Step 1. Create a repository

Create an app repository on GitHub in a [Supervisely-ecosystem organization](https://github.com/supervisely-ecosystem). *[How to create an app](from-script-to-supervisely-app.md)

### Step 2. Add GitHub workflow

Add a GitHub workflow file from [This file](https://github.com/supervisely-ecosystem/workflows/blob/master/.github/workflows/release.yml).

```yaml
name: Supervisely release
run-name: Supervisely ${{ github.repository }} app release
on:
  release:
    types: [published]
    branches:
      - main
      - master
jobs:
    ***
      SUBAPP_PATHS: "__ROOT_APP__, subapp"
```

In this file, you should change the following variable: `SUBAPP_PATHS` - Paths to directories with applications within the repository (directory where the `config.json` file is located). If the application is located in a root directory, then you should specify `__ROOT_APP__` instead of the path. Paths should be separated by commas. 

In the example above, releases are configured for two applications in the repository: the one which is located in `root` directory and the one which is located in the `subapp` directory. 
Example for the repository with two applications, located in `train` and `serve` directories: `SUBAPP_PATHS: "train, serve"`.

### Step 3. Create a release

The workflow we created in the previous step will be triggered when you publish a release in the repository. 

To create a release, go to the repository page on GitHub and click on the `Releases` tab. Then click on the `Create a new release` or `Draft a new release` button. Choose a tag version in semver format (v1.0.0) and a release title. Then click on the `Publish release` button.

{% hint style="warning" %}
Do not change the Target of the release. It should always be `main` or `master`.
{% endhint %}

<p align="center"><img src="https://github.com/supervisely/developer-portal/assets/61844772/42de54db-8f72-4c80-9e69-84ce4b887b90" style="width: 100%"/></p>
<p align="center"><img src="https://github.com/supervisely/developer-portal/assets/61844772/946b16ba-bb7c-4af0-a13b-c0f9666e80a3" style="width: 100%"/></p>
<p align="center"><img src="https://github.com/supervisely/developer-portal/assets/61844772/57200843-dcc2-4b21-be69-fcd230a6383a" style="width: 100%"/></p>
<p align="center"><img src="https://github.com/supervisely/developer-portal/assets/61844772/39468af0-595e-42b2-bde4-696891bf377b" style="width: 100%"/></p>

After the release is published the workflow will be triggered and you can see the release progress in the `Actions` tab. If the workflow is successful, the app will appear in the ecosystem.

<p align="center"><img src="https://github.com/supervisely/developer-portal/assets/61844772/187bca28-e3cc-4a9d-b772-65c279cd6c65" style="width: 100%"/></p>

### Step 4. Updating the app

When you need to update the app, you need to create a new release. To do so you need to create a new release like in the previous step. After the release is published the workflow will be triggered and you can see the release progress in the `Actions` tab. If the workflow is successful, a new release will appear in the `Releases` tab of your application.


## App development process

### App Development and local tests

The developer creates the application and tests it locally.

### Testing on development instance

When you done with development and local tests, you can test your app on the development instance. To do so you need to create a private app on the development instance and then create a new release. *[How to add private app](add-private-app.md).

The preferred way is to use CLI tool from the SDK. 
From the environment where you have installed the SDK, run the following command: `supervisely release` and follow the instructions. Make sure you set the correct server address and API token.
After the release is created, you can find the application in the [Private apps tab of the ecosystem](https://dev.supervise.ly/ecosystem/private).

{% hint style="info" %}
For development in a team you need to add `RELEASE_TOKEN` variable to your `~/supervisely.env` file. You can find `RELEASE_TOKEN` for Supevisely team in the slack: [message](https://deepsystems.slack.com/archives/CV28AA11P/p1687854996974239) 
{% endhint %}

### Releasing the app to the public

When you are ready to release your app to the public, you need to create a public app. To do so you need to follow the steps described in the [steps 1 to 3](#step-1-create-a-repository) of this tutorial. After the app is released to the public you will no longer be able to create releases via CLI tool.

{% hint style="info" %}
Do not forget to add the app to the [README_v2](https://github.com/supervisely-ecosystem/repository/blob/master/README_v2.md). It is needed for back-compatibility with older versions of Supervisley instances. You may check that your app is released on older instances by checking that app is available on the instance from [this message](https://deepsystems.slack.com/archives/CV28AA11P/p1687854996974239).
{% endhint %}

### Developing new features

{% hint style="warning" %}
Still in development
{% endhint %}

You may be asking yourself: "How can I develop new features for my app if I can't create releases via CLI tool?". There is a solution for that. Future feature development and testing are done in development branches (any branch other than `main` or `master`).

To activate this mechanism you need to add another workflow file to the repository: You can use [This file](https://github.com/supervisely-ecosystem/workflows/blob/master/.github/workflows/release_dev.yml).

```yaml
name: Supervisely release
run-name: Supervisely ${{ github.repository }} app release
on:
  push:
    branches-ignore:
      - main
      - master
jobs:
    ***
      SUBAPP_PATHS: "__ROOT_APP__, subapp"
```

Same as in [step 2](#step-2-add-github-workflow) you need to change the `SUBAPP_PATHS` variable.

This workflow will be triggered on any push to any branch other than `main` or `master`. It will create a release with the name of the branch. For example, if you push to the `dev` branch, the release will be created with the version `dev` and the name `dev branch release`.

{% hint style="info" %}
You can also limit branches on which the workflow will be triggered. To do so you need to replace `branches-ignore` parameter with `branches` parameter to the `on` section of the workflow. For example, if you want to trigger the workflow only on the `test` branch you need to add
on:
  push:
    branches:
      - test
{% endhint %}

### Updating the app

When you need to update the app, you need to create a new release. It is described in [step 4](#step-4-updating-the-app) of this tutorial.

