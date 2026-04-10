# AWS EKS

Everything you need to know about deploying Supervisely agent on an AWS EKS cluster.

## Deploy Supervisely Agent on AWS EKS

Supervisely can use an Amazon Elastic Kubernetes Service (EKS) cluster as a Kubernetes backend for running workloads. This guide explains how to create a minimal EKS cluster, deploy the Supervisely bootstrap resources, and connect the cluster in the Supervisely UI.

The example in this document uses the following configuration:

- AWS region: `us-east-1`
- Cluster name: `supervisely-eks`
- Kubernetes version: `1.35`
- One managed node group with `t3.large`
- Public worker nodes
- NAT gateway disabled to reduce baseline cost

This configuration is suitable for testing and initial integration. For production environments, review networking, ingress, DNS, TLS, node sizing, scaling, monitoring, and security requirements.

## Table of Contents

- Prerequisites
- How it works
- Step 1. Verify required tools
- Step 2. Configure AWS access
- Step 3. Create the EKS cluster
- Step 4. Verify cluster access
- Step 5. Generate the Supervisely Kubernetes configuration
- Step 6. Deploy Supervisely bootstrap resources
- Step 7. Get connection values from EKS
- Step 8. Connect the cluster in Supervisely
- Step 9. Verify the deployment
- Cleanup
- Troubleshooting

## Prerequisites

- AWS account that is allowed to launch EC2 instances
- AWS permissions for EKS, IAM, CloudFormation, EC2, VPC, Auto Scaling, and public SSM parameters
- `aws`, `kubectl`, and `eksctl` installed locally
- Access to the Supervisely instance
- Bash or another POSIX-compatible shell environment with AWS credentials configured

If you plan to run GPU workloads, use a GPU-capable node group instead of `t3.large` and enable GPU capability later in the Supervisely UI.

## How It Works

The EKS cluster provides the Kubernetes control plane and worker nodes.

The Kubernetes configuration generated in the Supervisely UI creates the resources required for Supervisely to work with the cluster:

- `sly-task-manager` service account and RBAC
- `sly-task-watcher` service account and RBAC
- `sly-task-manager-token` secret
- Image registry secret for Supervisely images

After these resources are created, read the service account token, Kubernetes API endpoint, and certificate authority data, and then enter those values in the Supervisely UI.

## Step 1. Verify Required Tools

Run the following commands to verify that the required tools are available:

```bash
aws --version
kubectl version --client
eksctl version
```

If any command is missing, install the tool before proceeding.

## Step 2. Configure AWS Access

Configure AWS credentials using either an AWS profile or environment variables.

Example using environment variables in bash:

```bash
export AWS_ACCESS_KEY_ID="<access-key-id>"
export AWS_SECRET_ACCESS_KEY="<secret-access-key>"
export AWS_DEFAULT_REGION="us-east-1"
export AWS_PAGER=""
```

Verify access:

```bash
aws sts get-caller-identity
```

The command must return the active AWS identity.

Important: successful AWS authentication is not enough on its own. The AWS account must also be allowed to launch EC2 instances, otherwise EKS worker nodes will not start.

## Step 3. Create the EKS Cluster

Use the following `eksctl` configuration for a minimal test cluster:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: supervisely-eks
  region: us-east-1
  version: "1.35"

autoModeConfig:
  enabled: false

accessConfig:
  authenticationMode: API_AND_CONFIG_MAP

iam:
  withOIDC: true

vpc:
  nat:
    gateway: Disable
  clusterEndpoints:
    publicAccess: true
    privateAccess: false

managedNodeGroups:
  - name: general
    instanceType: t3.large
    amiFamily: AmazonLinux2023
    desiredCapacity: 1
    minSize: 1
    maxSize: 2
    volumeType: gp3
    volumeSize: 30
    privateNetworking: false
    disableIMDSv1: true
