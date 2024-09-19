---
title: "Elevate Your Go App with DevOps Practises: The Ultimate Guide to Streamlined Deployment"
datePublished: Thu Sep 19 2024 04:10:50 GMT+0000 (Coordinated Universal Time)
cuid: cm18rxuct001d09l70aqwedi8
slug: elevate-your-go-app-with-devops-practises-the-ultimate-guide-to-streamlined-deployment
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1726689274948/129ca72c-1ad1-42fc-a879-878155df2a20.png
tags: productivity, docker, aws, go, github, projects, community, kubernetes, git, devops, eks, github-actions-1, devops-articles, argocd, aws-devops

---

### Introduction:

In the rapidly evolving software development world, efficient application deployment and management are key to success. This project showcases the transformation of a Go application into a highly automated, scalable, and maintainable system using advanced DevOps practices. The goal is to optimize the development workflow and ensure smooth integration and delivery through state-of-the-art tools and techniques

### **Project Overview:**

1. **Application Analysis and Local Setup:**
    
    * Start by examining the Go application’s structure and running it locally. This step helps in understanding its dependencies and configurations, which are crucial for creating an effective Docker container.
        
2. **Containerization:**
    
    * Develop a streamlined Docker image using a multi-stage Dockerfile, which separates the build and runtime environments. This approach results in a lightweight and secure image.
        
3. **Kubernetes Deployment:**
    
    * Create Kubernetes manifests to manage the deployment, service, and ingress of the application. Deploy these manifests on an AWS EKS cluster to ensure scalability and high availability.
        
4. **Ingress Setup:**
    
    * Implement an Nginx ingress controller to handle external access
        
5. **Helm Chart Development:**
    
    * Build a Helm chart to facilitate configuration and deployment across various environments, such as development and production. This approach simplifies the management of environment-specific settings.
        
6. **Continuous Integration with GitHub Actions:**
    
    * Configure a GitHub Actions CI pipeline to automate the build, test, and deployment processes. This setup ensures that every code change is automatically tested and deployed, maintaining high code quality.
        
