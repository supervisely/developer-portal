# Add private app

## Introduction

Supervisely supports both private and public apps.

🔒 **Private apps** are those that are available only on your private Supervisely Instance (Enterprise Edition) in your account.

🌎 **Public apps** are available on all private Supervisely Instances and in Community Edition. The guidelines for adding public apps will be covered in other tutorials.

This tutorial covers the case of adding a custom private app to your private instance. It means that this app will be available only for your account and only on your private Supervisely instance.

Apps, developed by the Supervisely team, are open-sourced and are available on all Supervisely instances (Community Edition and all private customer's instances with Enterprise Edition license). The case of publishing an app to the global public Supervisely Ecosystem will be covered in another tutorial.


## Version releases and Branch releases
There are two types of releases: `version` and `branch`. The `version` release is made from the `main` or `master` branch. With each `version` release, a release tag is added to the last commit. This tag is used to identify the release version and is important for the app versioning. If during the release process, the tag is not created, the release will be rejected.
With version releases, you can specify the release version and description. The `branch` releases are for testing and debugging and are made from any other branch except `main` or `master`. With branch releases, you can't specify the release version and description. The release version will be the branch name.

## Multi-app repositories
In Supervisely you can have a single git repository with multiple applications. It is advised to have connected applications with a common codebase in the same repository. By default `supervisely release` command will release the application from the root directory of the repository. If you have multiple applications in the repository, you can specify the path to the application directory with the `-a` flag. The application directory is a directory with a `config.json` file. Such applications are called `subapps`. For example, if you have a repository with two applications in the `train` and `serve` directories, you can release the `train` application with the following command:

```bash
supervisely release -a train
```

{% hint style="info" %}
When making a release the tag is added to the last commit. And since all of the applications are in the same repository, it is impossible to differentiate the release tags of different applications. Therefore, it is advised to do a release for each subapp with every new version.
{% endhint %}

## How to share the private app with Team members

After the private app is released it will be available for the user who released the app. The app can be shared with the team members without the need to share it with the whole instance. To do it, you need to run the app once while being a member of the team. After that, the app will appear on the `App sessions` page and will become available by URL for any team member.


## Option 1. \[👍 Recommended] CLI - Run command in terminal.

### Step 0. Install Supervisely SDK

Run command in terminal to install Supervisely SDK

```bash
pip install supervisely
```

### Create a .env file `~/supervisely.env` with the following content (learn more [here](../../getting-started/basics-of-authentication.md):

```python
SERVER_ADDRESS="<server-address>"
API_TOKEN="4r47N...xaTatb"
```

### Development in a team

For team development, you need to add `APP_RELEASE_TOKEN` variable to your `~/supervisely.env` file. This token will be used to authenticate your app during the release process. If `APP_RELEASE_TOKEN` is present in your `~/supervisely.env` file, then the app will be owned by the user associated with the token and any user will be able to do a release if he has the token. Otherwise, the app will be owned by the user who released the app and releases from other users will be rejected.

### How to get `APP_RELEASE_TOKEN`

1. Create a new user on your instance. This user will be used for releasing apps. You can name it `dev` or `dev-team` or whatever you want.

![dev-in-team-1](https://github.com/supervisely/developer-portal/assets/61844772/a1fefab3-cc6a-42f1-9509-feef38209b04) ![dev-in-team-2](https://github.com/supervisely/developer-portal/assets/61844772/a58d9fb4-d738-4d8c-9849-6735a2335519)

2. Login as this user and copy API token.

![dev-in-team-3](https://github.com/supervisely/developer-portal/assets/61844772/604b5f9a-41d7-4a92-9ccb-d7930ebcd3a0) ![dev-in-team-4](https://github.com/supervisely/developer-portal/assets/61844772/cb2b0602-7094-49ad-bd7e-cede58fa242e)

3. Use this token as `APP_RELEASE_TOKEN` in your `~/supervisely.env` file.

```python
SERVER_ADDRESS="<server-address>"
API_TOKEN="4r47N...xaTatb"
APP_RELEASE_TOKEN="xaTatb...4r47N"
```

### How to pass ownership of an app to another user

If you released an app without `APP_RELEASE_TOKEN` and now want to continue development in a team you can pass the ownership to the user created in previous steps. To do this you need to go to the private app page, navigate to the bottom left part and click `Change owner` button. Then input login of the user. You will be still able to see this app in your private apps. But to make new releases you will need to use `APP_RELEASE_TOKEN` of the new owner.

![dev-in-team-change-ownership](https://github.com/supervisely/developer-portal/assets/61844772/d7308c17-3e7d-46e2-88a5-c45c9cb4b76f)

### Step 1. Prepare a directory with app sources.

You are a developer and you implemented an app. App sources are on your local computer in some directory. Go to this folder. For example, the folder structure will look like this:

```
.
├── README.md
├── config.json
├── requirements.txt
└── src
    └── main.py
```

### Step 2. Release

Execute the following command in the terminal to release an app. By default, this command will pack and release files in the current folder.

```
supervisely release
```

You will be asked for a release description and in case of releasing from main/master branch for release version. After that, you will see a summary message and confirmation request. If releasing from main/master branch new tag will be created and pushed to remote (You may be asked for git authentication). Then if there are no errors you will see the "App release successfully!" message.

![release from main/master branch](https://user-images.githubusercontent.com/61844772/225958325-f6e2a964-ba74-4386-ac9f-28b5819ff40f.png)

![release from other branch](https://user-images.githubusercontent.com/61844772/225957782-2c6557e4-93ed-4ab2-a40e-4268b7110976.png)

You can provide release version and release description by providing `--release-version` and `--release-description` options to the CLI

Your app will appear in the section `🔒 private` apps` in Ecosystem.

![private apps](https://user-images.githubusercontent.com/12828725/205959921-7d631cb5-c1fc-4b0c-99d5-f2260c96708d.png)

Thus you can quickly do releases of your app. All releases will be available on the application page. Just select the release in the modal window in the `advanced` section before running the app. The latest release is selected by default.

![app versions](https://user-images.githubusercontent.com/12828725/205960656-615803f0-c081-4086-b7ba-45f4bbc60cb6.png)

{% hint style="info" %}
You can store several applications in one repository. To release an application from such repository, go to root folder of the repository, then run `supervisely release` with `-a` flag and specify the relative path to folder with application configuration file

```
cd ~/code/yolov5
supervisely release -a apps/train
```
{% endhint %}

## Option 2. Connect your git account (Github or Gitlab).

Since Supervisely app is just a git repository, we support public and private repos from the most popular hosting platforms in the world - **GitHub** and **GitLab**. You just need to generate and provide the access token to your repo.

### Step 1. Generate new personal token

#### GitHub

To access private GitHub repositories, you will need to generate a personal token. Please note, that this token will provide your Supervisely instance a read access to all repositories, available for this GitHub account — you may want to create a dedicated GutHub account for a single Supervisely App repository.

Open GitHub → Settings → Developer settings → [Personal access tokens](https://github.com/settings/tokens) and click Generate new token.

Select "repo" access scope and click "Generate token" button. Save generated token — you will need it later.

![](https://raw.githubusercontent.com/supervisely/docs/master/enterprise/private-apps/personal-token.png)

#### GitLab

To access private GitLab repositories, you will need to generate a personal token. Please note, that this token will provide your Supervisely instance a read access to all repositories, available for this GitLab account — you may want to create a dedicated GutLab account for a single Supervisely App repository.

Open GilLab → Settings → [Access Tokens](https://docs.gitlab.com/ee/user/profile/personal\_access\_tokens.html#create-a-personal-access-token)

Select with "read\_api", "read\_repository" scopes enabled and click "Create personal access token" button. Save generated token — you will need it later.

![](https://raw.githubusercontent.com/supervisely/docs/master/enterprise/private-apps/personal-token-gitlab.png)

### Step 2. Create repository

#### GitHub

Let's create a new GitHub repository that we will use to deploy a new Supervisely application. Create a [new private GitHub repository](https://github.com/new): do not forget to choose "Private" visibility option.

![](https://raw.githubusercontent.com/supervisely/docs/master/enterprise/private-apps/new-repo.png)

#### GitLab

Let's create a new GitLab repository that we will use to deploy a new Supervisely application. Create a new project.

![](https://raw.githubusercontent.com/supervisely/docs/master/enterprise/private-apps/new-repo-gitlab.png)

{% hint style="info" %}
You can create a public repository alright — you will still need a personal token and further steps are gonna be the same.
{% endhint %}

### Step 3. Make it a Supervisely App repository

In this tutorial we will use [While(true) app](https://github.com/supervisely-ecosystem/while-true-script) code-base as a starting point — it's a bare minimum sample application that, basically, just runs an infinite loop.

We will download its source code, extract it, create a new repository and initialize it:

```
wget -O while-true-app.zip  https://github.com/supervisely-ecosystem/while-true-script/archive/refs/heads/master.zip
unzip while-true-app.zip
cd while-true-script-master/
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/supervisely/my-first-private-app.git # Your actual repository name here...
git push -u origin main
```

You will find a few files in your new application:

* `config.json` describes your app name, type, etc.
* `requirements.txt` list python packages you will need
* `README.md` in markdown format
* `src/main.py` your entry-point python file

Let's leave it as is for now

### Step 4. Add Application to Supervisely

Go to Ecosystem page → Private apps → Click "Add private app"

![](https://raw.githubusercontent.com/supervisely/docs/master/enterprise/private-apps/add-app-page.png)

![](https://raw.githubusercontent.com/supervisely/docs/master/enterprise/private-apps/add-app-modal.png)

### Step 5. Check your first application

Now, open the Ecosystem page in the left menu and choose "Private Apps" in the right menu. You should see here your new application after a minute. Add it to your team and try it out!

Next time you push a new update to your repository, do not forget to open the application in Ecosystem and click "Refresh" button to update it.

## Option 3. Create a release on GitHub

{% hint style="info" %}
Only for Supervisely Team.
{% endhint %}

### Step 1. Create a repository

Create an app repository on GitHub in a [Supervisely-ecosystem organization](https://github.com/supervisely-ecosystem). \*[How to create an app](from-script-to-supervisely-app.md)

### Step 2. Add GitHub workflow

Add a GitHub workflow files from [this repository](https://github.com/supervisely-ecosystem/workflows):

1. For version releases (see step 3.1): [.github/workflows/release.yml](https://github.com/supervisely-ecosystem/workflows/blob/master/.github/workflows/release.yml)
2. For branch releases (see step 3.2): [.github/workflows/release\_branch.yml](https://github.com/supervisely-ecosystem/workflows/blob/master/.github/workflows/release\_branch.yml)

```yaml
name: Release
run-name: Release version "${{ github.event.release.tag_name }}"
on:
  release:
    types: [published]
    branches:
      - main
      - master
jobs:
  Supervisely-Release:
    ***
    with:
      ***
      SUBAPP_PATHS: "__ROOT_APP__, subapp" <-- Change this variable
```

In each of these files, you should change the following variable: `SUBAPP_PATHS` - Paths to directories with applications within the repository (directory where the `config.json` file is located). If the application is located in a root directory, then you should specify `__ROOT_APP__` instead of the path. Paths should be separated by commas.

In the example above, releases are configured for two applications in the repository: the one which is located in `root` directory and the one which is located in the `subapp` directory. Example for the repository with two applications, located in `train` and `serve` directories: `SUBAPP_PATHS: "train, serve"`.

### Step 3. Create a release

#### 3.1 Version release

The workflow we created in the previous step will be triggered when you publish a release in the repository.

To create a release, go to the repository page on GitHub and click on the `Releases` tab. Then click on the `Create a new release` or `Draft a new release` button. Choose a tag version in semver format (v1.0.0) and a release title. Then click on the `Publish release` button.

{% hint style="warning" %}
Do not change the Target of the release. It should always be `main` or `master`.
{% endhint %}

![](https://github.com/supervisely/developer-portal/assets/61844772/42de54db-8f72-4c80-9e69-84ce4b887b90)

![](https://github.com/supervisely/developer-portal/assets/61844772/946b16ba-bb7c-4af0-a13b-c0f9666e80a3)

![](https://github.com/supervisely/developer-portal/assets/61844772/57200843-dcc2-4b21-be69-fcd230a6383a)

![](https://github.com/supervisely/developer-portal/assets/61844772/39468af0-595e-42b2-bde4-696891bf377b)

#### 3.2 Branch release

The workflow we created in the previous step will be triggered when you push a branch (except "main" or "master") in the repository.

{% hint style="info" %}
You can disable branch release by adding the branch name to `branches-ignore` list in the `.github/workflows/release_branch.yml` workflow file. See below
{% endhint %}

```yaml
name: Release branch
run-name: Release "${{ github.ref_name }}" branch
on:
  push:
    branches-ignore:
      - main
      - master
      - branch-to-ignore <-- Add branch name here
jobs:
  Supervisely-Release:
```

#### After the release is published the workflow will be triggered and you can see the release progress in the `Actions` tab. If the workflow is successful, the app will appear in the ecosystem.

![](https://github.com/supervisely/developer-portal/assets/61844772/187bca28-e3cc-4a9d-b772-65c279cd6c65)
