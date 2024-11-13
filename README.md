# Deploying the 2048 Game on AWS EKS with Ingress
![Screenshot from 2024-11-13 08-36-02](https://github.com/user-attachments/assets/e703205e-4aa0-42ee-9731-bd827493e26a)


## Prerequisites

Ensure you have the following tools installed:
- **kubectl**: [Install kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- **eksctl**: [Install EKSctl](https://eksctl.io/installation/#)
- **AWS CLI**: [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

## 1. Setting up Your AWS Environment for EKS

### 1.1 Creating an AWS Account and Setting up IAM Users
Follow these steps:
- Create an AWS Account on [AWS Website](https://aws.amazon.com/).
- Verify your account, log in, and set up IAM users with necessary permissions.

### 1.2 Configuring AWS CLI and kubectl
Once IAM is set up:
- Configure AWS CLI:
  ```bash
  aws configure
  ```
- Configure kubectl:
  ```bash
  aws eks --region <your-region> update-kubeconfig --name <your-cluster-name>
  ```


## 2. EKS Cluster and Fargate Setup
### 2.1 Creating an EKS Cluster on Fargate

Create the cluster using:
```bash
eksctl create cluster --name <your-cluster-name> --region <your-region-name> --fargate
```
![Screenshot from 2024-11-12 09-35-38](https://github.com/user-attachments/assets/f7960d01-05e1-4f59-b369-84b27fa8f295)

### 2.2 Configuring kubectl Context

To switch kubectl to your EKS cluster:
```bash
kubectl config get-contexts
kubectl config use-context <eks-context-name>
```
Check the node status:
```bash
kubectl get nodes
```
![Screenshot from 2024-11-12 09-54-10](https://github.com/user-attachments/assets/77bf0ea0-74ef-4d9f-80a8-3168f72a57cc)

## 3. Creating Fargate Profile and Deploying 2048 Game
### 3.1 Create Fargate Profile for the game-2048 Namespace
```bash
eksctl create fargateprofile \
  --cluster <your-cluster-name> \
  --region <your-region> \
  --name <your-app-name> \
  --namespace game-2048
```
![Screenshot from 2024-11-12 10-38-54](https://github.com/user-attachments/assets/10e3088c-4354-42a9-9d25-61c4c1ac0828)


### 3.2 Deploying 2048 Game Application with Ingress
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```
Monitor pod creation:
```bash
kubectl get pods -n game-2048 -w
```
![Screenshot from 2024-11-13 06-52-22](https://github.com/user-attachments/assets/5ae54361-bb6d-417a-ba81-f22c96a7e944)

Check service status:
```bash
kubectl get svc -n game-2048
```
![Screenshot from 2024-11-13 07-19-12](https://github.com/user-attachments/assets/e78ecf55-12ae-4f9b-bc91-1b5db003af66)


## 4. ALB Controller and Ingress Setup
Initially, there is no Elastic IP address to access the app publicly, so we create an ALB ingress controller, with the OIDC connector setup as a prerequisite.
![Screenshot from 2024-11-13 07-23-02](https://github.com/user-attachments/assets/b2aed5d9-7b83-4957-bf70-715311829a72)

### 4.1 Configure IAM OIDC Provider

_refer to [`oidc-connecter-configuration.md`](oidc-connecter-configuration.md) for full IAM OIDC configuration commands._


### 4.2 Install AWS Load Balancer (ALB) Controller

Download IAM policy, create IAM role, and set up ALB using Helm.

_For all ALB controller setup commands, refer to [`alb-controller-add-on.md`](alb-controller-add-on.md)._

## 5. Verifying and Testing Deployment

To verify the ALB setup, check if the load balancer creation was successful:
```bash
kubectl get ingress -n game-2048
```
- Load balancer with assigned IP address: 
![Screenshot from 2024-11-13 08-35-38](https://github.com/user-attachments/assets/334af853-2a9d-4e04-b31c-589f07ade539)

- Accessing the 2048 game:

![Screenshot from 2024-11-13 08-36-02](https://github.com/user-attachments/assets/e703205e-4aa0-42ee-9731-bd827493e26a)
![Screenshot from 2024-11-13 08-35-15](https://github.com/user-attachments/assets/0407b93b-21b9-4549-82e1-f40d62fda4e9)


## 6. Cleanup

To avoid unnecessary charges, clean up resources after testing:
```bash
aws eks delete-fargate-profile --cluster-name your-cluster-name --fargate-profile-name your-fargate-profile-name --region your-region
aws eks delete-cluster --name your-cluster-name --region your-region
helm uninstall aws-load-balancer-controller -n kube-system
```