7. **Continuous Delivery with ArgoCD:**
    
    * Integrate ArgoCD for continuous delivery, providing a robust interface for managing and monitoring deployments. ArgoCD automatically deploys updates to the Kubernetes cluster when changes are detected.
        
    * ![Implementing DevOps Practices for Go Applications with Docker, Kubernetes, Helm, and ArgoCD](https://cdn.hashnode.com/res/hashnode/image/upload/v1722760077104/f6e965cc-ae38-4a56-8804-cf2840686d75.png?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp align="left")
        

### Running the application Locally

Before deploying the application with DevOps practices, it’s essential to verify that the Go application runs correctly on your local machine.

Clone the go lang application repository from GitHub by running the below command

```bash
git clone https://github.com/iam-veeramalla/go-web-app
cd go-web-app
```

Running the server:

To run the server, execute the following command:

```bash
go run main.go
```

You can access it by navigating to [`http://localhost:8080/courses`](http://localhost:8080/courses) in your web browser.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726663398659/0e1e6567-e52b-48eb-8b3e-67ad58e6d985.jpeg align="center")

Now lets apply all the devops practises on this go lang application.

### Step-1: Containerizing the application

lets write a multi stage docker file and build the docker image for this go lang application

```dockerfile
FROM golang:1.22.5 AS builder
WORKDIR /app
COPY go.mod .
RUN go mod download
COPY . .
RUN go build -o main .

FROM gcr.io/distroless/base


COPY --from=builder /app/main .
COPY --from=builder /app/static ./static

EXPOSE 8080
CMD ["./main"]
```

Lets break down the dockerfile:

* `FROM golang:1.22.5 AS builder`: This specifies the base image for the first stage of the build. It uses the `golang` image with version `1.22.5`
    
* `WORKDIR /app`: Sets the working directory inside the container to `/app`.
    
* `COPY go.mod .`: Copies the `go.mod` file from your local machine into the `/app` directory in the container.
    
* `RUN go mod download`: Downloads the Go module dependencies specified in `go.mod`.
    
* `COPY . .`: Copies the rest of the application code from your local machine into the `/app` directory in the container.
    
* `RUN go build -o main .`: Builds the Go application. The `-o main` flag specifies that the output binary should be named `main`.
    
* `FROM` [`gcr.io/distroless/base`](http://gcr.io/distroless/base): Specifies the base image for the second stage of the build. [`gcr.io/distroless/base`](http://gcr.io/distroless/base) is a minimal base image provided by Google that contains only the runtime libraries needed to run the application, but not the package manager or shell.
    
* `COPY --from=builder /app/main .`: Copies the `main` binary from the `builder` stage into the current directory (`.`) of this stage.
    
* `COPY --from=builder /app/static ./static`: Copies the `static` directory from the `builder` stage into the `./static` directory of this stage.
    
* `EXPOSE 8080`: Documents that the application will listen on port `8080`
    
* `CMD ["./main"]`: Sets the default command to run when a container is started from this image. In this case, it runs the `main` binary built earlier.
    

Now lets build and push it to the dockerhub

To build and push the docker image follow the below command

```dockerfile
docker build -t "your_dockerhub_username"/go-web-app:1
```

**Log In to Docker Hub**

Before pushing the image, you need to log in to Docker Hub.

```dockerfile
docker login
```

Push the docker image to the dockerhub

```dockerfile
docker push "your_dockerhub_username"/go-web-app:1
```

### Step-2: Kubernetes Manifests

**Create a deployment.yml file**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-web-app
  template:
    metadata:
      labels:
        app: go-web-app
    spec:
      containers:
      - name: go-web-app
        image: sujithsai/go-web-app:1
        ports:
        - containerPort: 8080
```

This deployment manifest defines a Kubernetes Deployment resource, which is responsible for managing a set of identical application pods.

**Create a service.yml file**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: go-web-app
  type: NodePort
```

his service manifest defines a Kubernetes Service resource, which serves several critical functions for managing and accessing deployed pods:

* **Load Balancing**: Distributes incoming traffic evenly across the pods that match the specified label (`app: go-web-app`), ensuring balanced load distribution.
    
* **Port Mapping**: Configures the service to expose an external port (`80`) and map it to the target port (`8080`) on the pods.
    
* **Service Type (NodePort)**: Allows external access to the service from outside the Kubernetes cluster via a specific port on each node
    

**Create a ingress.yml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-web-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: 
    http:
      paths: 
      - path: /
        pathType: Prefix
        backend:
          service:
            name: go-web-app
            port:
              number: 80
```

With Ingress, users create an Ingress resource, and cloud providers offer Ingress controllers. These controllers work with different load balancer solutions to provide advanced features like path-based routing, SSL termination, and better resource use.  
here in our case we are using nginx as loadbalancer.

### Step-3:Creating an EKS Cluster

Before getting started, we need to install some essential tools to interact with AWS EKS.

1. **kubectl**: Follow this [**documentation**](https://kubernetes.io/docs/tasks/tools/) to install the appropriate kubectl version based on your system specifications.
    
2. **eksctl**: eksctl is a command-line interface for interacting with EKS clusters. Follow this [**installation guide**](https://eksctl.io/installation/) to install and configure it.
    
3. **AWS CLI**: AWS CLI is a command-line interface for interacting with your AWS account. Follow this [**guide**](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) to install and configure it.
    

Now lets create an eks cluster, to create an eks cluster using aws cli follow the below command.

```yaml
eksctl create cluster --name go-web-app --region us-east-1
```

The created cluster is of EC2 type and not fargate, so EC2 instances will be created by EKS in the same region(`us-east-1`)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726666579410/4eba62d2-f4cd-4ba5-bdd1-d4677fbbc66f.jpeg align="center")

Next apply all the kubernetes manifests that we created by using the below command

```bash
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f ingress.yml
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726666727611/d120bccf-91c7-4768-bb72-b28d4f19a704.jpeg align="center")

you can verify using the below command

```bash
kubectl get all
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726666790746/da01c61f-c39d-4569-be23-bf9838a2bf0f.jpeg align="center")

### Step-4 Create an ingress controller

Since we are deploying on AWS EKS and need to access the application from outside, we need to create a LoadBalancer. We will be using an NGINX Ingress Controller for this purpose. You can use any other LoadBalancer or Ingress Controller of your choice if preferred.

now lets install the load balancer,run the below command..

```bash
kubectl apply -fhttps://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726667058918/344efb60-906b-4b64-923c-db0f635e6e40.jpeg align="center")

Now that we have created all the necessary Kubernetes manifests to deploy our application on the Kubernetes cluster and access it from the outside world, you can access the application by running the following command and copying the address:

```bash
kubectl get ingress
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726667155923/1000dd0e-9dae-4378-8165-dc51bf1f1648.jpeg align="center")

access the application using the above address/courses

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726667200441/ab6586f8-a86f-45e4-934b-40a6778d328c.jpeg align="center")

### Step-5 Helm chart creation:

The concept of Helm is designed to simplify the deployment of applications across different environments, such as development, production, and testing. Typically, you might create separate Kubernetes manifests for each environment, but this process is inefficient and time-consuming. To solve this problem, instead of writing individual Kubernetes manifests for each stage, we create a **Helm chart** and pass environment-specific values. This approach is **universal** and works for all stages, making the deployment process more streamlined and manageable.

create helm using the following command

```bash
helm create go-web-app-chart
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726683483071/10cadf08-82fb-43c3-9c24-94b719fcd3f0.jpeg align="center")

Now, copy all the Kubernetes manifest files to the templates folder in the go-web-app-chart. You can do this with the following command:

```bash
cd templates
cp ../../kubernetes/manifests/* .
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726683588494/37717d3d-f0e4-4e7a-8b73-c26c5ecca8c5.jpeg align="center")

Update values.yaml according to our manifests

```bash
# Default values for go-web-app-chart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: sujithsai/go-web-app
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
```

Now update the deployment.yml file as shown below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726683674478/8fcdee2f-9135-4870-8393-cffc85a63012.jpeg align="center")

The above change indicates that it refers to the `values.yaml` file in the `go-web-app-chart` folder and accesses the image using the “tag” field.

Now lets delete all the resources and re install using helm

```bash
kubectl delete deploy go-web-app
kubectl delete svc go-web-app
kubectl delete ing go-web-app
```

Verify

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726684027397/d5c41e75-83bf-4a56-a8e3-c87b1396e650.jpeg align="center")

Go back to the main helm directory and install it using below command

```bash
helm install go-web-app ./go-web-app-chart
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726684099963/affe6c74-b407-495a-8b8c-a6b2abde8769.jpeg align="center")

