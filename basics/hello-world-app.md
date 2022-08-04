# Hello World App

## Introduction

The main point: <mark style="color:green;">**any python script can be easily transformed into Supervisely App**</mark>**.** And in this tutorial, you will learn how to do it.  It will show you how to add the necessary files and structure to create the app from python script, and how to add it to the Supervisely platform.

We will write a simple Python program that prints user login to console (stdout) in ASCII art (also known as "computer text art").

We will go through the following steps:

****[**Step 1**](hello-world-app.md#step-1.-python-script)**.** Prepare a tiny python script.

****[**Step 2**](hello-world-app.md#step-2.-from-python-script-to-supervisely-app)**.** How to transform this script into Supervisely App

**Step 3.** How to add custom private app into Supervisley Platform.

**Step 4.** How to run it in Supervisely.

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/hello-world-app): source code and additional app files.
{% endhint %}

## Step 1. Python script

The python program that takes user login from ENV variable and prints it to console (STDOUT) as ASCII art using [ART python package](https://github.com/sepandhaghighi/art)  [![](https://img.shields.io/github/stars/sepandhaghighi/art.svg?style=social\&label=Stars)](https://github.com/sepandhaghighi/art).

```python
import os
from dotenv import load_dotenv
from art import tprint

# load ENV variables for debug
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

Supervisely App is just a git repository on Github or Gitlab. For this particular app the files structure  should be the following:

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
* `requirements.txt`  \[optional] - here you can specify all Python modules (pip packages) that are needed for your python program. This is a common convention in Python development.
* `config.json` - This file will contain all your app metadata information, like name, description, poster URL, icon URL, app tags for Ecosystem, dockerimage, and so on. This file will be explained in detail in the next guides.
* `src/main.py` our python program.&#x20;

The two files below are in the repo but they are used **ONLY** for debug purposes and are provided for your convenience.

```
.
├── create_venv.sh
└── local.env
```

### App configuration

App configuration is stored in `config.json` file:

