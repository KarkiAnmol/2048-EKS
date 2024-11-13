
# Setting up AWS Load Balancer (ALB) Controller for EKS

The ALB controller manages application load balancers for your EKS cluster. Follow these steps to set up the ALB controller using IAM policies and Helm.

## Steps

### 1. Download IAM Policy
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
### 2. Create IAM Policy
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
### 3. Create IAM Role

Replace <your-cluster-name> and <your-aws-account-id> with your specific cluster name and AWS account ID:
```bash
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
![Screenshot from 2024-11-13 07-56-13](https://github.com/user-attachments/assets/c63c0244-0a53-4259-a686-811c89fca585)

### 4. Deploy ALB Controller using Helm

- Add Helm repo:
```bash
helm repo add eks https://aws.github.io/eks-charts
```
- Update Helm repo:
```bash
helm repo update eks
```
- Install ALB Controller: Replace <your-cluster-name>, <region>, and <your-vpc-id> with your cluster name, AWS region, and VPC ID.
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
      -n kube-system \
      --set clusterName=<your-cluster-name> \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller \
      --set region=<region> \
      --set vpcId=<your-vpc-id>
```
![Screenshot from 2024-11-13 08-14-07](https://github.com/user-attachments/assets/1c7ce92d-4a2c-4c3d-80f5-5c555255622a)


### 5. Verify Deployment

Check if the ALB controller deployment is running:
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
![Screenshot from 2024-11-13 08-16-33](https://github.com/user-attachments/assets/dda32b54-04b3-415b-85d6-e1c8924dcbc7)
![Screenshot from 2024-11-13 08-29-26](https://github.com/user-attachments/assets/4a7db316-d4ac-424b-b045-e302d0482201)

The ALB controller should now be active, continuously monitoring ingress resources to create and manage load balancers.
