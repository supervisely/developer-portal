---
description: >-
  Distil knowledge from large ML models to lightweight, fast and easy to deploy ML models using Supervisely.
---

# Offline knowledge distillation in Supervisley Ecosystem

## What is knowledge distillation?

Knowledge distillation is a technique in machine learning aimed at transferring knowledge from heavy, slow ML model to a lighter, faster one. Large ML models usually provide high prediction accuracy, but many of them have low speed of inference and it can be difficult to integrate such models in real world applications because of their high time and space complexity - the resources of devices are limited. At the same time there are many lightweight and fast ML models, what makes them easy to deploy, but these models usually demonstrate significantly lower prediction accuracy than large ML models.

The goal of knowledge distillation is to take strong sides of both types of models and neutralize their weaknesses. The main idea of knowledge distillation is to make lightweight model (student) imitate behaviour of large model (teacher) during training. There are many variations of knowledge distillation. The most common classifications are hard / soft and offline / online knowledge distillation.

Soft knowledge distillation assumes adding a summand to a student model's original loss function, this summand will contain similarity loss between teacher model's and student model's outputs (logits):

Hard knowledge distillation is completely similar to soft one except the fact that similarity between teacher and student models' predictions (predicted classes) will be calculated:

Speaking about offline / online knowledge distillation, offline variant assumes usage of pretrained teacher model to guide the student model, while online variant assumes that teacher and student models will be trained simultaneously in one session (this option is useful when pretrained teacher model is not available).
