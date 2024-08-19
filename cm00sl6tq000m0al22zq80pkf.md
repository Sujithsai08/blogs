---
title: "Deploying a 2048 Game Application on AWS EKS"
seoTitle: "AWS EKS"
datePublished: Mon Aug 19 2024 09:27:07 GMT+0000 (Coordinated Universal Time)
cuid: cm00sl6tq000m0al22zq80pkf
slug: deploying-a-2048-game-application-on-aws-eks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1724059473915/d17baa6c-b478-42d0-a207-8cca78e8dad0.png
tags: aws, kubernetes, devops, load-balancer, ingress, eks, aws-eks, aws-devops, 2048-game

---

### Introduction:

In this guide, we will deploy a 2048 game application on Amazon Elastic Kubernetes Service (EKS). We will use a Virtual Private Cloud (VPC) to manage network resources, an Ingress controller to handle external access, and AWS Fargate to manage and scale our worker nodes. This setup will ensure that the application is secure, scalable, and efficiently managed, leveraging EKS's container orchestration capabilities and Fargate's serverless compute environment. The tutorial will cover configuring the VPC, setting up the Ingress controller, and deploying the application to make it accessible to users.

### Traffic Flow:

* **Client Request** -&gt; **AWS ALB** -&gt; **Ingress Controller** -&gt; **Kubernetes Service** -&gt; **2048 Application Pods** running on **AWS Fargate** within **AWS EKS**.
    

### **What is EKS:**

EKS stands for Amazon Elastic Kubernetes Service. AWS EKS manages the Kubernetes control plane, which includes handling the infrastructure, scalability, and availability of the Kubernetes master nodes. This allows users to focus on deploying and managing their applications, while AWS handles the operational overhead of the Kubernetes control plane.

### Prerequisites:

Before getting started, we need to install some essential tools to interact with AWS EKS.

1. **kubectl**: Follow this [documentation](https://kubernetes.io/docs/tasks/tools/) to install the appropriate kubectl version based on your system specifications.
    
2. **eksctl**: eksctl is a command-line interface for interacting with EKS clusters. Follow this [installation guide](https://eksctl.io/installation/) to install and configure it.
    
3. **AWS CLI**: AWS CLI is a command-line interface for interacting with your AWS account. Follow this [guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) to install and configure it.
    

### Step-1:Creating an EKS Cluster:

Open your AWS console and search for the AWS EKS service.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724045819972/57694355-0113-4a74-afea-74b60b8a4dd1.png align="center")

Generally, it is preferable to use eksctl to create EKS clusters because, through the UI, we need to provide a lot of parameters. So, we will be using eksctl.

```bash
eksctl create cluster --name demo-2048 --region us-east-1 --fargate
```

The above command creates a cluster named demo-2048 in the us-east-1 region. It also sets up public and private subnets within the VPC, configures the cluster to use AWS Fargate for running Kubernetes pods, and creates the necessary IAM roles and policies for the EKS cluster and Fargate.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724047101078/d4150864-8bd5-4a26-aa79-1c630127a3da.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724047118006/11b34c6d-4f58-4295-aa38-cca6cde89d13.png align="center")

now verify on the aws console.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724047382803/1f427b60-ff5b-465b-8bc5-945f099c85e2.png align="center")

For kubectl to communicate with the Kubernetes cluster that we just created, we need the kubeconfig file for that.

```bash
aws eks update-kubeconfig --name demo-2048 --region us-east-1
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724047452943/264be9c5-dcc2-4e9d-93d6-bab53563101c.png align="center")

### Step-2: Deploying 2048 application

To deploy the application, we need a Fargate profile. Fargate is a serverless computing service that allows you to run containers without managing the underlying infrastructure, similar to AWS Lambda. However, while Lambda is typically used for quick, event-driven actions, Fargate is designed to run containers for longer-running processes.

Now, let's create a Fargate profile.

```bash
eksctl create fargateprofile \
    --cluster demo-2048 \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724047639040/ac7ed5e1-3649-4f47-ae97-5316d682ccc9.png align="center")

You can view the Fargate profile under the Compute section in the AWS EKS console.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724047756478/01ad5401-7a62-4197-a844-2b98fce8553b.png align="center")

Now let's deploy our 2048 application.

Run the command below:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

the above command creates a namespace,deployment, service, and ingress resources using a single YAML file.

