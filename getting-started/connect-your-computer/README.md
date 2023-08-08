# Connect your computer

Supervisely Agent is a tiny docker container that allows you to connect your computational resources (cloud server or PC) to the platform. You can run any task from web interface (for example Neural Network training/inference/deploy). Running tasks with GPU will enhance performance and efficiency for your computer vision and deep learning projects.

After you run Agent on your computer, Agent will automatically connect your server to Supervisely platform. You will see this information on the "Team Cluster" page.

{% hint style="info" %} We only run the tasks that you have explicitly started yourself on your agents. We will never use your nodes for the benefit of others. {% endhint %}

![Team Cluster](https://github.com/supervisely/developer-portal/assets/48913536/885e3dbf-4b82-428a-a9bb-775cb6286018)

## Instructions for different operating systems:

* [Linux](gpu-agent-linux-installation.md)
* [Windows WSL](gpu-agent-wsl-installation.md)
