---
title: "End-to-End CI/CD Project using Jenkins and argoCD"
datePublished: Sat Jun 15 2024 10:44:46 GMT+0000 (Coordinated Universal Time)
cuid: clxfzqnpx000209l0g7aj7cp2
slug: end-to-end-cicd-project-using-jenkins-and-argocd
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1718605815778/77b2cffa-f18c-46c3-8968-1c413947ac76.png
tags: docker, aws, continuous-integration, continuous-deployment, nodejs, sonarqube, kubernetes, npm, devops, jenkins, cicd-cjy1vtdk2005kjjs17n8couc3, argocd

---

![](https://www.simpleimageresizer.com/_uploads/photos/b8eed390/cicd_project_flow_1600x670.png align="left")

### Introduction:

This CI/CD project automates the deployment of the Car Showroom Node.js application on Kubernetes. Leveraging Jenkins for automation, ArgoCD for GitOps-based deployment, Docker for containerization, and SonarQube for code analysis, we ensure efficient, scalable, and high-quality software delivery. This pipeline integrates testing, Docker image builds, and automatic updates to `deployment.yaml`, streamlining development and ensuring consistent deployments across environments.

Tools/Technologies used:

1. **Jenkins**: Automation server for building, testing, and deploying applications.
    
2. **Node.js-based Application**: The application being built and deployed in this pipeline.
    
3. **npm**: Package manager for Node.js used for dependency management and script running.
    
4. **SonarQube**: Continuous code quality inspection tool.
    
5. **Docker**: Containerization platform used to package the Node.js application and its dependencies.
    
6. **ArgoCD**: Continuous Delivery tool for Kubernetes applications.
    
7. **Shell Scripting**: Used for scripting tasks within the CI/CD pipeline.
    
8. **Kubernetes**: Container orchestration platform for deploying, scaling, and managing containerized applications.
    

### **Prerequisites**

Before we begin, ensure you have the following:

* Node.js application code hosted on a Git repository (link to the repo given down below)
    
* Jenkins server
    
* Kubernetes cluster
    
* AWS Account
    
* Argo CD
    

Git Reporsitory link: [`https://github.com/Sujithsai08/carshowroom_frontend`](https://github.com/Sujithsai08/carshowroom_frontend)

### **Running the Application Locally:**

Before setting up the CI/CD pipeline, let's ensure the Node.js application runs correctly on your local machine:

Step-1:

Clone the Node.js application repository from GitHub by running the below command

```bash
git clone https://github.com/Sujithsai08/carshowroom_frontend.git
cd carshowroom_frontend
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718346784146/eb8cd378-8000-4564-bc23-0e2e43061ad3.png align="center")

Step-2:  
The git repo contains a docker file so build the docker image by the following command

```bash
docker build -t carshowroom:1 .
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718346896489/cf6a2174-14a0-48c3-8071-8dbe7563e221.png align="center")

Step-3:

After the docker image is built, lets run the docker container

To run the container, run the following command:

```bash
docker run -d -p 3000:3000 carshowroom:1
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718347040253/01e45b6d-4891-4f50-a5aa-c26169e30895.png align="center")

Step-4:

Open your web browser and navigate to [`http://localhost:3000`](http://localhost:3000) to verify that the Dockerized application is running as expected.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718347108674/c78f4139-bf36-4328-ab4a-22ba398fd010.png align="center")

Now lets build our end to end cicd project

### Project Structure:

* ### **Initializing a Git Repository:**
    
    To begin our CI/CD journey for deploying a Node.js application, we'll set up a Git repository to host the source code. This repository will serve as the central location where developers can commit changes, and upon these actions (commits or pull requests), our Jenkins pipeline will be automatically triggered through Webhooks.
    
* ### Building with npm:
    
    Next, npm will build the source code from our GitHub repository. This includes installing all necessary dependencies specified in the `package.json` file. During this stage, unit tests and static code analysis specific to Node.js applications will be executed.
    
* ### Static Code analysis using SONARQUBE:
    
    Upon completing the build stage, SonarQube conducts static code analysis on our Node.js application to evaluate its quality, security, and maintainability. A detailed report is then generated, outlining issues like code quality, security vulnerabilities, duplication, complexity, and maintainability
    
* ### Building and Pushing the Docker Image to Docker Hub:
    
    Next, we will build a Docker image for our Node.js application and push it to Docker Hub.
    
* ### Continous Delivery using ArgoCD:
    
    After building and pushing the Docker image, we will use a shell script to update the image reference in the deployment.yaml file within the Git repository. ArgoCD, a Kubernetes controller, monitors the repository. Upon detecting changes (e.g., in deployment.yaml or service.yaml), ArgoCD automatically synchronizes and deploys the application on Kubernetes according to the specifications defined in the deployment.yaml file.
    

### Step-1: Launching an EC2 Instance:

To get started, we need to create an EC2 instance with ubuntu as an operating system. We will be using a `t2.large` instance type.

1. Login to your aws console --&gt; Select EC2 --&gt; Click on launch a new instance
    

**Configuring EC2 Instance:**

1. Choose Ubuntu as the operating system (OS).
    
2. Select t2.large as the instance type.
    
3. Set up a key pair for SSH access to the EC2 instance. If you don't have a key pair, click "Create new key pair".
    
4. Click on the "Launch instance" button to proceed.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718349749401/1efc7985-21de-44d8-828c-8d049a3c4c43.png align="center")

