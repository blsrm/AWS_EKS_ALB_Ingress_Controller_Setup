# AWS EKS ALB Ingress Controller Setup
This document provides detailed step by step procedure for creating EKS Fargate ALB Ingress Controller. I want to set up the Application Load Balancer (ALB) Ingress Controller on an Amazon Elastic Kubernetes Service (Amazon EKS) cluster for AWS Fargate.

## Short description
Before completing the steps in the Resolution section, consider the following:

Indent YAML files correctly using spaces and not tabs.
Replace placeholder values with your own values when you see <placeholder-value> in a command.
Be aware that the --region variable isn't used in any of the following commands, because the default value for your AWS Region is used. To check the default value, run the aws configure command. To change the AWS Region, use the -region flag.
Be aware that Amazon EKS for Fargate is available only in the following AWS Regions: US East (N. Virginia), US East (Ohio), Europe (Ireland), and Asia Pacific (Tokyo).
Use eksctl version 0.11.1 or greater.


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


