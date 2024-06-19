---
title: "Installing and Configuring ArgoCD : A Step-by-Step Guide"
datePublished: Wed Jun 12 2024 13:03:39 GMT+0000 (Coordinated Universal Time)
cuid: clxbudpfz000408k140567qro
slug: installing-and-configuring-argocd-a-step-by-step-guide
tags: docker, aws, kubernetes, devops, devsecops, argocd, kunalkushwaha

---

## Introduction

ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. It allows you to manage your Kubernetes resources using Git repositories as the source of truth.

## Prerequisites

Before you begin, ensure you have the following:

* A Kubernetes cluster
    
* `kubectl` command-line tool installed
    

## Installation

Before installing ArgoCD, make sure that your Kubernetes cluster is up and running! You can verify using the below command:

```sh
minikube status
```

You will get an output like this:

![Minikube Status](https://cdn.hashnode.com/res/hashnode/image/upload/v1718193597358/b00f3bca-0c3c-4cf3-8daf-5604970726bc.png align="left")

We can install ArgoCD in three ways:

1. Plain Manifests
    
2. Helm Charts
    
3. Operators
    

In this blog, I will show you how to install using Plain Manifests.

### Step 1:

Open your command line interface and run the following command:

```sh
kubectl create namespace argocd
```

![Create Namespace](https://cdn.hashnode.com/res/hashnode/image/upload/v1718193951630/da82e225-9fc6-43ce-b148-9ef262c83194.png align="left")

This command creates a namespace called "argocd".

Next, run the below command:

```sh
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

![Apply Manifests](https://cdn.hashnode.com/res/hashnode/image/upload/v1718194003045/2a57835f-b036-45ed-994b-de1cc7659aea.png align="left")

### Step 2:

Once ArgoCD is installed, wait for the pods to come up. You can view the pods by running the following command:

```sh
kubectl get pods -n argocd -w
```

![Get Pods](https://cdn.hashnode.com/res/hashnode/image/upload/v1718194309689/1386001b-bfd2-4139-80c5-690f0aff7af2.png align="left")

### Step 3:

Once the pods are up and running, run the following command in your command line interface:

```sh
kubectl get svc -n argocd
```

This command lists the services available in the "argocd" namespace. The ArgoCD server is responsible for interacting with the UI using the command line interface.

![Get Services](https://cdn.hashnode.com/res/hashnode/image/upload/v1718194418316/f1965983-d9c9-4cd1-8f66-51a417117040.png align="left")

### Step 4:

Next, run the following command:

```sh
kubectl edit svc argocd-server -n argocd
```

The reason for editing the "argocd-server" file is that, by default, the ArgoCD server type is set to ClusterIP. We need to change the type of "argocd-server" from ClusterIP to NodePort to access the service from the browser.

![Edit Service](https://cdn.hashnode.com/res/hashnode/image/upload/v1718194743756/c90e4bdb-e33e-4976-9c95-2bc13d45d329.png align="left")

![Edit Service](https://cdn.hashnode.com/res/hashnode/image/upload/v1718194770672/6e87ada9-9db0-4111-a95a-89ea1a1b30a2.png align="left")

### Step 5:

Since we have changed the type from ClusterIP to NodePort, we can now access the service from the terminal. However, if we want to access it using a browser, we need to do port forwarding or tunneling.

To do that, run the following command to list the services in the "argocd" namespace:

```sh
minikube service list -n argocd
```

![Service List](https://cdn.hashnode.com/res/hashnode/image/upload/v1718195095976/c09bfa1a-9c85-4883-ab20-d5142a2c5161.png align="left")

Now, run the command below:

```sh
minikube service argocd-server -n argocd
```

This command creates a tunnel and provides you an IP address to access it.

![Service Tunnel](https://cdn.hashnode.com/res/hashnode/image/upload/v1718195243382/1336d3a5-e549-460b-aa15-597d35a8a129.png align="left")

The first URL is for HTTP access, and the second URL is for HTTPS access. Paste either URL into your browser.

After pasting the URL, you will see a caution message like this:

![Security Warning](https://cdn.hashnode.com/res/hashnode/image/upload/v1718195333079/883f32e5-b091-4e30-8d7c-db109e909afe.png align="left")

Click on "Proceed". After clicking on "Proceed", you will be on the ArgoCD login page.

![Login Page](https://cdn.hashnode.com/res/hashnode/image/upload/v1718195382698/69f03d47-d490-4226-8634-26d5bb58e912.png align="left")

### Step 6:

To log in to ArgoCD, we need a username and password.

The username is "admin."

To get the password, open a new tab in the command line interface and run the following command:

```sh
kubectl get secret -n argocd
```

The above command lists the secrets in the "argocd" namespace.

![Get Secret](https://cdn.hashnode.com/res/hashnode/image/upload/v1718195652239/6282ba39-cb60-452a-a879-e16e138499ce.png align="left")

Now go to the "argocd-initial-admin-secret" by running the following command:

```sh
kubectl edit secret argocd-initial-admin-secret -n argocd
```

The above command opens the "argocd-initial-admin-secret" file where the password is stored. Now, copy the password as shown below.

![Copy Password](https://cdn.hashnode.com/res/hashnode/image/upload/v1718195811838/2c97cb68-103d-418e-b8a8-827562255719.png align="left")

Since the password is encrypted in base64 format, we need to decrypt it.

To decrypt it, run the following command:

```sh
echo "your_password" | base64 --decode
```

Replace "your\_password" with the original password. Now, paste the decrypted password in ArgoCD.

And that's it! You are now successfully logged into ArgoCD.

![ArgoCD Dashboard](https://cdn.hashnode.com/res/hashnode/image/upload/v1718196102076/3f323753-e5ce-49c4-a9b4-9309f0dd7e34.png align="left")

Thanks for reading my blog!