3. **Connect to the instance:**
    
    once the ec2 instance up and running connect to the instance using ssh
    
    ```bash
    ssh -i path/to/your_key_pair.pem ubuntu@your_ec2_instance_ip
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718350022429/1de23814-c638-40c2-8eaf-813bb5897db0.png align="center")
    
4. **Configuring Security Groups for the EC2 Instance:**
    
    To enable external access to Jenkins, follow these steps:
    
    1. Navigate to your EC2 instance and click on "Security".
        
    2. Select "Inbound rules" and click "Edit inbound rules".
        
    3. Add a new rule with the following settings:
        
        * Type: All traffic
            
        * Protocol: All
            
        * Port Range: All
            
        * Source: 0.0.0.0/0 (or specify "Anywhere" for unrestricted access)
            
    4. Click "Save" to apply the changes. This configuration will allow all inbound traffic from any IPv4 address to reach your EC2 instance.
        
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718350537050/ddc0fae0-598e-4da2-9664-91505d4c16a4.png align="center")
    

### Step-2: Installing Jenkins:

As jenkins is a java based application, before installing jenkins we need to install java.

Follow these commands.

```bash
sudo apt update && sudo apt install openjdk-17-jre
java -version
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718350250040/af0dcb6f-4361-491c-bda7-0490c7d87abd.png align="center")

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

### Step-3:Configuring Jenkins:

Access Jenkins using the following URL: `http://{your_ec2_instance_ip_address}:8080`.

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718350821937/7dfed0b0-a0f3-4833-ab43-10da4e7097b6.png align="center")

Use the password to log in, then:

1. Install the suggested plugins.
    
2. Save and finish to access the Jenkins dashboard.
    

### Step-4:Installing necessary plugins:

Before setting up our pipeline, we need to install the following plugins in Jenkins:

To install these plugins, navigate to "Manage Jenkins" &gt; "Manage Plugins" &gt; "Available" and search for each of the following:

* Docker Pipeline
    
* Docker
    
* SonarQube Scanner
    
* NodeJS
    

Install these plugins to enable Docker integration, SonarQube analysis, and Node.js support within Jenkins for our CI/CD setup.

### Step-5:Installing sonarqube on ec2:

Since we are using SonarQube for static code analysis, we need to install SonarQube on our EC2 instance. To install SonarQube, follow these commands:

First, switch to the root user by running the following command:

```bash
sudo su -
```

Next, we need to install the unzip package to extract the files we will download later. To install it, run the following command:

```bash
apt install unzip
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718351546882/887301d3-3748-442d-a975-6fb5ce671d72.png align="center")

next we need to create a new user and download SonarQube:

```bash
adduser sonarqube
su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip sonarqube-9.4.0.54424.zip
chmod -R 755 sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

Access SonarQube at `http://{ec2_instance_ip_address}:9000`.  
Log in to SonarQube with the username: admin and the password: admin.  
Remember to change your password afterward.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718352169303/490bfcb3-884b-4e4e-bc42-44905c362d3e.png align="center")

Since we need to integrate SonarQube with Jenkins, we must generate a token for Jenkins.

go to **Administrator &gt; Security&gt; click on generate new token**.

### **Step 6: Configuring SonarQube in Jenkins**

* Navigate to "Manage Jenkins" &gt; "Manage Credentials"
    
* Select "Global Credentials (unrestricted)"
    
