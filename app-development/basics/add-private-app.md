# Add private app

## Introduction

Supervisely supports both private and public apps.

🔒 **Private apps** are those that are available only on your private Supervisely Instance (Enterprise Edition) in your account.

🌎 **Public apps** are available on all private Supervisely Instances and in Community Edition. The guidelines for adding public apps will be covered in other tutorials.

This tutorial covers the case of adding custom private app into your private instance. It means that this app will be available only for your account and only on your private Supervisely instance.

Apps, developed by Supervisely team, are open-sourced and are available on all Supervisely instances (Community Edition and all private customer's instances with Enterprise Edition license). The case of publishing an app into global public Supervisely Ecosystem will be coved in another tutorial.

## Option 1. \[👍 Recommended] Run command in terminal.

### Step 0. Install Supervisely App CLI tool.

Run command in terminal (this is one-time procedure) to install CLI tool:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/supervisely/supervisely/master/cli/install.sh)"
```

### Create .env file `~/supervisely.env` with the following content (learn more [here](../../getting-started/basics-of-authentication.md):

```python
SERVER_ADDRESS="<server-address>"
API_TOKEN="4r47N...xaTatb"
```

---------

**Chnages:**

- SERVER_ADDRESS - your instance serverver address (e.g. https://app.supervise.ly)
- API_TOKEN - your user API_TOKEN (to get api token click to your user name in top right corner of the screen -> Account settings -> API token)

{% hint style="info" %}
**Supervisely team only!** You should use "supervisely" user API token
{% endhint %}

---------

### Step 1. Prepare a directory with app sources.

You are a developer and you implemented an app. App sources are on your local computer in some directory. Go to this folder. For example, folder structure will look like this:

```
.
├── README.md
├── config.json
├── requirements.txt
└── src
    └── main.py
```

Be sure, that the `config.json` file contains the following fields with release information:

```
  "slug": "<organization_name>/<app-name>",
  "release": { "version":"v0.0.1", "name":"init" },
```

---------

**Chnages:**

#### Slug

Field `slug` is your application unique identifier. It **must** contain `/` symbol. Examples: `"slug": "maxim/my-super-app",` or `"slug": "my-company/my-super-app",`, or `"slug": "xxx/my-super-app",`.

{% hint style="warn" %}
You should not change slug after release on the instace
{% endhint %}

{% hint style="info" %}
If you store multiple applications in the same repository, slug in each application config.json file must contain path to that application folder.
Examples: `"slug": "my-company/my-super-project/apps/train",`.
{% endhint %}

{% hint style="info" %}
**Supervisely team only!** To develop and test applications on the dev server you should use slug starts with "supervisely-ecosystem/..."
{% endhint %}


#### Release

Release version support two different formats:
- vX.X.X (e.g v1.2.35) - stable version. After release stable versions are immutable 
- {text} (e.g debug, new-feature) - branch. Preferred for development as it supports updates by calling the "release" command again

{% hint style="info" %}
After development of a new feature is completed, it is recommended to make a new stable version release (e.g. v1.2.36)
{% endhint %}

---------

### Step 2. Release

Execute the following command in the terminal to release an app. By default this command will pack and release files in the current folder.

```
supervisely-app release
```

Your app will appear in section `🔒 private apps` in Ecosystem.

![private apps](https://user-images.githubusercontent.com/12828725/205959921-7d631cb5-c1fc-4b0c-99d5-f2260c96708d.png)

Thus you can quickly do releases of your app. All releases will be available on the application page. Just select the release in modal window in advanced section before running the app. The latest release is selected by default.

![app versions](https://user-images.githubusercontent.com/12828725/205960656-615803f0-c081-4086-b7ba-45f4bbc60cb6.png)

---------

**Chnages:**

{% hint style="info" %}
**Supervisely team only!** Apps you add to the dev server will appear in the general app list, not in "Private apps"
{% endhint %}

---------

{% hint style="info" %}
You can store several applications in one repository. To release an application from such repository, go to root folder of the repository, then run `supervisely-app` with `-a` flag and specify the relative path to folder with application configuration file

```
cd ~/code/yolov5
supervisely-app release -a apps/train
```
{% endhint %}

#### Apps development in a team

Development of private app is available only to its author by default. In order to give access to another developer, go to the app page in the ecosystem and click "Access" button. Select the users you want to give access to and click "Apply".

## Option 2. Connect your git account (Github or Gitlab).

Since Supervisely app is just a git repository, we support public and private repos from the most popular hosting platforms in the world - **GitHub** and **GitLab**. You just need to generate and provide access token to your repo.

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
You can create a public repositry alright — you will still need a personal token and further steps are gonna be the same.
{% endhint %}

### Step 3. Make it a Supervisely App repository

In this tutorial we will use [While(true) app](https://github.com/supervisely-ecosystem/while-true-script) code-base as a starting point — it's a bare minimum sample application that, basically, just runs an infinite loop.

We will download it's source code, extract it, create a new repository and initialize it:

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

Now, open Ecosystem page in the left menu and choose "Private Apps" in the right menu. You should see here your new application after a minute. Add it to your team and try it out!

Next time you push a new update to your repository, do not forget to open application in Ecosystem and click "Refresh" button to update it.

### Optional: Releases

Supervisely Apps support multiple versions via GitHub releases — this is a convenient feature once you are ready to mark your first version as a release. Just go to Releases section of your GitHub repository, click "Create a new release" button and choose a "v1.0.0" tag — next time you refresh your app in Ecosystem, you will see your release and will be able to switch between them.
