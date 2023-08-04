---
description: Everything you need to know about deploying Supervisely agent on Windows WSL
---

![Poster](https://github.com/supervisely/developer-portal/assets/48913536/65111fd2-e58b-4a8e-86be-12bab6709b68)

# How to deploy Supervisely agent with GPU on Windows WSL

Our machine specs:

OS Name: Microsoft Windows 11 Pro
OS Version: 10.0.22621 Build 22621
GPU: NVIDIA GeForce RTX 3080 Ti (Laptop)
UBUNTU: 22.04.2 LTS
Docker Desktop Version: 4.1.1 (69879)
GPU Driver Version: 536.67

# Table of Contents

- [How to deploy Supervisely agent with GPU on Windows WSL](#how-to-deploy-supervisely-agent-with-gpu-on-windows-wsl)
- [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [How to install](#how-to-install)
  - [Step 1. Turn on WSL](#step-1-turn-on-wsl)
  - [Step 2. Install Windows Terminal](#step-2-install-windows-terminal)
  - [Step 3. Install Ubuntu](#step-3-install-ubuntu)
  - [Step 4. Install NVIDIA GPU Driver](#step-4-install-nvidia-gpu-driver)
  - [Step 5. Docker Desktop](#step-5-docker-desktop)
  - [Step 6. Install NVIDIA Container Toolkit](#step-6-install-nvidia-container-toolkit)
  - [Step 7. Deploy Supervisely Agent](#step-7-deploy-supervisely-agent)

## Prerequisites

* Windows 10 Pro or Enterprise (64-bit edition). Version 1903 or higher, with Build 18362 or higher.

or

* Windows 11 Pro or Enterprise (64-bit edition).

and

* [Windows Terminal](https://www.microsoft.com/store/productid/9N0DX20HK701) installed.
* [Ubuntu 22.04.2](https://www.microsoft.com/store/productid/9PN20MSR04DW?ocid=pdpshare) installed.
* [WSL 2](https://docs.microsoft.com/en-us/windows/wsl/install-win10) installed and running.
* [NVIDIA GPU Driver](https://www.nvidia.com/Download/index.aspx?lang=en-us) installed.
* [Docker Desktop](https://www.docker.com/products/docker-desktop) installed and running.


## How to install


## Step 1. Turn on WSL

Use windows search to find "Turn Windows features on or off" and open it.

![Turn Windows features on or off](https://github.com/supervisely/developer-portal/assets/48913536/c25b3ddb-af4c-4066-9037-c1c7bb77c171)

Scroll down and locate "Windows Subsystem for Linux" and check the box.

![Windows Subsystem for Linux](https://github.com/supervisely/developer-portal/assets/48913536/8afd1be8-f1b0-4bf8-8a26-3102449a7a7d)

Restart your computer.

## Step 2. Install Windows Terminal

![Windows Terminal](https://github.com/supervisely/developer-portal/assets/48913536/4be351b1-aed7-4b71-af9f-bc5c743689d9)

## Step 3. Install Ubuntu

Open Microsoft Store and find Ubuntu 22.04.2 and press Get.

![Ubuntu 22.04.2](https://github.com/supervisely/developer-portal/assets/48913536/4be2475e-acbd-4cd6-80aa-04eda2394d49)

## Step 4. Install NVIDIA GPU Driver

Go to [NVIDIA](https://www.nvidia.com/Download/index.aspx?lang=en-us) site and download the latest driver for your GPU.

Fill the form and press Search.

![NVIDIA Search](https://github.com/supervisely/developer-portal/assets/48913536/5b37a6a8-7340-45e7-9166-905e0a28a0a0)

Press download and install the driver.

![NVIDIA Download](https://github.com/supervisely/developer-portal/assets/48913536/35cc54d9-096e-4217-9514-43e173051315)

## Step 5. Docker Desktop

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) and install it.
Open Docker Desktop and go to Settings -> Resources -> WSL integration.

![Docker Desktop Settings](https://github.com/supervisely/developer-portal/assets/48913536/c89cab0a-b74c-4715-8a69-8d1f1fbde256)

Enable integration with my default WSL distro and check the box for Ubuntu 22.04 and press Apply & Restart as shown above.

## Step 6. Install NVIDIA Container Toolkit

Install [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#step-1-install-nvidia-container-toolkit) repository for your distribution by running the following command:

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Update the APT repository cache and install the `nvidia-container-toolkit` package:

```bash
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
```

Restart Docker Desktop.

Enter the following command to verify that the installation was successful:

```bash
sudo docker run --rm --runtime=nvidia --gpus all nvidia/cuda:12.2.0-runtime-ubuntu22.04 nvidia-smi
```

Docker image will be pulled and you will see nvidia-smi output.

![NVIDIA SMI](https://github.com/supervisely/developer-portal/assets/48913536/ec23d667-a068-46fd-b36c-cd7ed24d1018)

Check nvidia runtime with this command:

```bash
cat /etc/docker/daemon.json
```

Output:

```json
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

**If it is not there, copy above json snippet and paste it into /etc/docker/daemon.json file.**

## Step 7. Deploy Supervisely Agent

Deploy Supervisely Agent with GPU support on Windows WSL.

<a href="https://youtu.be/aO7Zc4kTrVg">
    <img src="https://github.com/supervisely/developer-portal/assets/48913536/7f29248f-3573-4e11-ac5b-651f8c9ea14b" style="max-width:100%;">
</a>