the content of the above yaml file goes here:

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: game-2048
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: game-2048
  name: deployment-2048
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: app-2048
  replicas: 5
  template:
    metadata:
      labels:
        app.kubernetes.io/name: app-2048
    spec:
      containers:
      - image: public.ecr.aws/l6m2t8p7/docker-2048:latest
        imagePullPolicy: Always
        name: app-2048
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  namespace: game-2048
  name: service-2048
spec:
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  type: NodePort
  selector:
    app.kubernetes.io/name: app-2048
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: game-2048
  name: ingress-2048
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: service-2048
              port:
                number: 80
```

Details of the YAML file:

* It creates a namespace named `game-2048` in the Kubernetes cluster.
    
* It creates a deployment with 5 replicas using the Docker image [`public.ecr.aws/l6m2t8p7/docker-2048:latest`](http://public.ecr.aws/l6m2t8p7/docker-2048:latest).
    
* It creates a service named `service-2048` with NodePort type, mapping incoming traffic on port 80.
    
* It defines an Ingress resource named `ingress-2048` with annotations for the AWS ALB Ingress Controller, directing traffic to the `service-2048` service.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724048041294/13863a84-f8da-494f-a2b2-f92704195462.png align="center")

verify the pods are up and running by running below command

```bash
kubectl get pods -n game-2048
kubectl get svc -n game-2048
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724048242258/e7834b8c-58f1-4c88-91ee-bc4a8fad9b59.png align="center")

As you can see, there is no external IP address because we exposed our application using the `NodePort` type. To expose our application to the world, we need to use a load balancer in conjunction with an Ingress controller. The load balancer will route external traffic to the Ingress controller, which will then direct it to the appropriate service within the cluster.

### Step 3: Installing and Configuring Ingress Controller

Before installing and configuring the Ingress controller, we need to configure the OIDC connector. The reason for configuring the OIDC connector is that the ALB controller, which will be managing the Application Load Balancer, requires the necessary permissions to access and manage the load balancer.

```bash
eksctl utils associate-iam-oidc-provider --cluster demo-2048 --approve
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724048776968/17200b89-5f54-4e96-bf63-4777ee8e417c.png align="center")

#### Create IAM Policy

We will soon install the ALB controller, which is a pod that manages communication with the AWS Application Load Balancer service. To enable this communication, we must assign the necessary permissions to the service role. Begin by downloading the IAM policy JSON file to set up these permissions.

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

Create an IAM policy using the downloaded JSON file.

```bash
 aws iam create-policy \
 --policy-name AWSLoadBalancerControllerIAMPolicy \
 --policy-document file://iam_policy.json
```

**Create an IAM Service Role**

Create an IAM service role to provide the ALB controller pod with the necessary permissions to interact with the AWS Application Load Balancer. Attach the IAM policy to specify the exact actions the controller can perform.

```bash
eksctl create iamserviceaccount --cluster=demo-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --approve
```

**Install the ALB Ingress Controller**

Download and install Helm. Refer: [**https://helm.sh/docs/intro/install/**](https://helm.sh/docs/intro/install/)

after downloading helm, run the following commands to Add the EKS Helm repository and install the ALB Ingress Controller.

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=demo-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=<your-vpc-id>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724049624218/2b211b8a-bc39-436f-b0b1-7e0aff22fd1e.png align="center")

now lets verify the deployment

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl get pods -n kube-system
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724049712412/46369601-ba64-4be0-97d1-3fc60565bf11.png align="center")

### Step-4: Accessing the application

To access our application, we need its address. Run the following command, copy the address, and paste it into your browser.

```bash
kubectl get ingress -n game-2048
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724049778080/141d3cc4-b0f6-474d-89e5-cb02ed6287ae.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724050006765/2710754c-3c9c-4f1f-93fb-bdaf2dd73cac.png align="center")

We have successfully deployed our 2048 Game Application!

### Conclusion

In this project, we successfully deployed a 2048 game application on Amazon Elastic Kubernetes Service (EKS). By leveraging AWS Fargate for serverless compute, we efficiently managed and scaled the application's worker nodes. We configured a Virtual Private Cloud (VPC) and an Ingress controller to ensure secure and scalable access to the application. Through careful setup and management, we achieved a robust and accessible deployment, demonstrating the powerful capabilities of EKS in orchestrating containerized applications. This project highlights the benefits of using managed Kubernetes services and serverless compute to streamline application deployment and management.