* Click "Add Credentials"
    
* Choose kind as Secret text
    
* Provide an ID as `sonarqube`
    
* Paste the SonarQube authentication token into the "Secret" field
    
* Click "OK" to save the credentials
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718352490103/e6cf9a69-a881-470d-9083-38bf0e9012e0.png align="center")

### **Step 7: Setting Up DockerHub and GitHub Credentials**

As we will be pushing the Docker image to DockerHub and updating the deployment.yaml file in the GitHub repo, we need to provide DockerHub and GitHub credentials to Jenkins.

To add DockerHub credentials in Jenkins:

1. Navigate to your Jenkins dashboard.
    
2. Click on "Manage Jenkins" in the left sidebar.
    
3. Select "Manage Credentials" under the "System Configuration" section.
    
4. Click on "(global)" to manage credentials that apply to all jobs.
    
5. Click "Add Credentials" on the left sidebar.
    
6. Choose "Username with password" from the "Kind" dropdown.
    
7. Enter your DockerHub username and password.
    
    * Username: Enter your DockerHub username
        
    * Password: Enter our DockerHub password
        
8. Set an ID, such as "docker-cred",
    
9. Click "OK" to save the credentials.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718358491415/958b67e6-6c0a-424b-867e-8dd470e1dab7.png align="center")
    

**Adding GitHub Credentials:**  
Generate GitHub Personal Access Token

1. **Log in to GitHub**: Go to [https://github.com/](https://github.com/) and log in.
    
2. **Access Personal Access Tokens**:
    
    * Click your profile icon &gt; Settings.
        
    * In the left sidebar, click "Developer settings".
        
3. **Generate New Token**:
    
    * Click "Personal access tokens" &gt; "Generate new token".
        
    * Click "Generate token" and copy it.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718353217446/312f2ab4-a0d7-47b6-ba44-cc882479b11e.png align="center")
        

**Add GitHub Credentials in Jenkins:**

1. Navigate to your Jenkins dashboard.
    
2. Click on "Manage Jenkins" in the left sidebar.
    
3. Select "Manage Credentials" under the "System Configuration" section.
    
4. Click on "(global)" to manage credentials that apply to all jobs.
    
5. Click "Add Credentials" on the left sidebar.
    
6. Choose "secret text" from the "Kind" dropdown.
    
7. Enter your copied secret text from github
    
8. set an ID as "github"
    

Click "OK" to save the credentials.

### Step-8: Installing Node.js

Since the application is built using Node.js, we need to install Node.js on Jenkins. To configure Node.js, follow these steps:

1. **Go to Jenkins Dashboard**: Navigate to your Jenkins dashboard.
    
2. **Manage Jenkins**: Click on "Manage Jenkins" in the left sidebar.
    
3. **Global Tool Configuration**: Click on "Global Tool Configuration" under the "System Configuration" section.
    
4. **Add Node.js**:
    
    * Scroll down to the "NodeJS" section and click on "Add NodeJS".
        
    * Enter the name as "NodeJS".
        
    * Select the latest version from the dropdown menu.
        

**Save Configuration**: Click on "Save" to apply the changes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718358842286/f9fe5cf7-b685-4b22-80f1-7326d1b5ee0a.png align="center")

### Step-9: Docker Slave Configuration

Since we will be using docker to build the application using dockerfile and push it to the dockerhub we need to install docker on our instance, To install docker follow these steps:

1. Go back to the EC2 command line interface. If you are logged in as the SonarQube user, log out using the following command:
    
    ```bash
    logout
    ```
    
2. or simply use the following command to switch back to the root user
    
    ```bash
    sudo su -
    ```
    
3. Now, let's install Docker on our instance and grant the Jenkins user and Ubuntu user permission to the Docker daemon.
    
    ```bash
    sudo apt install docker.io
    sudo usermod -aG docker jenkins
    sudo usermod -aG docker ubuntu
    systemctl restart docker
    ```
    

### Step-10 : Understanding the Jenkinsfile

