---
title: "Building a Kubernetes Application: From Pods to Ingress"
datePublished: Mon Jul 15 2024 14:31:28 GMT+0000 (Coordinated Universal Time)
cuid: clyn31rf8000008mh1sha1i9a
slug: building-a-kubernetes-application-from-pods-to-ingress
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721053678471/d7dba61f-65cd-46f6-a781-ad972780d72c.png
tags: docker, aws, kubernetes, devops, containers, ingress, argocd

---

### Introduction:

In the world of container orchestration, Kubernetes has become the de facto standard for deploying and managing containerized applications. This blog post will take you on a journey from understanding the foundational components of Kubernetes—such as Pods, Deployments, Services, and Ingress—to deploying a real-world application. We will explore each of these core concepts in detail and demonstrate how to use them to deploy the i-notes application. By the end of this post, you will not only have a deeper understanding of Kubernetes but also the practical knowledge to deploy and manage your own applications using these powerful tools. We'll conclude by showing how to use Ingress for path-based routing, ensuring efficient and accessible access to your application. Let's dive in!

### Overview:

The I-Notes application is a note-taking application consisting of a front-end (React) and a back-end (Node.js/Express) with a database (MongoDB). This guide will walk you through deploying the I-Notes application on a Kubernetes cluster using Pods, Deployments, Services, and Ingress.

