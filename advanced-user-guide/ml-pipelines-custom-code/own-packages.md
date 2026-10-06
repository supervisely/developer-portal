---
description: Run the Custom Code node with your own Python packages, including a private package index
---

# Your own packages

A Custom Code script can import anything installed in the app's Docker image. To use your own packages, and to turn the node on, you build your own image of the ML Pipelines app and release your copy of the app as a [private app](../../app-development/basics/add-private-app.md). Supervisely does not build or maintain these images for you.

The steps:

1. Fork the app's repository.
2. Build an image with your packages and push it to a registry your agents can pull from.
3. Set the image in `config.json` and release the fork as a private app.

## 1. Fork the app

Fork [supervisely-ecosystem/data-nodes](https://github.com/supervisely-ecosystem/data-nodes). Keep the changes in your fork to the image and `config.json`: then taking our updates is a simple rebase.

In `config.json`, give your copy its own name, so your team can tell it from the app in the Ecosystem:

```json
"name": "ML Pipelines (Custom Code)",
```

## 2. Build the image

Start from the app's image and add your packages. The version in `FROM` is the `docker_image` tag in the app's `config.json`.

```dockerfile
FROM supervisely/data-nodes:6.74.7

# Your private index is used only while building, passed as a BuildKit secret: it is not stored
# in the image or its history.
RUN --mount=type=secret,id=pip_index_url \
    /opt/venv/bin/python -m ensurepip && \
    /opt/venv/bin/python -m pip install --no-cache-dir \
        --index-url "$(cat /run/secrets/pip_index_url)" \
        --extra-index-url https://pypi.org/simple \
        "your-package==1.2.3" "numpy<2"

# Turns the Custom Code node on.
ENV ML_PIPELINES_CUSTOM_CODE=1
```

The app's image has no `pip` (it is removed to keep the image small), so `ensurepip` installs it first.

Build and push it. For example, with a private index in AWS CodeArtifact and an image in Amazon ECR:

```bash
TOKEN=$(aws codeartifact get-authorization-token --domain my-domain --domain-owner 123456789012 \
    --query authorizationToken --output text)
ENDPOINT=$(aws codeartifact get-repository-endpoint --domain my-domain --domain-owner 123456789012 \
    --repository my-repo --format pypi --query repositoryEndpoint --output text)
echo "https://aws:${TOKEN}@${ENDPOINT#https://}simple/" > pip_index_url

docker build --secret id=pip_index_url,src=pip_index_url \
    -t 123456789012.dkr.ecr.eu-west-1.amazonaws.com/ml-pipelines-custom:1.0.0 .
rm pip_index_url
docker push 123456789012.dkr.ecr.eu-west-1.amazonaws.com/ml-pipelines-custom:1.0.0
```

Keep these constraints of the app:

- **Python 3.12.** Your packages must support it.
- **`numpy<2`.** The app's augmentation library (imgaug) does not work with NumPy 2. Keep the pin in the same `pip install` command, so pip resolves your packages against it.

To rebuild the image from scratch instead, add your packages to `dev_requirements.txt` and build `docker/Dockerfile.tmpl` from the repository. A private index then has to be available in its `requirements-builder` stage.

## 3. Set the image and release

Set your image in the fork's `config.json`:

```json
"docker_image": "123456789012.dkr.ecr.eu-west-1.amazonaws.com/ml-pipelines-custom:1.0.0",
```

Every agent that runs the app must be able to pull this image: log the agent's machine in to your registry (for ECR, `aws ecr get-login-password | docker login ...` or the ECR credential helper).

Then release the fork as a private app on your instance, as [Add private app](../../app-development/basics/add-private-app.md) describes:

```bash
supervisely release
```

See [config.json](../../app-development/basics/app-json-config/config.json.md) for the other fields.

## Choosing where it runs

Launch your private app on any agent of the team. The whole session runs there, and its videos are processed in parallel on that agent's cores ([Parallel processing and where it runs](README.md#parallel-processing-and-where-it-runs)). For example, connect an on-demand worker in your cloud as an agent and launch the app on it, so heavy processing does not run on the machine that serves your instance.

## Check that it works

1. Launch your app on a video project. **Custom Code** is listed under **Video transforms**. If it is not, the image does not set `ML_PIPELINES_CUSTOM_CODE=1`.
2. Run a script that imports your package:

```python
import your_package


def process(video_path, video_info, ann, params):
    print(your_package.__version__)
    return [(0, video_info.frames_count - 1)]
```

The version is printed to the app log for every video, and the videos pass through unchanged.
