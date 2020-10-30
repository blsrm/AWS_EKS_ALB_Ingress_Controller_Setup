# AWS EKS ALB Ingress Controller Setup
This document provides detailed step by step procedure for creating EKS Fargate ALB Ingress Controller

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


