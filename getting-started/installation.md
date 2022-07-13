---
description: Everything you need to know about installation of Supervisely SDK for Python
---

# Installation

This part of the documentation covers the installation of Supervisely SDK for Python. The first step to using any software package is getting it properly installed.

## Prerequisites

### Python

You should use **Python 3.7 or greater**, which can be installed either through the [Anaconda](https://www.anaconda.com/products/distribution) package manager, [Homebrew](https://brew.sh/), or the [Python website](https://www.python.org/downloads/mac-osx/).

### Libraries

```bash
apt-get update
apt-get install ffmpeg libgeos-dev libsm6 libxext6 libexiv2-dev libxrender-dev libboost-all-dev -y
```

## Installation

### Pip

The latest stable version [is available on PyPI](https://pypi.org/project/supervisely/). Either add `supervisely` to your `requirements.txt` file or install with pip:

```
pip install supervisely
```

### Source code

Supervisely is actively developed on GitHub, where the code is [always available](https://github.com/supervisely/supervisely).

You can either clone the public repository:

```
git clone https://github.com/supervisely/supervisely.git
```

Or, download the [zipball](https://github.com/supervisely/supervisely/archive/refs/heads/master.zip):

```
$ curl -OL https://github.com/supervisely/supervisely/archive/refs/heads/master.zip
```

Once you have a copy of the source, you can embed it in your own Python package, or install it into your site-packages easily:

```
unzip master.zip
cd supervisely-master
python -m pip install .
```

## Docker image

Supervisely SDK for python also has prebuilt [docker image](https://hub.docker.com/r/supervisely/base-py-sdk) with everything already installed.

You can use the latest version

```
docker pull supervisely/base-py-sdk:latest
```

or some specific on that has completely the same tag as [PIP releases](https://pypi.org/project/supervisely/), for example:

```
docker pull supervisely/base-py-sdk:6.33.0
```

Here are the links to dockerfiles ([base image](https://github.com/supervisely/supervisely/blob/master/base\_images/py/Dockerfile), [result image](https://github.com/supervisely/supervisely/blob/master/base\_images/py\_sdk/Dockerfile)) where you can find the complete list of all recommended dependencies.
