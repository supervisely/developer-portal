# How to add private app into Supervisely instance

## Introduction

This tutorial covers the case of adding custom private app into your private instance. It means that this app will be available only for your account and only on your Supervisely instance.

Apps, developed by Supervisely team, are open-sourced and are available on all Supervisely instances (Community Edition and all private customer's instances with Enterprise Edition license). The case of publishing an app into global public Supervisely Ecosystem will be coved in another tutorial.

## Option 1. [Recommended] Run command in terminal.

### Step 0. Install Supervisely App CLI tool.

Run command in terminal (this is one-time procedure) to install CLI tool:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/supervisely/supervisely/master/cli/supervisely-app.sh)"
```

### Step 1. Install Supervisely App CLI tool.


You are a developer and you implemented an app. App sources are on your local computer in directory. Go to this folder. For example, folder structure will look like this:

```
.
├── README.md
├── config.json
├── requirements.txt
└── src
    └── main.py
```



