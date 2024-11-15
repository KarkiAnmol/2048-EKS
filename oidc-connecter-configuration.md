# Setting up IAM OIDC Provider for EKS

To enable the AWS Load Balancer Controller, you need to configure an IAM OIDC provider.

## Steps

1. **Set up IAM OIDC provider:**
    ```bash
    export cluster_name=demo-cluster
    oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
    ```
2. **Confirm or create the OIDC provider (if not found):**
     ```bash
     eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
     ```
     ![Screenshot from 2024-11-13 07-39-55](https://github.com/user-attachments/assets/45b9f156-902e-4182-9290-ad4d37c2b9cf)

This setup enables the cluster to use IAM roles with the AWS Load Balancer Controller.
