# Connect your computer to Supervisely

Supervisely Agent is a tiny docker container that allows you to connect your computational resources (cloud server or PC) to the platform. You can run any task from web interface (for example Neural Network training/inference/deploy).

After you run Agent on your computer, Agent will automatically connect your server to Supervisely platform. You will see this information in "Team Cluster" page.


**Note:** We only run the tasks that you have explicitly started yourself on your agents. We will never use your nodes for benefit of others.

## Instructions for different operating systems:

* [Unix](gpu-agent-unix-installation.md)
* [Windows WSL](gpu-agent-wsl-installation.md)