GitHub link for the I-Notes application: [https://github.com/VishalRayabarapu-01/I-NotesApplication.git](https://github.com/VishalRayabarapu-01/I-NotesApplication.git)

### Prerequisites

Before diving into this blog on building a Kubernetes application from Pods to Ingress, you should have a basic understanding of the following:

1. **Docker**: Familiarity with containerization concepts and Docker commands, as Kubernetes builds on these container technologies.
    
2. **Command Line Interface (CLI)**: Basic skills in using the terminal or command prompt, as we will use kubectl commands to interact with the Kubernetes cluster.
    
3. **YAML**: Understanding of YAML syntax, as Kubernetes configuration files are written in YAML.
    
4. **Kubernetes Basics**: A rudimentary knowledge of Kubernetes concepts, such as clusters, nodes, and namespaces, will be helpful but not mandatory.
    

Before we begin, follow these steps to create an EC2 instance and install Docker and Kubernetes.

### **Create an ec2 instance:**

To get started, we need to create an EC2 instance with ubuntu as an operating system. We will be using a `t2.medium` instance type.

1. Login to your aws console --&gt; Select EC2 --&gt; Click on launch a new instance
    

**Configuring EC2 Instance:**

1. Choose Ubuntu as the operating system (OS).
    
2. Select t2.large as the instance type.
    
3. Set up a key pair for SSH access to the EC2 instance. If you don't have a key pair, click "Create new key pair".
    

Click on the "Launch instance" button to proceed.

**Connect to the instance:**

once the ec2 instance up and running connect to the instance using ssh

```bash
  ssh -i path/to/your_key_pair.pem ubuntu@your_ec2_instance_ip
```

Let's install Docker and Kubernetes by following the commands below:

```bash
# Steps:-

# For Docker Installation
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker

# For Minikube & Kubectl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube 

sudo snap install kubectl --classic
minikube start --driver=docker
```

### Drawbacks of Containers:

Before diving into Kubernetes, let's look at the drawbacks of containers.  
Here are the four major drawbacks:

1. **Ephemeral Nature**:
    
    Containers are designed to be ephemeral, meaning they can start, stop, and restart at any time. This transience can lead to issues if not managed properly, as containers can disappear unexpectedly, leading to application downtime.
    
2. **Single Host Limitation**:  
    Containers running on a single host share the host's resources. If one container (e.g., container 1) consumes a significant amount of memory or CPU, it can adversely affect other containers (e.g., container 50) due to resource contention. This problem is known as the single host issue, where resource isolation is not as strong as needed
    
3. **Lack of Autohealing**:  
    If a container fails or someone manually kills it, the application inside the container becomes inaccessible. Docker, by default, does not provide a built-in mechanism for autohealing, meaning you must manually restart the container or use additional tools to handle this automatically.
    
4. **Lack of Autoscaling**:  
    During periods of high load, such as festive days or special events, applications may need to handle a surge in requests. Docker does not natively support autoscaling, so scaling applications to meet demand must be done manually, which can be inefficient and slow.
    
5. **Limited Enterprise-Level Features**:
    
    * Docker, by itself, does not provide out-of-the-box support for several enterprise-level features required for robust application deployment and management. These features include:
        
        * **Load Balancers**: To distribute traffic efficiently across multiple containers or instances.
            
        * **Firewalls**: For enhanced security and controlled access to the containers.
            
        * **Autoscaling**: To automatically scale the number of running containers based on the load.
            
        * **Autohealing**: To automatically detect and recover from container failures.
            

### What is Kubernetes ??

Kubernetes, or K8s for short, is an open-source platform that helps manage containerized applications. It takes care of deploying, scaling, and managing your containers automatically. This means it solves problems like running everything on a single host, not having autohealing, and not having autoscaling.

* **Cluster Architecture**:
    
    * **Single Host Problem**: Kubernetes runs on a cluster setup, which is a bunch of nodes (machines) working together. It uses a master-node setup, where the master node manages everything. If one container (pod) messes up another, Kubernetes can move containers around to different nodes, balancing things out and making sure resources are available.
        
* **Autoscaling**:
    
    * **Replica Sets**: Kubernetes uses Replica Sets to keep a certain number of pod copies running all the time. This makes it super easy to scale apps by just changing the number of replicas.
        
    * **Horizontal Pod Autoscaler (HPA)**: HPA automatically changes the number of pods in a deployment based on CPU usage or other metrics. This helps apps handle different loads smoothly.
        
* **Autohealing**:
    
    * **Kubernetes Controllers**: Kubernetes controllers, like the Deployment controller, keep an eye on the pods. If a pod fails or gets killed, the controller starts a new one to replace it, making sure everything stays up and running.
        
* **Enterprise-Level Features**:
    
    * Kubernetes is designed for enterprise use and includes built-in features for robust application deployment and management:
        
        * **Load Balancing**: Distributes network traffic across multiple pods to ensure no single pod becomes a bottleneck.
            
        * **Security and Networking**: Supports various security features, including network policies and firewalls.
            
        * **Autoscaling**: Automatically scales applications based on demand.
            
        * **Autohealing**: Monitors and replaces failing pods to maintain application health.
            
    * **Kubernetes Architecture:**
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721050642258/0ebc0c65-315b-4945-9fa0-4e5d8a671ea9.jpeg align="center")
        
        **Control Plane:**
        
        1. **API Server:**
            
            * The API server is the heart of Kubernetes, handling all the requests and decisions.
                
            * It’s the part that exposes the Kubernetes API, letting users and components interact with the cluster.
                
        2. **Scheduler:**
            
            * The scheduler's job is to assign pods and resources to nodes based on what they need and any constraints.
                
            * It makes sure workloads land on nodes with enough resources and the best conditions.
                
        3. **etcd:**
            
            * etcd is a distributed key-value store that keeps the whole cluster's state.
                
            * It stores info as objects or key-value pairs, including configuration data, metadata, and the current state of the cluster.
                
        4. **Controller Manager:**
            
            * The controller manager runs a bunch of controllers that make sure the cluster stays in the desired state.
                
            * These controllers handle tasks like node lifecycle, replication, and endpoints.
                
        5. **Cloud Controller Manager:**
            
            * The cloud controller manager is an open-source tool that includes cloud-specific control logic.
                
            * It lets Kubernetes interact with cloud provider APIs, managing resources like load balancers, storage, and networking in a cloud environment.
                
        
        **Data Plane:**
        
        1. **kubelet:**
            
            * The kubelet is like an agent that runs on each node in the cluster.
                
            * It makes sure that containers (pods) are running smoothly on each node.
                
            * It talks to the container runtime to start, stop, and manage containers.
                
        2. * **Container Runtime:**
                
                * The container runtime is the software that runs containers.
                    
                * Kubernetes works with several container runtimes, like Docker, containerd, and CRI-O.
                    
                * The kubelet uses the container runtime to manage containers' lifecycle, including pulling images, creating containers, and running them.
                    
        3. **kube-proxy:**
            
            * kube-proxy keeps network rules on nodes, letting network traffic flow to and from pods.
                
            * It manages network routing and traffic, giving services unique IP addresses and enabling communication within the cluster.
                

