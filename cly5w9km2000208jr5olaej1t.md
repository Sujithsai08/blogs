---
title: "OPTIMIZE YOUR DOCKER FILE: UPTO 100% Image Size Reduction Using Multistage and Distroless Images"
datePublished: Wed Jul 03 2024 13:49:30 GMT+0000 (Coordinated Universal Time)
cuid: cly5w9km2000208jr5olaej1t
slug: optimize-your-docker-file-upto-100-image-size-reduction-using-multistage-and-distroless-images
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1720014400129/c63276a7-6791-4c2d-ac72-7ad5b768ba94.png
tags: docker, aws, devops, containers, docker-images, distroless, multistage-docker-build

---

### **Introduction:**

In the fast-paced world of containerized applications, optimizing Docker images is crucial for efficient deployment and resource utilization. One of the most effective strategies is leveraging multi-stage builds with Distroless images to significantly reduce image size. By eliminating unnecessary dependencies and focusing solely on runtime essentials, developers can trim down Docker images by up to 100%, enhancing both performance and security.

### **Step by Step guide**

### **Create an ec2 instance:**

To get started, we need to create an EC2 instance with ubuntu as an operating system. We will be using a `t2.micro` instance type.

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

* After connecting to your instance switch to root user by the following command
    

```bash
sudo su -
```

* Install all the necessary dependencies required to run our application using a Dockerfile.
    
* ```bash
      sudo apt update
      sudo apt install docker.io
      sudo apt install npm
    ```
    
* lets verify the installation of docker
    
    ```bash
    docker run hello-world
    ```
    
    you will get an output like this
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719927099340/64e4861c-132b-4565-a742-bffbb605ceb2.png align="center")
    

**Cloning the repository:**

Clone the github repository that contains a nodejs application

link : [https://github.com/VishalRayabarapu-01/I-NotesApplication.git](https://github.com/VishalRayabarapu-01/I-NotesApplication.git)

```bash
git clone https://github.com/VishalRayabarapu-01/I-NotesApplication.git
```

Navigate to the cloned application and create a docker file using the following command

```bash
cd I-NotesApplication
cd front-end
vim Dockerfile
```

**Single stage Docker file:**

```bash
FROM node:14.17.1-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
RUN npm install -g serve
EXPOSE 3001
CMD ["serve","-s","build","-l","3001"]
```

Before building the docker image lets give all the necessary permissions to our application:

```bash
chmod +x node_modules/.bin/react-scripts
```

Now lets build the Docker image:

```bash
docker build -t app:1 .
```

Now lets run the docker container

```bash
docker run -p 3001:3001 app:1
```

Access your application from http:// @instance\_ip\_address:3001

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719928747979/010002bb-8c6e-428a-bebe-855769ed1c0a.png align="center")

Check the image size:

```bash
docker images
```

After building the Docker image, you'll notice that its size is around 915 MB, which is quite large.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719928860781/8ab0ce60-49ee-43f1-9289-f38810eeea76.png align="center")

To run the application, we only need the Node runtime, but the current Dockerfile includes steps to build the application from scratch as well as to run it, which takes up a lot of storage. To solve this problem, we can use a multistage Dockerfile with a distroless image.

### Benefits of Multistage Docker Builds

* **Smaller Image Size**: By splitting the build and runtime stages, we get rid of unnecessary files and dependencies, making the final image much smaller.
    
* **Better Security**: Only the stuff we actually need goes into the final image, reducing the risk of security issues from extra tools and libraries.
    
* **Faster Performance**: Smaller images mean quicker deployment and faster network transfers, which is great for CI/CD pipelines.
    
* **Easier Build Process**: Multistage builds make the Dockerfile cleaner and easier to manage. Developers can just focus on the essential steps to build and run the app.
    

### **What is a distroless image?**

A distroless image is a very minimalistic Docker image that contains only the necessary runtime for the application, with hardly any additional packages. For example, if you choose a Python distroless image, it will include only the Python runtime. This means that common commands like `find`, `curl`, `wget`, and `ls` are not available, resulting in errors if you try to use them. This minimalistic approach significantly reduces the image size and attack surface, making the image more secure and efficient.

**Create a Multi stage docker file with distroless image:**

remove the single stage docker file and create a new docker file using below commands

```bash
rm Dockerfile
vim Docker file
```

Multi stage docker file with Distroless image:

```bash
FROM node:14.17.1-alpine AS builder


WORKDIR /app


COPY package*.json ./


RUN npm install


COPY . .
RUN npm install -g serve

FROM gcr.io/distroless/nodejs:14

WORKDIR /app


COPY --from=builder /app/build ./build
COPY --from=builder /usr/local/lib/node_modules/serve /usr/local/lib/node_modules/serve

EXPOSE 3001

CMD ["node", "/usr/local/lib/node_modules/serve/build/index.js", "-s", "build", "-l", "3001"]
```

The above multi-stage Dockerfile contains two stages: one for building the application and another for running it. In the first stage, the application is built. After building, we take the binary image from stage one to stage two. At the end of the Docker build, stage one will not be present. Stage one builds the application, and we copy the binary image to stage two, where we use a distroless image to run the application.

**Build the new Docker image:**

```bash
docker build -t app:2 .
```

**Compare the Image Sizes:**

After building the multi-stage Docker image, you'll notice a significant size reduction. The new image size is just 111MB. This drastic decrease shows the effectiveness of using multi-stage builds and distroless images.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719931719150/31b35bc3-e94b-4f8d-9d4f-4be1f5d2e6b3.png align="center")

### Advantages of Distroless Images

1. **Reduced Attack Surface**: These images only have the stuff your app needs to run, cutting down on potential security risks compared to full OS-based images.
    
2. **Smaller Image Size**: By leaving out unnecessary OS parts and tools, distroless images are much smaller. This means faster downloads, quicker deployments, and lower storage costs.
    
3. **Improved Security**: With fewer packages and libraries, there are fewer chances for security issues. Distroless images focus just on what your app needs to run, making them more secure by default.
    
4. **Simplified Maintenance**: They usually need less upkeep because you don't have to deal with a full OS. Updates mainly focus on app dependencies, not OS components.
    

**Conclusion:**

Utilizing multistage builds alongside Distroless Images represents a robust strategy for crafting lightweight, secure, and efficient Docker containers. This approach significantly reduces image size and minimizes the attack surface, thereby bolstering security and optimizing performance, which is especially advantageous in production environments. Distroless Images focus solely on essential runtime components needed for application execution, ensuring a streamlined deployment process and facilitating easier maintenance. Overall, this combination empowers developers to deliver containers that are both lean and resilient, meeting the demands of modern software deployment practices effectively.

For more distroless images tailored to various programming languages, refer to the [Distroless GitHub Repository](https://github.com/GoogleContainerTools/distroless). This repository provides a comprehensive collection of minimalist container images optimized for specific language runtimes, ensuring lightweight and secure deployments across diverse application environments.