```bash
pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS' // Ensure the name matches the NodeJS installation configured in Jenkins
    }
    
    environment {
        FRONTEND_REPO = 'https://github.com/Sujithsai08/carshowroom_frontend.git'
        FRONTEND_BRANCH = 'main'
        SONAR_URL = 'http://18.206.184.8:9000' // Replace this URL with your SonarQube server URL
        REGISTRY_CREDENTIALS = credentials('docker-cred')
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: "${env.FRONTEND_REPO}", branch: "${env.FRONTEND_BRANCH}"
            }
        }
        
        stage('Install Dependencies') {
            steps {
                withEnv(['CI=false']) {
                    sh 'npm install'
                }
            }
        }
        
        stage('Build Application') {
            steps {
                sh 'npm run build' 
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh 'npx sonar-scanner -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL} -Dsonar.projectKey=carshowroom_frontend -Dsonar.projectName="Car Showroom Frontend" -Dsonar.sources=src'
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "sujithsai/carshowroom:${env.BUILD_NUMBER}"
            }
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "carshowroom_frontend"
                GIT_USER_NAME = "Sujithsai08"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                    git config user.email "sujithsai.sirimalla33@gmail.com"
                    git config user.name "${GIT_USER_NAME}"
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" manifests/deployment.yaml
                    git add manifests/deployment.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GIT_USER_NAME}:${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
            }
        }
    }
    
    post {
        always {
            // Clean up any workspace or resources
            cleanWs()
        }
    }
}
```

Let's break down the Jenkinsfile. Our Jenkinsfile consists of six stages:

1. **Clone Repository**:
    
    * The `git` step uses the repository URL and branch name defined in the environment variables (`FRONTEND_REPO` and `FRONTEND_BRANCH`) to clone the repository into the Jenkins workspace.
        
2. **Install Dependencies**:
    
    * The `sh 'npm install'` command installs the dependencies listed in the `package.json` file.
        
    * We use `withEnv(['CI=false'])` to ensure that certain CI-specific behaviors in some packages are disabled. This is particularly useful because our website has some code quality errors. By setting `CI` to `false`, Jenkins will not halt the build process when it encounters these code quality errors, allowing the pipeline to proceed.
        
3. **Build Application**:
    
    * The `sh 'npm run build'` command runs the build script defined in the `package.json` file.
        
4. **SonarQube Analysis**:
    
    * Performs static code analysis using SonarQube to assess code quality, security vulnerabilities, and maintainability of the source code.
        
5. **Build and Push Docker Image**:
    
    * The `sh 'docker build -t ${DOCKER_IMAGE} .'` command builds the Docker image with the current build number as the tag.
        
    * The `def dockerImage = docker.image("${DOCKER_IMAGE}")` line defines the Docker image.
        
    * The `docker.withRegistry('[https://index.docker.io/v1/](https://index.docker.io/v1/)', "docker-cred") { dockerImage.push() }` step logs into DockerHub using the provided credentials and pushes the image.
        
6. **Update Deployment File**:
    
    The `withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')])` step securely provides the GitHub authentication token, updates the `deployment.yaml` file with the new Docker image tag, configures Git, commits the change, and pushes it to the repository, while `sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yaml` replaces the placeholder with the current build number.
    

### Step-11: Understanding the Dockerfile

```bash
FROM node:16-alpine
RUN npm install -g npm@8
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
RUN npm install -g serve
EXPOSE 3001
CMD ["serve", "-s", "build", "-l", "3001"]
```

Lets break down the dockerfile:

* **FROM**: Specifies the base image to build upon. Here, it uses `node:16-alpine`, a lightweight version of Node.js version 16 based on Alpine Linux.
    
* **RUN:** Executes commands in the Docker image. This command installs a specific version of npm (`npm@8`) globally (`-g`).
    
* **WORKDIR:** Sets the working directory inside the container.
    
* **COPY:** Copies files from the Docker host into the Docker image's filesystem. Here, it copies `package.json` and `package-lock.json` from the Docker host's current directory (`./`) into the `/app` directory in the Docker image.
    
* **RUN:** Executes the `npm install` command. This installs the dependencies listed in `package.json` into the `/app/node_modules` directory.
    
* **COPY:** Copies all files and directories from the Docker host's current directory into the `/app` directory of the Docker image.
    
* **RUN:** Executes the `npm run build` command. This compiles the application source code.
    
* **RUN:** Installs the `serve` package globally within the Docker image. `serve` is a simple HTTP server.
    
* **EXPOSE:** Informs Docker that the container listens on the specified network ports at runtime. Here, it exposes port 3001 for incoming connections.
    

**CMD:** Provides the default command to run when the Docker container starts. It specifies running `serve` with `-s build -l 3001`, which serves the static files from the `/app/build` directory on port 3001.