### What is Pod??

A Pod is the lowest level of deployment in Kubernetes. Previously, in Docker, we used to run a container using the `docker run` command (e.g., `docker run -p 3000:3000 app:1`). In Kubernetes, instead of passing the necessary fields like port number and image name through a command, we create a `pod.yaml` file which includes all the necessary fields to run a Pod.

Now lets demostrate how to create a pod.

clone this repository using

```bash
git clone https://github.com/VishalRayabarapu-01/I-NotesApplication.git
cd I-NotesApplication
cd front-end
```

Create a pod.yml file using the command below:

```bash
vim pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: i-notes
spec:
  containers:
  - name: i-notes
    image: sujithsai/i-notes:1
    ports:
    - containerPort: 80
```

Apply the pod using below command

```yaml
kubectl apply -f pod.yml
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721021204395/7ea53914-fee4-4867-92e3-f60795a1b43a.png align="center")

Lets check the pod is running or not

```yaml
kubectl get pods -o wide
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721021257587/c279919b-8e51-473b-84f7-ff61a90df97c.png align="center")

Just implementing a Pod creates a single container instance. This approach doesn't provide the advantages of autoscaling and autohealing. To achieve these benefits, we need to create a `deployment.yml` file in Kubernetes.

### What is a Deployment??

The main reason we use Kubernetes is for its autoscaling and autohealing features. To leverage these capabilities, we create a `deployment.yml` file. While similar to a `pod.yml` file, a `deployment.yml` file includes a `replicas` field, which defines the desired number of identical pods to run. Kubernetes uses a ReplicaSet, a type of controller, to manage these pods, ensuring they are scaled up or down based on the specified number of replicas.

Now,lets create a deployment.yml file

```yaml
vim deployment.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: i-notes
  labels:
    app: i-notes
spec:
  replicas: 2
  selector:
    matchLabels:
      app: i-notes
  template:
    metadata:
      labels:
        app: i-notes
    spec:
      containers:
      - name: i-notes
        image: sujithsai/i-notes:1 
        ports:
        - containerPort: 3001
```

now lets apply by below command

```bash
kubectl apply -f deployment.yml
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721021790812/181c57f5-9bb1-4b9f-b355-9986260c0ade.png align="center")

one of the major advantages of using deployment.yml file is that it autoscales based upon the replica sets and even if we delete a pod it comes up with another pod as shown below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721021956960/4be4057a-432d-4cee-81a7-779609bcb81b.png align="center")

When using Deployments in Kubernetes, if a Pod is deleted or replaced, the new Pod often gets a different IP address. Managing these dynamic IP addresses can be challenging. To address this issue, Kubernetes introduces the concept of a Service. A Service provides a stable endpoint, typically an IP address and DNS name, for accessing a group of Pods. This abstraction simplifies network management by allowing other parts of your application or external services to reliably access your application through a consistent endpoint, regardless of Pod changes.

### What is a Service??

A Kubernetes **Service** provides a stable network endpoint (IP address and DNS name) to access a group of Pods that match a label selector, enabling load balancing and reliable communication within the cluster.

To solve the problem of managing dynamic IP addresses in Kubernetes, Services are used instead of directly accessing `deployment.yml` files. Services provide several benefits including load balancing, service discovery, and exposing applications to the outside world. Kubernetes offers three main types of Services:

* **ClusterIP:** Provides internal cluster-only access, not accessible from outside the Kubernetes cluster.
    
* **NodePort:** Exposes the Service on a static port on each node's IP, allowing access from outside the cluster.
    
* **LoadBalancer:** Automatically provisions an external load balancer and assigns a fixed, external IP address to expose the Service to the internet, allowing global access to the application."
    
* a service.yml file generally captures our application based on the selectors and labels
    

lets create a service.yml file

```bash
vim service.yml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: i-notes
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 3001
    protocol: TCP
  selector:
    app: i-notes