Verify the creation.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726684128777/cb8e843d-aa7e-4ed2-a617-7b99b92c7a1e.jpeg align="center")

All the resources are created as mentioned in the templates and helm is working, so uninstall all the resources again.

```bash
helm uninstall go-web-app-chart
```

### Step-6 Continuous Integration using GitHub Actions

Create a folder .github/workflows

Before creating the `ci.yml` file, you need to add the required credentials to the repository's secrets.

You will need:

* **Docker credentials** to push the built Docker image.
    
* **GitHub credentials** to update the Helm values.
    

To add these credentials, follow these steps:

1. Navigate to the repository's **Settings**.
    
2. Go to **Secrets and variables**.
    
3. Select **Actions secrets**.
    
4. Store all the required credentials here.Similarly add “DOCKERHUB\_TOKEN” and “TOKEN”(Github token”)
    

Now create a ci.yml file

```bash
# CICD using GitHub actions

name: CI/CD

# Exclude the workflow to run on changes to the helm chart
on:
  push:
    branches:
      - main
    paths-ignore:
      - 'go-web-app-chart/**'
      - 'kubernetes/**'
      - 'README.md'

jobs:

  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Go 1.22
      uses: actions/setup-go@v2
      with:
        go-version: 1.22

    - name: Build
      run: go build -o go-web-app

    - name: Test
      run: go test ./...
  
  code-quality:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Run golangci-lint
      uses: golangci/golangci-lint-action@v6
      with:
        version: v1.56.2
  
  push:
    runs-on: ubuntu-latest

    needs: build

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Login to DockerHub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and Push action
      uses: docker/build-push-action@v6
      with:
        context: .
        file: ./Dockerfile
        push: true
        tags: ${{ secrets.DOCKERHUB_USERNAME }}/go-web-app:${{github.run_id}}

  update-newtag-in-helm-chart:
    runs-on: ubuntu-latest

    needs: push

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
      with:
        token: ${{ secrets.TOKEN }}

    - name: Update tag in Helm chart
      run: |
        sed -i 's/tag: .*/tag: "${{github.run_id}}"/' go-web-app-chart/values.yaml

    - name: Commit and push changes
      run: |
        git config --global user.email "sujithsai.sirimalla33@gmail.com"
        git config --global user.name "Sujithsai08"
        git add go-web-app-chart/values.yaml
        git commit -m "Update tag in Helm chart"
        git push
```

Breakdown of the above ci.yml file

* The CI workflow is triggered when somebody pushes to repository however it ignores any pushes made to “Readme”,”kubernetes”,”go-web-app-chart” folders
    
* #### Stage 1 - Build and Unit Test:
    
    In this job, the application code is checked out, the Go environment is set up, the application is built, and tests are run.
    
* #### Stage 2 - Static Code Analysis:
    
    In this job, `golangci-lint` is used to ensure code quality.
    
* #### Stage 3 - Building Docker Image and Pushing to Docker Hub:
    
    In this job, a Docker image is built from the repository and pushed to Docker Hub using the credentials stored as secrets in the repository.
    
