# This is a EKS course from beginner to advanced.

# Amazon EKS on AWS — Beginner to Advanced

Hands-on guides to create an Amazon EKS cluster, provision managed node groups, and expose workloads using AWS Load Balancer Controller with NLB and ALB.

## Repository structure

- Create cluster (Console + CloudFormation)
  - [00-create-cluster/00-cluster-using-aws-console.md](00-create-cluster/00-cluster-using-aws-console.md)
  - [00-create-cluster/01-managed-node-groups-aws-console.md](00-create-cluster/01-managed-node-groups-aws-console.md)
  - [00-create-cluster/02-create-cluster-cloudformation.md](00-create-cluster/02-create-cluster-cloudformation.md)
  - CloudFormation template: [00-create-cluster/EKS-cloudformation.yaml](00-create-cluster/EKS-cloudformation.yaml)
- AWS Load Balancer Controller
  - Install: [01-loadbalancer-controller/1-install-controller.md](01-loadbalancer-controller/1-install-controller.md)
  - NLB Service: [01-loadbalancer-controller/2-create-service-loadbalancer.md](01-loadbalancer-controller/2-create-service-loadbalancer.md)
  - ALB Ingress: [01-loadbalancer-controller/3-create-ingress.md](01-loadbalancer-controller/3-create-ingress.md)

## Prerequisites

- AWS account with admin or equivalent permissions
- AWS CLI v2 configured
- kubectl, eksctl, and Helm 3
- VPC with 2 public and 2 private subnets tagged for ELB/ALB (see [VPC setup](00-create-cluster/00-cluster-using-aws-console.md))

## Getting started

Choose one of the two paths below.

### A) Create the cluster via CloudFormation

1. Follow: [Create via CloudFormation](00-create-cluster/02-create-cluster-cloudformation.md)
2. Template: [EKS-cloudformation.yaml](00-create-cluster/EKS-cloudformation.yaml)
3. After stack completion, update kubeconfig and test:
   ```
   sh
   aws eks update-kubeconfig --region ap-south-1 --name Prod
   kubectl get nodes -o wide
   kubectl get ns
   ```

### B) Create the cluster via AWS Console

1. Provision VPC, subnets, and routing: [VPC steps](00-create-cluster/00-cluster-using-aws-console.md)
2. Create managed node group role and node groups: [Managed node groups](00-create-cluster/01-managed-node-groups-aws-console.md)
3. Configure your workstation or a bastion EC2 with kubectl/eksctl/helm (script included in the guide), then:
   ```
   sh
   aws eks update-kubeconfig --region ap-south-1 --name <your_cluster>
   kubectl get nodes
   ```

## Expose workloads

### Install AWS Load Balancer Controller

- Follow: [Install controller (IRSA + Helm)](01-loadbalancer-controller/1-install-controller.md)
- Verify controller is running:
  ```
  sh
  kubectl -n kube-system logs -f --selector app.kubernetes.io/instance=aws-load-balancer-controller
  ```

### Option 1: Network Load Balancer (Service type LoadBalancer)

- Deploy sample app and NLB Service: [NLB service steps](01-loadbalancer-controller/2-create-service-loadbalancer.md)
- After service is created, get the NLB DNS:
  ```
  sh
  kubectl get svc nginx-service
  ```

### Option 2: Application Load Balancer (Ingress)

- Create ClusterIP Service and Ingress: [ALB ingress steps](01-loadbalancer-controller/3-create-ingress.md)
- Get the ALB address:
  ```
  sh
  kubectl get ingress nginx-ingress
  ```

## Cleanup

- If created via CloudFormation: delete the stack from the console.
- If created manually: delete Kubernetes resources, node groups, the EKS cluster, and supporting infrastructure (ALBs/NLBs, target groups, security groups, and EIPs).

## Notes

- Default region used in examples: ap-south-1
- Default cluster name in the template: Prod
- Ensure subnets are correctly tagged for ELB/ALB and that the controller has IRSA with the required IAM policy.

