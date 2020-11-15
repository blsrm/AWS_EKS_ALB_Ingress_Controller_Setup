# AWS EKS ALB Ingress Controller Setup for Fargate

This document provides detailed step by step procedure for creating EKS Fargate ALB Ingress Controller. 

I want to set up the Application Load Balancer (ALB) Ingress Controller on an Amazon Elastic Kubernetes Service (Amazon EKS) cluster for AWS Fargate.

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

4. To create a service account, run the following command:

```
$ eksctl create iamserviceaccount --name your-service-account-name --namespace kube-system --cluster your-cluster-name --attach-policy-arn IAM-policy-arn --approve --override-existing-serviceaccounts
```
5. To verify that the new service role was created, run the following command:

```
$ eksctl get iamserviceaccount --cluster your-cluster-name --name your-service-account-name --namespace your-namespace
```
**Note**: The role name begins with **eksctl-<cluster-name>-addon-iamserviceaccount-**.

6.  To create RBAC permissions and a service account for the ALB Ingress Controller, run the following command:

```
$ curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
```
7. Open the rbac-role.yaml file in a text editor, and then make the following changes only to the ServiceAccount section:
```
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/name: alb-ingress-controller
  annotations:                                                                              # Add the annotations line
    eks.amazonaws.com/role-arn: arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/YOUR_IAM_ROLE_NAME    # Add the IAM role
  name: alb-ingress-controller
  namespace: kube-system
```
8. Save the rbac-role.yaml file, and then run the following command:

```
$ kubectl apply -f rbac-role.yaml
```

## Set up the ALB Ingress Controller

To run the ALB Ingress Controller as a Fargate pod, you must use **iam-for-pods**.

**Note**: For testing purposes only, add AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY as environment variables in the **alb-ingress-controller.yaml** file. It's a best practice to create [IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html).


1. To download the YAML file for the controller, run the following command:
```
$ curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/alb-ingress-controller.yaml
```

2. Open the **alb-ingress-controller.yaml** file in a text editor, and then make the following changes:

```
    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=your-cluster-name                           #<-- Add the cluster name
        - --aws-vpc-id=vpc-xxxxxxxxxxxxxxxxx                         #<-- Add the VPC ID 
        - --aws-region=eu-west-1                                     #<-- Add the region 
        image: docker.io/amazon/aws-alb-ingress-controller:v1.1.4    #<======= Please make sure the Image is 1.1.4 and above. 
        imagePullPolicy: IfNotPresent
```
3. To apply the **alb-ingress-controller.yaml** file, run the following command:

```
$ kubectl apply -f alb-ingress-controller.yaml
```

4. To check the status of the alb-ingress-controller deployment, run the following command:

```
$ kubectl rollout status deployment alb-ingress-controller -n kube-system
```

## Test the ALB Ingress Controller

You can create ALB Ingress resources and a Fargate profile to test the ALB Ingress Controller.

1. In the [Amazon EKS console](https://console.aws.amazon.com/eks/), create a [Fargate profile](https://docs.aws.amazon.com/eks/latest/userguide/fargate-profile.html) for the namespace 2048-game, or run the following command to create the profile using eksctl:

```
$ eksctl create fargateprofile --namespace 2048-game --cluster your-cluster-name
```

**Note**: If you create the profile with the Amazon EKS console, use the private subnets associated with your Amazon Virtual Private Cloud (Amazon VPC). Pods running on Fargate aren't assigned public IP addresses. Fargate profiles support only private subnets (with no direct route to an internet gateway).

2. To download the 2048-ingress file, run the following command:
```
$ curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-ingress.yaml
```

3. Open the 2048-ingress file in a text editor, and then make the following changes to the annotations:
```
 annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip                       # Add this annotation
    alb.ingress.kubernetes.io/security-groups: your-security-group  # Custom security group
```
**Note**: The ALB Ingress Controller works only in the IP mode on Amazon EKS for Fargate. For more information, see [Ingress annotations](https://kubernetes-sigs.github.io/aws-load-balancer-controller/guide/ingress/annotations/) on the AWS ALB Ingress Controller website.

4. To apply the files for the test deployment, run the following commands:

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-namespace.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-deployment.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-service.yaml
$ kubectl apply -f 2048-ingress.yaml
```
5. To see the 2048 page using the address that you receive, run the following command:

```
$ kubectl get ingress/2048-ingress -n 2048-game
```

The command output should have the load balancer's fully qualified domain name (FQDN) that you can access from a web browser.


## Troubleshoot the ALB Ingress Controller

If you have issues setting up the ALB Ingress Controller, run the following commands:
```
$ kubectl logs your-alb-ingress-controller -n kube-system
$ kubectl get endpoints -A
$ kubectl get ingress/2048-ingress -n 2048-game
```

The output from the logs command returns error messages (for example, with tags or subnets) that can help you troubleshoot [common errors](https://github.com/kubernetes-sigs/aws-alb-ingress-controller/issues). The get endpoints and get ingress commands can show you ingress resources that aren't deployed successfully.

If you run the YAML file for the ALB Ingress Controller without changing the image to 1.1.4, then you receive an error stating that the ALB Ingress Controller is unable to find the instance ID. To resolve this error, see [aws-alb-ingress-controller](https://github.com/kubernetes-sigs/aws-alb-ingress-controller/commit/4d1f94caa146a79f662ce66f7cabfdf0d355ac69#diff-0e2522e4ac7bdfb6c24a6093a6e09cea) on the Kubernetes GitHub website.

**Note**: The image 1.1.3v has the configurations to check for the network interface of your Amazon Elastic Compute Cloud (Amazon EC2) instance. To enable the ALB Ingress Controller to run on Fargate, the image code is changed to find the associated elastic network interfaces instead of the Amazon EC2 worker nodes.


## Reference Links

### AWS ALB
https://aws.amazon.com/premiumsupport/knowledge-center/eks-alb-ingress-controller-fargate/
https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/master/docs/guide/walkthrough/echoserver.md
https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/
https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html

### NGINXC
https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/