* #### Stage 4 - Update Tag in Helm Chart:
    
    In this job, the "tag" in the Helm chart values is updated with the GitHub unique runner ID and the changes are pushed to the github repo with the given credentials
    

Now, push the changes to your repository to run the `ci.yml`.

Once the push is done, GitHub will automatically trigger the workflow.

Verify the continuous integration using GitHub Actions.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726686674156/b378d0d6-b854-4fc3-a2a6-f50031677ece.jpeg align="center")

next verify the docker image that is pushed into dockerhub by ci.yml

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726686656733/02bdfc87-9f51-4f7b-940f-fc0ec9da6763.jpeg align="center")

next verify the image tag name in the values.yaml file

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726686705675/20bc002b-4bf3-4d7f-9a53-96c781613f09.jpeg align="center")

As you can see, the new Docker image tag that was pushed to Docker Hub has also been updated in the `values.yaml` file.

### Step-7 CD using **ArgoCD**

install argocd by using below command:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Accessing the argocd ui(as a loadbalancer)**

apply the below command

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726687116207/308a95f5-bd1f-4055-9a66-304d00034a87.jpeg align="center")

Access the ArgoCD UI using the external IP address. You can find it by using the command below:

```bash
kubectl get svc -n argocd
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726687194201/5bb12885-f727-4b96-ae5e-b1333feabab7.jpeg align="center")

Copy the external IP address and paste it into your browser to access the ArgoCD UI.

Now Get the password using

```bash
kubectl edit argocd-intial-admin-secret -n argocd
```

The password you obtained is encoded with base64, so we need to decode it in order to log in to the ArgoCD UI.

To decode the password, use the following command:

```bash
echo "password" | base64 --decode
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726687519645/ae00f1a3-9c9c-4b67-acf8-963eb0731d31.jpeg align="center")

Copy the decoded password and paste it into the password field in the ArgoCD UI. The username is `admin`, and the password is the one you just decoded.

Now click on new app and pass the fields as shown below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726687631234/d29b9214-45ac-468f-b825-ab922eaccfce.jpeg align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726687644932/5ad518a8-3834-4c8a-9df7-44b3876d6061.jpeg align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726687649876/7d5bcfb9-9e44-49f3-ba11-a60f71e0e48b.jpeg align="center")

click on create.

Now verify whether argocd deployed our application on eks

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726687721228/60b45c9a-6685-4110-b098-c8d5e0ed83e0.jpeg align="center")

access the application from the above address

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726687742860/65165dd5-e978-477f-a346-29be2c23faa7.jpeg align="center")

Now, let's verify that all the components are working correctly.

To do this, make a small change in the `home.html` file. Replace “Learn DevOps from basics” with “Go-web-app project by Sujithsai.”

The expected outcome is that when any changes are made to the files in the GitHub repository, the CI pipeline should be triggered. It should then execute all the stages automatically, including:

1. **Checkout** the code.
    
2. **Build** the application.
    
3. **Push** the Docker image to Docker Hub.
    
4. **Update** the “tag” in the `values.yaml` file.
    

All these steps should happen automatically as part of the CI process.

Lets edit home.html file as shown below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726688002654/f49c5740-789f-43df-9aa3-868868ca9d50.jpeg align="center")

Verify continuous integration using GitHub Actions.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726688029713/0afab100-91d3-4469-9d29-e49b134bfab2.jpeg align="center")

Verify the Docker image tag in Docker Hub.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726688050263/1e007c9d-bfa6-4153-bbde-9a8cca44f2ba.jpeg align="center")

Verify the “tag” in the values.yaml file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726688067776/3d9cb5f0-1b8e-4eb5-8b39-2696b42bda48.jpeg align="center")

After the completion of the CI process, ArgoCD will monitor the repository for any changes. If changes are detected, it will automatically deploy the updated application to the desired Kubernetes cluster.

Let's go ahead and verify this behavior.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726688132666/38633d69-39c8-4c3f-bea1-5681bb270f88.jpeg align="center")

As you can see, the change has been automatically detected, updated, and deployed by ArgoCD.

### Conclusion:

This project showcases the process of transforming a Go application into an automated, scalable deployment using modern DevOps practices. We containerized the app with a multi-stage Dockerfile, created Kubernetes manifests, and deployed it on an AWS EKS cluster. An Nginx ingress controller was set up for external access. Using a Helm chart, we enabled easy configuration and deployment across environments. GitHub Actions handled continuous integration, automating build, test, and deployment. Finally, we integrated ArgoCD for continuous delivery, streamlining the entire workflow.