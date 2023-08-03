---
description: >-
  Distil knowledge from large ML models to lightweight, fast and easy to deploy ML models using Supervisely.
---

# Knowledge distillation in Supervisley Ecosystem

## What is knowledge distillation?

Knowledge distillation is a technique in machine learning aimed at transferring knowledge from heavy, slow ML model to a lighter, faster one. Large ML models usually provide high prediction accuracy, but many of them have low speed of inference and it can be difficult to integrate such models in real world applications because of their high time and space complexity - the resources of devices are limited. At the same time there are many lightweight and fast ML models, what makes them easy to deploy, but these models usually demonstrate significantly lower prediction accuracy than large ML models.

The goal of knowledge distillation is to take strong sides of both types of models and neutralize their weaknesses. The main idea of knowledge distillation is to make lightweight model (student) imitate behaviour of large model (teacher) during training. There are many variations of knowledge distillation. The most common classifications are hard / soft and offline / online knowledge distillation.

Soft knowledge distillation assumes adding a summand to a student model's original loss function, this summand will contain similarity loss between teacher model's and student model's outputs (logits):

![soft_distillation](https://user-images.githubusercontent.com/91027877/258198611-ba3a469e-562b-4177-8eec-35ff77a2e6fe.png)

Hard knowledge distillation is completely similar to soft one except the fact that similarity between teacher and student models' predictions (predicted classes) will be calculated:

![hard_distillation](https://user-images.githubusercontent.com/91027877/258198935-57e30ea9-4fe5-4157-9e8f-e591cd09bae8.png)

Speaking about offline / online knowledge distillation, offline variant assumes usage of pretrained teacher model to guide the student model, while online variant assumes that teacher and student models will be trained simultaneously in one session (this option is useful when pretrained teacher model is not available).

In this tutorial we are not going to use any of the above variations in its pure form. Instead of this we will take offline knowledge distillation as a basis and combine it with another technique called pseudo-labeling.

## Pseudo-labeling with OWL-ViT

Pseudo-labeling involves usage of pretrained ML model to predict labels on the dataset and then use these labels (called pseudo-labels) to train model in them. We will combine the idea of transferring knowledge from large ML model to lighter one and usage of pretrained models for data labeling.

[OWL-ViT](https://github.com/google-research/scenic/tree/main/scenic/projects/owl_vit) is a perfect example of large foundation ML model since it can detect almost any object using reference image mode without additional training (see full tutorial [here](https://supervisely.com/blog/owl-vit/)) - we will take it as a teacher model. But OWL-ViT is pretty slow and heavy, so it will be difficult to deploy it in production. [YOLOv8](https://github.com/ultralytics/ultralytics), on the contrary, demonstrates high speed of inference, it is quite light and easy to deploy (you can learn about YOLOv8 Supervisely integration [here](https://supervisely.com/blog/train-yolov8-on-custom-data-no-code/)) - that's why we will use it as a student model. We will take OWL-ViT as a teacher model, use it to create pseudo-labels on our dataset, then train YOLOv8 as a student model on pseudo-labels, created by OWL-ViT. In that way we will transfer a part of OWL-ViT knowledge to YOLOv8 and receive a fast, lightweight and easy to deploy machine learning model for our specific task without labeling our data - we will use OWL-ViT to label data automatically instead.

## Step 1. Create pseudo-labels for your dataset using OWL-ViT

Run [Apply OWL-ViT to Images Porject](https://ecosystem.supervisely.com/apps/apply-owl-vit-to-images-project) app from Supervisely Ecosystem, select reference image and preview model predcitions:

Once predictions preview looks satisfying, you can start autolabeling your dataset with OWL-ViT (here is full [video-guide](https://www.youtube.com/watch?v=PnhAsG-GFHo&t=344s)).

## Step 2. Split your dataset on train, validation and test set

Now, when we got pseudo-labels from OWL-ViT, it is necessary to split our dataset into 3 subsets. You can do it with the help of [Split Datasets](https://ecosystem.supervisely.com/apps/split-dataset) app:

## Step 3. Train YOLOv8 on predictions of OWL-ViT

Now we can easily train YOLOv8 using corresponding [application](https://ecosystem.supervisely.com/apps/yolov8/train) (here is full [video guide](https://www.youtube.com/watch?v=Rsr8xWJ6s9I&t=457s)):

## Step 4. Test YOLOv8 model with knowledge distilled from OWL-ViT

Now, when we our YOLOv8 model is trained, we can deploy it as REST API service by using correspong [app](https://ecosystem.supervisely.com/apps/yolov8/serve) and test its performance on unseen data:

## Summary

It this tutorial we learned how to combine knowledge distillation and pseudo-labeling techniques to successfully train a robust machine learning model that will not require significant memory and computational resources from your device without having to manually label your data. This approach is very flexible and can be used for numerous computer vision tasks.