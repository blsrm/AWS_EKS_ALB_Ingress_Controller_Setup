# AWS EKS ALB Ingress Controller Setup
This document provides detailed step by step procedure for creating EKS Fargate ALB Ingress Controller

Step1: Deploy the relevant RBAC roles and role bindings as required by the AWS ALB Ingress controller
 
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
```
