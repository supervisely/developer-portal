# Create app from any py-script

## Introduction

The main point: ✅ **any python script can be easily transformed into Supervisely App **<mark style="color:green;">**\*\*\*\***</mark>** ✅.** And in this tutorial, you will learn how to do it. It will show you how to add the necessary files and structure to create the app from a python script, and how to add it to the Supervisely platform.

We will write a simple Python program that prints user login to console (stdout) in ASCII art (also known as "computer text art").

We will go through the following steps:

\*\*\*\*[**Step 1.**](from-script-to-supervisely-app.md#step-1.-python-script) Prepare a tiny python script.

\*\*\*\*[**Step 2.**](from-script-to-supervisely-app.md#step-2.-from-script-to-supervisely-app) \*\*\*\* How to transform this script into Supervisely App

\*\*\*\*[**Step 3.** ](from-script-to-supervisely-app.md#step-3.-how-to-add-your-private-app)How to add custom private app into Supervisley Platform.

\*\*\*\*[**Step 4.**](from-script-to-supervisely-app.md#step-4.-run-your-app-in-supervisely) \*\*\*\* How to run it in Supervisely.

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/hello-world-app): source code and additional app files.
{% endhint %}

## Step 1. Python script

The python program that takes user login from ENV variable and prints it to console (STDOUT) as ASCII art using [ART python package](https://github.com/sepandhaghighi/art) [![](https://img.shields.io/github/stars/sepandhaghighi/art.svg?style=social\&label=Stars)](https://github.com/sepandhaghighi/art).

```python
import os
from dotenv import load_dotenv
from art import tprint

# load ENV variables for debug
# has no effect in production
load_dotenv("local.env")


def main():
    name = os.environ["context.userLogin"]
    print("Hello World! This app is run by the user:")
    tprint(name)


if __name__ == "__main__":
    main()
```

Here is an example of the output of this tiny python program:

```
Hello World! This app is run by the user:
                        
 _ __ ___    __ _ __  __
| '_ ` _ \  / _` |\ \/ /
| | | | | || (_| | >  < 
|_| |_| |_| \__,_|/_/\_\
                        
```

## Step 2. From script to Supervisely App

### Repository structure

Supervisely App is just a git repository on Github or Gitlab. For this particular app the files structure should be the following:

```
.
├── README.md
├── config.json
├── requirements.txt
└── src
    └── main.py
```

Let's explain every file:

* `README.md` \[optional] - contains an explanation of what this app does and how to use it. You can provide here all information that can be useful for the end-user (screenshots, gifs, videos, demos, examples).
* `requirements.txt` \[optional] - here you can specify all Python modules (pip packages) that are needed for your python program. This is a common convention in Python development. In our example we use two additional packages: [`art`](https://pypi.org/project/art/) [![](https://camo.githubusercontent.com/d367bde73fa3ec8a38cc54d187094f0a6d2c24f81ec5bba70cd88dc4d6047467/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f73746172732f736570616e6468616768696768692f6172742e7376673f7374796c653d736f6369616c266c6162656c3d5374617273)](https://github.com/sepandhaghighi/art)to do cool prints to console and [`black`](https://pypi.org/project/black/) ![GitHub Org's stars](https://img.shields.io/github/stars/psf/black?style=social) for automatic code formatting.

<pre><code><strong># supervisely SDK
</strong><strong>supervisely
</strong>
<strong># used to print cool text to stdout
</strong>art==5.7 

# my favorite code formatter
black==22.6.0 
</code></pre>

* `config.json` - This file will contain all your app metadata information, like name, description, poster URL, icon URL, app tags for Ecosystem, docker image, and so on. This file will be explained in detail in the next guides.
* `src/main.py` our python program.

The two files below are in the repo but they are used **ONLY** for debug purposes and are provided for your convenience.

```
.
├── create_venv.sh
└── local.env
```

### App configuration

App configuration is stored in `config.json` file. A detailed explanation of all possible fields in app configuration will be covered in other tutorials. Let's check the config for our small app:

```json
{
  "main_script": "src/main.py",
  "headless": true,
  "name": "Hello World!",
  "description": "Demonstrates how to turn your python script into Supervisely App",
  "categories": ["development"],
  "icon": "https://user-images.githubusercontent.com/12828725/182186256-5ee663ad-25c7-4a62-9af1-fbfdca715b57.png",
  "poster": "https://user-images.githubusercontent.com/12828725/182181033-d0d1a690-8388-472e-8862-e0cacbd4f082.png"
}
```

Let's go through the fields:

* `main_script` - relative path to the main script (entry point) in a git repository
* `"headless": true` means that app has no User Interface
* `name`, `description` and `poster` define how the app will look in the Supervisely Ecosystem

![poster, name, descripotion](https://user-images.githubusercontent.com/12828725/182863249-0b4d672f-f50d-4bbb-b769-ec1016539ccd.png)

* `icon`, `categories` - categories help to navigate in the Supervisely Ecosystem and it is a user-friendly way to explore apps

![icon and categories](https://user-images.githubusercontent.com/12828725/182864521-319fb450-d025-4e1c-806e-ebc0dd19260f.png)

## Step 3. How to add your private app

There are two following ways to add an application

### Add app from git repository

Supervisely supports both private and public apps.

🔒 **Private apps** are those that are available only on private Supervisely Instances (Enterprise Edition).

🌎 **Public apps** are available on all private Supervisely Instances and in Community Edition. The guidelines for adding public apps will be covered in other tutorials.

Since Supervisely app is just a git repository, we support public and private repos from the most popular hosting platforms in the world - **GitHub** and **GitLab**. You just need to generate and provide access token to your repo. Learn more in [the documentation](https://docs.supervise.ly/enterprise-edition/advanced-tuning/private-apps).

Go to `Ecosystem` -> `Private Apps` -> `Add private app`.

![Add private app](https://user-images.githubusercontent.com/12828725/182870411-6632dde4-93ed-481c-a8c2-79718b0f5a7d.gif)

### Add app directly to the supervisely instance via apps-cli

Install supervisely-app cli via following command:

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/supervisely/supervisely/master/cli/supervisely-app.sh)"
```

Add `release` and `slug` properties in `config.json`:

```
  "release": { "version":"v1.0.0", "name":"init" },
  "slug": "<organication_name>/<app-name>",
```

Create .env file `~/supervisely.env` with the following content:

```python
SERVER_ADDRESS="https://<server-address>"
API_TOKEN="4r47N...xaTatb"
```

Go root folder of your app folder and run:

```
supervisely-app release
```

As an alternative to creating `~/supervisely.env`, you can use `-t` and `-s` flags when publishing a new version:

```
supervisely-app release -s https://<server-address> -t 4r47N...xaTatb
```

## Step 4. Run your app in Supervisely

There are multiple ways how application can be integrated into Supervisely Platform. App can be run from context menu of project / dataset / labeling job / user / and so on ... Or app can be run right from labeling interface. All possible running options will be covered in next tutorials.

For simplicity, we will run our app from the Ecosystem page.

![Let's run our app](https://user-images.githubusercontent.com/12828725/182894602-5ec6a5c6-e954-429b-9fc1-877d662a21ec.gif)
