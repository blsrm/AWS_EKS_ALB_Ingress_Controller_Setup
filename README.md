# AWS EKS ALB Ingress Controller Setup
This document provides detailed step by step procedure for creating EKS Fargate ALB Ingress Controller. I want to set up the Application Load Balancer (ALB) Ingress Controller on an Amazon Elastic Kubernetes Service (Amazon EKS) cluster for AWS Fargate.

## Short description
Before completing the steps in the Resolution section, consider the following:

- Indent YAML files correctly using spaces and not tabs.
- Replace placeholder values with your own values when you see <placeholder-value> in a command.
- Be aware that the --region variable isn't used in any of the following commands, because the default value for your AWS Region is used. To check the default value, run the aws configure command. To change the AWS Region, use the -region flag.
- Be aware that Amazon EKS for Fargate is available only in the following AWS Regions: US East (N. Virginia), US East (Ohio), Europe (Ireland), Europe (Frankfurt), and Asia Pacific (Tokyo).
- Use eksctl version 0.11.1 or greater.


## Resolution

### Create an Amazon EKS cluster, service account policy, and RBAC policies

1. To use eksctl to create an Amazon EKS cluster for Fargate, run the following command:

  ```
  $ eksctl utils associate-iam-oidc-provider --cluster your-cluster-name --approve
  ```

2. To allow the cluster to use AWS Identity and Access Management (IAM) for service accounts, run the following command:

```
$ eksctl utils associate-iam-oidc-provider --cluster your-cluster-name --approve
```
**Note**: The **FargateExecutionRole** is the role that the **kubelet** and **kube-proxy** run your Fargate pod on, but it's not the role for the Fargate pod (that is, the **alb-ingress-controller**). For the Fargate pod, you must use the IAM role for the service account.

3. Create an IAM policy for the service account using the correct [permissions](https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json) from the Kubernetes GitHub website, and note the Amazon Resource Name (ARN) of the IAM policy.

**Note**: The ALB Ingress Controller requires several API calls to provision the ALB components for the ingress resource type.


Step1: Deploy the relevant RBAC roles and role bindings as required by the AWS ALB Ingress controller
 
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
```






# Reference Links

### AWS ALB
https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/master/docs/guide/walkthrough/echoserver.md
https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/
https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html

### NGINXC
https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/