### **Step 12: Building the Pipeline:**

* **Go to Jenkins Dashboard**.
    
* **Create a New Job**:
    
    * Click on "New Item".
        
    * Enter a name for your job.
        
    * Select "Pipeline" and click "OK".
        
* **Configure the Job**:
    
    In the job configuration page, under "Pipeline", select "Pipeline script from SCM".
    
    * **Select SCM**: Choose "GIT".
        
    * **Repository URL**: Enter [`https://github.com/Sujithsai08/carshowroom_frontend`](https://github.com/Sujithsai08/carshowroom_frontend).
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718391582411/de68cee7-cd09-4dd4-ba58-2ca58ba35b23.png align="center")
        
    * **Branches to build**: Ensure it is set to `*/main`.
        
    * **Script Path**: Enter the path to your Jenkinsfile, e.g., `jenkins/Jenkinsfile`.
        
    * **Save** your configuration.
        
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718391954580/3bfbfcd8-102f-4378-88b7-83c636aa0e6f.png align="center")
        

To run the job, go to the job's page and click the "Build Now" link on the left-hand side.

The pipeline ran successfully.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718392214919/cee3b44d-fdc1-4791-bf46-0d87c1d213f8.png align="center")

**Review SonarQube Results**:

* SonarQube results are available and have been processed successfully.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718392282147/2bc3f3f6-ac8b-4532-a087-f9c99fecb495.jpeg align="center")
    
    Verify **DockerHub Image**:
    
    * DockerHub now contains the new image that was built during the pipeline execution.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718444341484/d59db53d-f030-4fad-a4bb-8c68a7989a45.png align="center")
        
        **Review Deployment.yaml file**:
        
        * The image version inside the `deployment.yml` file has also been updated accordingly (updated to version 5 according to our jenkins job build number).
            
            ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718444262881/73b784f1-f73e-4274-b311-1c368790a9b0.png align="center")
            

### Step-13: Continous Delivery Using ArgoCD

We need ArgoCD to deploy our application. After completing all the stages in the CI process, ArgoCD monitors our repository. As soon as any changes or commits are made, ArgoCD triggers and deploys our application on Kubernetes.

Assuming you have already installed and configured ArgoCD on your PC, if you haven't, check out my blog on how to install and configure ArgoCD: [Installing and Configuring ArgoCD - A Step-by-Step Guide](https://sujith08.hashnode.dev/installing-and-configuring-argocd-a-step-by-step-guide).

Now login to the argoCD Dashboard by username as admin and password: obtained while configuring argocd

![Login Page](https://cdn.hashnode.com/res/hashnode/image/upload/v1718195382698/69f03d47-d490-4226-8634-26d5bb58e912.png align="left")

**After logging in, click on "+New App" and enter the following configurations: Application Name: project, Project Name: default, Sync Policy: auto, Repository URL:**[`https://github.com/Sujithsai08/carshowroom_frontend`](https://github.com/Sujithsai08/carshowroom_frontend), Revision: HEAD, Path (path of deployment.yml): `manifests`, Cluster URL: [`https://kubernetes.default.svc/`](https://kubernetes.default.svc/), Namespace: default, then click on create.

* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718443122736/94a46fb7-80f3-46cf-9bd9-01424284feff.png align="center")
    
    After some time, ArgoCD automatically syncs and deploys the application on the Kubernetes cluster.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718443194028/4e3856e3-59fc-43e4-ba40-43e66b018595.png align="center")
    
* Now, let's verify that the application was deployed successfully.  
    To verify, execute the following commands:
    
    ```bash
    kubectl get pods
    kubectl get svc
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718443323360/39eaaa40-1693-421e-8941-4f450ca70516.png align="center")
    
* Now let's access the application.  
    To access the application, first list the services in Kubernetes. To do that, execute the following command:
    
    ```bash
    minikube service list
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718443710173/42704473-15f6-4537-a51e-7548e60fbc8f.png align="center")
    
* Next execute the following command to access our application via tunneling
    
    ```bash
    minikube service carshowroom-app-service
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718443695880/212825cb-e2cd-4db2-8be6-7fbad2192854.png align="center")
    
* Finally, you should be able to access your application by pasting the url in your browser, by completing the deployment process. Congratulations on successfully deploying your application using ArgoCD and Minikube
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718443977263/1fc532cc-3921-42cf-ad77-74a01a2a82db.png align="center")
    
    Thanks for reading the blog!