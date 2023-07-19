---
description: >-
  Guide explains how to train and deploy Neural Networks in Supervisely
---

# Neural Networks training and deployment in Supervisely

## Scheme 1: Data and Hardware communication diagram

![Scheme 1 Data and Hardware communication diagram](https://user-images.githubusercontent.com/48245050/233949624-3354327c-94bc-4557-827f-86d19ff01d4a.jpg)

The typical Supervisely setup for neural network training and deployment consists of a main VM or server on which the Supervisely instance software is installed. The installation is done using Docker containers managed by Docker Compose and Supervisely CLI.

Supervisely instance can store persistent data such as training weights or configuration on local disk drive or be configured to use a number of cloud providers such as AWS S3 as a primary data storage.

There is also one or more VMs or servers equipped with GPU devices connected to the Supervisely instance via Supervisely Agent — special software component that connects to the Supervisely instance via API and can be used to run tasks, such as neural network training or serving. Both Supervisely Agent and Tasks that it can run are Docker containers. Both Supervisely Agent and Tasks do not use its host VM as a primary data storage: both components connect to the main Supervisely instance via API and pull / push data artifacts, such as training weights to the primary storage of the Supervisely instance (local or cloud).

## Scheme 2: Neural Networks training pipeline

![Scheme 2 Neural Networks training pipeline](https://user-images.githubusercontent.com/48245050/233949903-b4444e61-d93b-4a8c-9279-fa33b0e2c90f.jpg)

Start by preparing a Training Data and creating a corresponding Project in Supervisely. Run a Supervisely App from Supervisely Ecosystem such as Train YOLO application. Supervisely App will be deployed on the selected Supervisely Agent, connect to the Supervisely instance via API and push training artifacts such as training weights.

Run a Serving App such as Serve YOLO application on the same or a different Supervisely Agent. The App will connect to the Supervisely instance via API and pull required training artifacts from the storage.

Use the Serving App via API, SDK, or run another Supervisely App that can connect to Serving Apps, such as Apply NN to Images Project to apply the trained model to every image in the specified Supervisely Project.

## Scheme 3: Use Supervisely as backend (MLOps platform) for organizing custom data - neural network pipelines

![Scheme 3 Use Supervisely as backend](https://github.com/supervisely/developer-portal/assets/79905215/e29e68e7-4312-4a62-817e-7e449b904d38)

In the example above we create the following pipeline:
1. Import new data from customer storage infrastructure to Supervisely using API, SDK or
manually
2. Trigger deployed neural network models to obtain annotated images
3. Automatically run code to either send annotated images back to customer infrastructure or
create new labeling jobs and send notifications to the labeling team for the results to be
manually corrected