```

Create the cluster:

```bash
eksctl create cluster -f ./eksctl-supervisely-cluster.yaml
```

Expected duration: 15 to 30 minutes.

This command creates:

- The EKS control plane
- VPC networking for the cluster
- IAM resources required by EKS
- One managed node group
- Local kubeconfig entry for the cluster

## Step 4. Verify Cluster Access

After cluster creation completes, run:

```bash
kubectl get nodes
kubectl get namespaces
```

Expected result:

- At least one node is in `Ready` state
- The API server responds to `kubectl`

Immediately after control plane creation, `kubectl get nodes` may temporarily return `No resources found` while the node group is still provisioning. Wait a few minutes and retry.

## Step 5. Generate the Supervisely Kubernetes Configuration

Open the Supervisely instance and go to:

`Cluster -> Connect -> Kubernetes`

This page contains the Kubernetes connection form and the generated deployment command.

Create a new Kubernetes connection and copy the generated deployment command or the generated Kubernetes configuration URL.

The command has the following form:

```bash
curl -fsSLg "<generated-kubernetes-config-url>" | kubectl -n supervisely apply -f -
```

Do not submit the form yet. The required token and cluster values will be collected in the next steps.

The following values come directly from the Supervisely UI:

- Agent name
- Generated Kubernetes configuration URL
- Generated deployment command
- The Kubernetes connection form itself

## Step 6. Deploy Supervisely Bootstrap Resources

Create the target namespace:

```bash
kubectl create namespace supervisely
```

Apply the generated manifest:

```bash
curl -fsSLg "<generated-kubernetes-config-url>" | kubectl -n supervisely apply -f -
```

Expected result:

- `sly-task-manager` service account is created
- `sly-task-watcher` service account is created
- RBAC roles and bindings are created
- `sly-task-manager-token` secret is created
- Image registry secret is created

You can verify the resources with:

```bash
kubectl -n supervisely get serviceaccounts
kubectl -n supervisely get secrets
kubectl -n supervisely get roles,rolebindings
```

## Step 7. Get Connection Values from EKS

The Kubernetes connection form in Supervisely contains fields that must be filled from different sources.

Use the following mapping:

| Field in Supervisely UI | Source | Notes |
| --- | --- | --- |
| Agent name | Supervisely UI | Set in the `Cluster -> Connect -> Kubernetes` dialog |
| Supervisely server address | Supervisely instance | Use the base URL of the current Supervisely instance |
| Service account token | Kubernetes secret | Read from `sly-task-manager-token` after applying the manifest |
| Kubernetes API endpoint | AWS EKS | Read with `aws eks describe-cluster` |
| Certificate Authority | AWS EKS or Kubernetes secret | AWS output can be used directly |
| Namespace | Supervisely UI / Kubernetes | Use `supervisely` |
| Ingress address | Environment-specific | Leave empty unless ingress is already configured |
| GPU capability of the cluster | Deployment choice | Enable only for GPU-capable node groups |
| Use kubernetes services instead of ingress | Deployment choice | Leave disabled for normal EKS deployment |
| Ingress manifest for Apps routing | Environment-specific | Configure later when ingress is installed |

### Supervisely Server Address

Use the base URL of the Supervisely instance.

Example:

```text
https://your-instance-address.com
```

### Service Account Token

Read the token from the Kubernetes secret and decode it:

```bash
kubectl -n supervisely get secret sly-task-manager-token -o jsonpath='{.data.token}' | base64 --decode
```

If the token is empty immediately after applying the manifest, wait a few seconds and retry.

### Kubernetes API Endpoint

Read the EKS cluster endpoint:

```bash
aws eks describe-cluster --region us-east-1 --name supervisely-eks --query "cluster.endpoint" --output text
```

### Certificate Authority

Read the EKS cluster certificate authority data:

```bash
aws eks describe-cluster --region us-east-1 --name supervisely-eks --query "cluster.certificateAuthority.data" --output text
```

The value returned by AWS is base64-encoded and can be used directly in the Supervisely UI.

If needed, the same value can also be read from the Kubernetes secret:

```bash
kubectl -n supervisely get secret sly-task-manager-token -o jsonpath='{.data.ca\.crt}'
```

### Namespace

Use the following value:

```text
supervisely
```

### Optional UI Fields

Use the following defaults unless the cluster is already configured differently:

- Ingress address: leave empty
- GPU capability of the cluster: enable only for GPU-capable node groups
- Use Kubernetes services instead of ingress: leave disabled unless Supervisely itself runs inside the same cluster
- Ingress manifest for apps routing: configure later when ingress is installed

## Step 8. Connect the Cluster in Supervisely

Fill the Kubernetes connection form in Supervisely with the following values:

- Supervisely server address: Supervisely base URL
- Service account token: decoded token from `sly-task-manager-token`
- Kubernetes API endpoint: value from `aws eks describe-cluster`
- Certificate Authority: base64 CA value from `aws eks describe-cluster`
- Namespace: `supervisely`

Then save the connection.

Navigation path:

`Cluster -> Connect -> Kubernetes`

## Step 9. Verify the Deployment

Expected result:

- The cluster appears as connected in Supervisely
- Supervisely can create resources in the `supervisely` namespace

Recommended verification steps:

```bash
kubectl -n supervisely get serviceaccounts
kubectl -n supervisely get secrets
kubectl -n supervisely get roles,rolebindings
```

After the cluster is connected, launch a simple workload from Supervisely and verify that Kubernetes resources are created successfully.

## Cleanup

Delete the cluster when it is no longer needed:

```bash
eksctl delete cluster --name supervisely-eks --region us-east-1
```

The example configuration creates billable AWS resources. Remove the cluster after testing to avoid unnecessary charges.

## Troubleshooting

### `aws sts get-caller-identity` fails

The AWS credentials are missing or invalid. Reconfigure AWS access and retry.

### `RunInstances` returns `Blocked`

This is an AWS account-level restriction. The IAM user may be valid, but the account is not currently allowed to launch EC2 instances.

Resolve the account restriction with the AWS account owner or AWS Support before retrying EKS creation.

### Managed node group stays in `CREATING`

If the control plane is healthy but worker nodes do not appear, inspect:

- EKS node group status
- Auto Scaling activities
- CloudFormation events for the node group stack

Common causes include EC2 quota limits, blocked EC2 instance launches, or account verification restrictions.

### `kubectl get nodes` returns `No resources found`

The control plane may be ready while the node group is still provisioning. Wait a few minutes and run the command again.

### `kubectl get nodes` fails after cluster creation

Refresh the kubeconfig entry and retry:

```bash
aws eks update-kubeconfig --region us-east-1 --name supervisely-eks
kubectl get nodes
```

### `kubectl -n supervisely apply -f -` fails because the namespace does not exist

Create the namespace first:

```bash
kubectl create namespace supervisely
```

### `sly-task-manager-token` exists but the token is empty

Wait a few seconds and retry the token command. Kubernetes can populate service account token secrets asynchronously.

### Images cannot be pulled in the cluster

The Supervisely manifest creates the image registry secret, but worker nodes still need outbound internet access to pull images.

The example configuration uses public worker nodes. If you move to private worker nodes, add the required egress design before deploying workloads.
