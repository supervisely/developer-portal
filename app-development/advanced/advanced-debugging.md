---
description: This guide explains how to debug your application
---

# Advanced Debugging

## Introduction

In this tutorial we are going to show you how to run application in debug mode and interact with it from Supervisely platform

---

## Prepare environment

**Step 1.** Prepare  `~/supervisely.env` file with credentials. [Learn more here.](../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/integrate-inst-seg-model) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/integrate-inst-seg-model
cd integrate-inst-seg-model
./create_venv.sh
```

**Step 3.** Install [wireguard-tools](https://www.wireguard.com/install/)
```bash
sudo apt install wireguard
```

**Step 4.** Open repository directory in Visual Studio Code.&#x20;

```bash
code -r .
```

**Step 5.** Change `TEAM_ID` of `"Advanced mode for Supervisely Team"` configuration in `.vscode/launch.json` file by copying the ID from the context menu of the team. A new debug app session will be created for the team you define:

```python
{
    "name": "Advanced mode for Supervisely Team",
    ***
    "env": {
        ***
        "TEAM_ID": "7", // ⬅️ change value
        ***
    }
}
```

**Step 6.** Copy contents of `my_model` directory to `results` and download model weights.
```bash
mkdir -p results/model
cp -R my_model/* results/model
cd results/model
./init.sh
cd ../..
```

---

## Run debug

**Step 1.** Go to `Run and Debug` section `(Ctrl+Shift+D)`.

**Step 2.** Select `Advanced mode for Supervisely Team` from configuration dropdown

**Step 3.** Press green triangle or `F5` to start debugging. You should see `Debug task has been successfully created: 12345` or `Debug task already exists: 12345` message in console logs.

{% hint style="info" %}
If you need to debug SDK, you can clone Supervisely repository and create a Symbolic Link to it inside of your project.
```bash
cd *your_desired_dir*
git clone git@github.com:supervisely/supervisely.git
cd back to project
ln -s *your_desired_dir*/supervisely/supervisely ./supervisely
```
{% endhint %}

---

## Interact with app from Supervisely paltform

**Step 1.** Go to Supervisely platform
- [Community Edition](app.supervise.ly/)
- [Dev Edition](dev.supervise.ly/)
- Or your enterprise platform

**Step 2.** Select `App sessions` from start menu. You will see a list of applications. Find app named `Develop and Debug` and click on sessions button. There you should see your debug app session with the same `ID` as in **Step 3** of previous section.

**Step 3.** Now you can try the app. Choose any project or create new one e.g. [Demo Images](https://dev.supervise.ly/ecosystem/projects/demo-images). Open dataset in `Basic image labeling toolbox`.

**Step 4.** Run `NN Image Labeling` app from the list and connect to your app session. Press `Apply model to image` button `(Ctrl+m)`. Predictions are loaded and added to image automatically without need to refresh the page.

**Step 5.** You can controll the execution of the application with any debugging instrument such as breakpoints. 
