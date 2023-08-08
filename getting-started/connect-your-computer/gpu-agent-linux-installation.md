---
description: >-
  Everything you need to know about deploying Supervisely agent on Linux based
  operating systems
---

# Linux

This guide aims to lead you through the step-by-step process for establishing and configuring the Supervisely agent within your Unix-based operating system environment.

## Prerequisites

* Linux OS (Kernel 3.10 or higher)
* [Docker](https://docs.docker.com/engine/install/) (Version 18.0 or higher)
* [CUDA](https://developer.nvidia.com/cuda-downloads) (Version 9.0 or higher)
* [NVIDIA Docker](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

![Requirements](https://github.com/supervisely/developer-portal/assets/48913536/ebc176c8-7648-4cb8-8f50-32e690394838)

**Make sure that the computer you want to use in Supervisely meets the above requirements.**

## Deploy Supervisely Agent

Deploy Supervisely Agent with GPU support on Linux.

Open Supervisely instance and go to the Start -> Team Cluster page and press "Add" button

![Add Agent](https://github.com/supervisely/developer-portal/assets/48913536/ced70275-777f-4643-aefd-991ffc902971)

Select "Supervisely agent".

![Select Agent](https://github.com/supervisely/developer-portal/assets/48913536/753cff60-1a9e-49ad-9121-193141bb2e4e)

In the modal window go to "advanced settings" and check "Use nvidia runtime" option to enable GPU support.

![Agent Settings](https://github.com/supervisely/developer-portal/assets/48913536/014aab71-6dad-4f9f-b5d8-9a2cce36f66e)

Copy the command to the terminal and run it.

![Terminal](https://github.com/supervisely/developer-portal/assets/48913536/6053c5b3-c983-4060-acea-b85152178735)

Wait until the Docker image is pulled and you see the message: "You can close this terminal safely now".

![Terminal Bash](https://github.com/supervisely/developer-portal/assets/48913536/58da5569-7bf3-4176-9a2b-063a0b731bcb)

That's it! Now your agent is deployed and running.

## Video Guide

To connect the agent you can follow the steps as they described in the following video guide:

{% embed url="https://github.com/supervisely/docs/assets/48245050/832ae03d-58d4-4abb-9e8c-307ca8f4b072" %}