```

lets apply it!

```bash
kubectl apply -f service.yml
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721022061851/bae742f5-e863-4909-963d-290b15834509.png align="center")

lets cross check that our service is created

```bash
kubectl get svc
```

Notice the type of our application; it says NodePort since we specified it in the service.yml file. This means we can access the application from our browser.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721022079158/bd8a6d00-853a-4e02-888f-82ac468a60d2.png align="center")

Let's access our application. To access the application, we need to port-forward our service. Follow the commands below:

```bash
kubectl port-forward svc/i-notes 3001:80 --address 0.0.0.0
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721045386321/cdd1670c-6a49-43e0-a9c1-6dc09a2c97d9.png align="center")

access the application from your browser by navigating to [`http://<your_ec2_instance_ip_address:3001`](http://@your_ec2instance_ip_address:3001)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721045406465/80edea5a-c67a-4f6b-87f1-031824922848.png align="center")

Now let's use Ingress.

### What is Ingress??

Before Kubernetes version 1.0 in 2015, people used Kubernetes without built-in support for cool load balancing features like sticky sessions, path-based routing, and weighted load balancing. These features were common in enterprise load balancers like F5 and Nginx used with virtual machines. Back then, Kubernetes only had a basic LoadBalancer service that used round-robin scheduling.

Also, each Kubernetes Service usually needed a static IP address, which was pretty costly when managing multiple services.

To fix these issues, Kubernetes introduced Ingress. With Ingress, users create an Ingress resource, and cloud providers offer Ingress controllers. These controllers work with different load balancer solutions to provide advanced features like path-based routing, SSL termination, and better resource use.

Now lets do path based routing for our application using ingress

first install ingress on kubernetes by below command

```bash
minikube addons enable ingress
```

after installing ingress verify the ingress installation using

```bash
kubectl get ingress
```

Now let's create an `ingress.yml` file.

```bash
vim ingress.yml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: i-notes
            port:
              number: 3001
```

In this `ingress.yml` file:

* The `host` field specifies that requests coming to [`foo.bar.com`](http://foo.bar.com) will be handled by this Ingress.
    
* The `http` section defines HTTP-specific routing rules.
    
* The `paths` array specifies that requests to the path `/bar` (using `pathType: Prefix`) will be forwarded to the `i-notes` Service on port 3001.
    

To ensure requests to [`foo.bar.com/bar`](http://foo.bar.com/bar) are routed to the `i-notes` service using the `ingress.yml` file, you must first add [`foo.bar.com`](http://foo.bar.com) to the Ingress controller configuration. To determine the IP address assigned to the Ingress, use the command:

```bash
kubectl get ingress
```

To link the IP address of the Ingress controller with the domain [`foo.bar.com`](http://foo.bar.com) in the `/etc/hosts` section of the Ingress controller, use the following command:

```bash
sudo vim /etc/hosts
```

Add the IP address of the Ingress controller and the domain name to the file as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721046064474/5f536b88-274a-4809-96e6-e3f101859606.png align="center")

Now, let's use `curl` [`foo.bar.com/bar`](http://foo.bar.com/bar) to verify that it redirects to our application.

```bash
curl -l foo.bar.com/bar
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721046119772/5412dd2d-1cc3-4f0a-ac48-30b760869aac.png align="center")

And yes, we have successfully accessed the HTML page of our application. This confirms that we have implemented path-based routing with Ingress correctly. Whenever we curl [`foo.bar.com/bar`](http://foo.bar.com/bar), we are redirected to our `i-notes` application.

### Conclusion:

Through these steps, we've effectively deployed the i-notes application using Kubernetes features like Deployment for managing application instances, Service for internal communication and load balancing, and Ingress for routing external requests to the application. This setup ensures efficient deployment, scalability, and accessible routing within the Kubernetes cluster.