---
description: >-
  Use Command Line Interface for managing the new CLI and its daemon. Automate the initialization and upgrading processes effortlessly.
---

{% hint style="warning" %}
### Beta. Release coming soon.
{% endhint %}

# Introduction

This documentation guides you through the usage of the new Supervisely CLI tool, designed to manage your Supervisely instance.
CLI consists of two parts: the CLI itself and the daemon. The CLI is a command-line tool that you can use to manage the daemon. The daemon is a system service that runs on your machine and performs the necessary actions on your instance.

{% hint style="info" %}
The CLI commands provided here are subject to updates. If you have specific functionalities you'd like to see or suggestions for improvements, feel free to reach out to the development team.
{% endhint %}

## Downloading and Installation

Placeholder Text

![Placeholder Image]()

## Prerequisites

First, make sure you don't have the Supervisely SDK installed globally. In the other case, there may be a conflict in the namespace.

```bash
pip show supervisely
WARNING: Package(s) not found: supervisely
```

☝️ Remember that it is always better to work in a virtual environment in which the SDK is installed. Therefore, it is recommended to remove Supervisely SDK from the global environment and install it in the virtual environment.

Then check that you have the latest version of the CLI package installed.

```bash
supervisely version
 Supervisely CLI 2.0.57 is up to date
 # If you have an older version installed, you will see the following
 Supervisely CLI 2.0.54
 Version 2.0.57 is out! Upgrade now for the latest features. 
 Run 'supervisely self-update' in your terminal.
```

If you decided to update to the latest version run the following command.

```bash
supervisely self-update
 Superviselyd updated to 2.0.24
 Downloading latest CLI version: 100%|████████████████| 231M/231M [00:00<00:00, 770MB/s]
 Supervisely CLI updated to 2.0.57
```
Now you're ready to use the CLI. Refer to the following sections for specific commands.

# CLI Tool Modules
1. [**Enterprise module 🏢**](./enterprise.md)
2. [**Instance interaction module 💻**](./instance.md)