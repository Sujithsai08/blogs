---
title: "Automate Your AWS Resource Monitoring Process"
datePublished: Fri Jun 21 2024 19:15:43 GMT+0000 (Coordinated Universal Time)
cuid: clxp2musy000008ichwc8fdxr
slug: automate-your-aws-resource-monitoring-process
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1718997082736/477bd730-fc4f-425b-9e9d-989587977e1f.gif
tags: cloud, docker, aws, bash, kubernetes, cloud-computing, devops, script, aws-lambda, scripting, kunalkushwaha

---

### Introduction:

In the fast-paced world of cloud computing, managing AWS resources efficiently is crucial, especially for students who may inadvertently leave resources running, leading to unnecessary costs. To address this common issue, I developed a Bash script designed to automate the monitoring and management of AWS resources.

### Overview:

As a proactive solution, my Bash script is scheduled to run daily at 2:00 AM using a cron job. Its primary objective is to scan AWS environments for active instances, S3 buckets, and Lambda functions. If any of these resources are found to be active and beyond their intended usage, the script triggers notifications via AWS SNS (Simple Notification Service). These notifications serve as alerts, reminding users to take action, such as stopping or terminating resources, to avoid unnecessary costs.

### Key Features:

1. **Resource Monitoring**: The script systematically checks:
    
    * **EC2 Instances**: Identifies running instances and monitors CPU utilization. If any instance exceeds predefined thresholds, an alert is triggered.
        
    * **S3 Buckets**: Checks for active buckets and monitors storage sizes. If a bucket's storage exceeds specified limits, it sends an alert.
        
    * **Lambda Functions**: Lists active functions and monitors invocation rates and errors. Alerts are sent if these metrics surpass predefined thresholds.
        
2. **Automated Alerts**: Utilizes AWS SNS to send email notifications directly to specified recipients. This ensures that users promptly receive alerts about active resources or excessive usage, enabling timely action.
    
3. **Cost Efficiency**: By automating resource checks and notifications, the script helps in maintaining cost efficiency by minimizing the risk of unused or over-utilized AWS resources.
    

### Step-1 : Create an ec2 instance:

To get started, we need to create an EC2 instance with ubuntu as an operating system. We will be using a `t2.micro` instance type.

1. Login to your aws console --&gt; Select EC2 --&gt; Click on launch a new instance
    

**Configuring EC2 Instance:**

1. Choose Ubuntu as the operating system (OS).
    
2. Select t2.large as the instance type.
    
3. Set up a key pair for SSH access to the EC2 instance. If you don't have a key pair, click "Create new key pair".
    
4. Click on the "Launch instance" button to proceed.
    
    **Connect to the instance:**
    
    once the ec2 instance up and running connect to the instance using ssh
    
    ```bash
     ssh -i path/to/your_key_pair.pem ubuntu@your_ec2_instance_ip
    ```
    

### Step-2 : Install AWS CLI

Install the AWS Command Line Interface (CLI) on your EC2 instance:

```bash
sudo apt update
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### **Step-3 : Configure AWS CLI**

Configure the AWS CLI with your access credentials:

```bash
aws configure
```

You will be prompted to enter your AWS Access Key, Secret Key, region, and output format. To obtain these, go to the AWS console --&gt; Security credentials --&gt; Under Access keys, click on Create access key --&gt; Copy the access key and secret access key, and paste them into the command line interface.

Select your default AWS region and set the output format to JSON.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718988495035/9883f01d-9457-4919-96ce-69dbe7b95c28.png align="center")

### Step - 4 : configuring AWS SNS

* Go to your AWS console and search for SNS.
    
* Click on "Simple Notification Service."
    
* Under "Create topic," give a name to your topic..
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718988684817/ade74536-278f-4ae4-96e7-44c3353a30f6.png align="center")

* Next, click on "Create topic."
    
* Then, click on "Create subscription."
    
    * Under "Protocol," select "Email."
        
    * In the "Endpoint" field, enter your email address.
        
    * Click on "Create subscription.""
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718988790574/f50a7c55-b581-4a9d-b300-3c4e2db00e8d.png align="center")

After creating the subscription, go to your email and accept the subscription to receive notifications.

After accepting the subscription, return to your AWS SNS console, click on the topic, and copy the ARN.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718989065059/9046e892-ffc0-46ce-9aa6-a4fee5a3a144.png align="center")

### Step-5 : create a bash script

Go back to your EC2 command line interface and execute the following command:

The `vim` command is used to create and edit the file.

```bash
vim aws_tracker.sh
```

```bash
#!/bin/bash

##########
# Author: Sujith Sai ##mention your name here
# Date :20-06-2024
# This script automates the process of tracking aws resources 

#########


REGION='us-east-1' # mention your aws region here

SNS="arn:aws:sns:us-east-1:701088230187:alert" # mention your arn here

alert(){
	local message="$1"
	local subject="$2"

	aws sns publish --region $REGION --topic-arn "$SNS" --message "$message" --subject "$subject"
}



check_ec2(){

	echo "listing ec2 instances...."

	local cpu_threshold=5 #mention your cpu_threshold here
	    local start_time=$(date -u -d '15 minutes ago' +%FT%T)
	local end_time=$(date -u +%FT%T)
	local period=300
	Running_instances=$(aws ec2 describe-instances --region "$REGION" --filters Name=instance-state-name,Values=running --query "Reservations[*].Instances[*].InstanceId" --output text)
	if [ -z "$Running_instances" ]; then
		echo "no instances are running"
	else
		

		alert "Instance : $Running_instances is currently running please stop/terminate it..." "Active instances alert"

		for instance in $Running_instances;do
			local utilization=$(aws cloudwatch get-metric-statistics --region "$REGION" --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value="$instance" --start-time "$start_time" --end-time "$end_time" --period "$period" --statistics Average --query 'Datapoints[0].Average' --output text)

			 if (( $(echo "$utilization > $cpu_threshold" | bc -l) ));then
				 alert "High cpu usage detected:$utilization on instance :$Running_instances" "cpu usage alert"
				else
					echo "cpu usage : $utilization on instance : $Running_instances"
			 fi
		 done
	fi
}

check_s3(){
	echo " listing s3 buckets ...."
local s3_buckets=$(aws s3api list-buckets --query "Buckets[].Name" --output text)

if [ -z "$s3_buckets" ];then
	echo "no buckets found"
else
	alert "buckets  :$s3_buckets is currently active please stop/terminate it" "Active buckets alert"

	for bucket in $s3_buckets; do
        echo "Bucket: $bucket"

       
        local storage_info=$(aws s3 ls s3://$bucket --summarize --human-readable | tail -n1)

        
           
            local current_size=$(echo "$storage_info" | awk '{print $3}')

            echo "Current Size: $current_size"
	    local threshold="100 KiB"  #mention your s3_bucket threshold here

                      if (( $(echo "$current_size" ">" "$threshold" | bc -l) )); then
                echo "High storage detected: ${current_size} on bucket: $bucket"
            
                alert "High storage detected: ${current_size} on bucket: $bucket" "S3 Storage Alert"
            fi


        echo
    done
fi
}

check_lambda() {
    echo "Listing Lambda functions..."
    local lambda_functions=$(aws lambda list-functions --region "$REGION" --query "Functions[].FunctionName" --output text)

    if [ -z "$lambda_functions" ]; then
        echo "No Lambda functions found"
    else
        alert "lambda function : $lamda_functions found... please stop/terminate it"
        
        local invocation_threshold=100  
        local error_threshold=1         
        
        for function in $lambda_functions; do
            echo "Checking usage for Lambda function: $function"


            local invocations=$(aws cloudwatch get-metric-statistics --region "$REGION" --namespace AWS/Lambda --metric-name Invocations \
                --dimensions Name=FunctionName,Value="$function" --start-time $(date -u -d '1 day ago' +%FT%T) --end-time $(date -u +%FT%T) \
                --period 86400 --statistics Sum --query 'Datapoints[0].Sum' --output text)


            local errors=$(aws cloudwatch get-metric-statistics --region "$REGION" --namespace AWS/Lambda --metric-name Errors \
                --dimensions Name=FunctionName,Value="$function" --start-time $(date -u -d '1 day ago' +%FT%T) --end-time $(date -u +%FT%T) \
                --period 86400 --statistics Sum --query 'Datapoints[0].Sum' --output text)

            invocations=${invocations:-0}
            errors=${errors:-0}

            echo "Invocations: $invocations"
            echo "Errors: $errors"

            if (( $(echo "$invocations > $invocation_threshold" | bc -l) )); then
                echo "High invocation count detected: $invocations for function: $function"
                alert "High invocation count detected: $invocations for function: $function" "Lambda Usage Alert"
            fi

            if (( $(echo "$errors > $error_threshold" | bc -l) )); then
                echo "Errors detected: $errors for function: $function"
                alert "Errors detected: $errors for function: $function" "Lambda Error Alert"
            fi

            echo 
    done
    fi
}

check_ec2
check_s3
check_lambda
```

**Explanation of the script:**

1. * The `alert()` function sends an alert to AWS SNS and takes two parameters:
        
        1. **message**: The alert message.
            
        2. **subject**: The title of the message.
            
    * `$1` represents the first argument.
        
    * `$2` represents the second argument.
        
    * The function uses the AWS CLI (`aws sns publish`) to send the message to the specified SNS topic identified by `SNS`.
        
2. **check\_ec2()**
    
    * This function checks the status and CPU utilization of EC2 instances in a specific AWS region (`us-east-1` in this case).
        
    * **Listing EC2 Instances**: It retrieves a list of running EC2 instances using the command available in the AWS Command Line Interface documentation.
        
    * ```bash
              (aws ec2 describe-instances --region "$REGION" --filters Name=instance-state-name,Values=running --query "Reservations[*].Instances[*].InstanceId" --output text)
        ```
        
    * If any running instances are found, it will send a message to the registered email via AWS SNS publish.
        
    * **No Instances Running**: If no instances are running, it notifies that no instances are currently active.
        
    * **CPU Utilization Check**: For each running instance, it queries AWS CloudWatch to get the average CPU utilization over the past 15 minutes by the below command
        
    * ```bash
              aws cloudwatch get-metric-statistics --region "$REGION" --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value="$instance" --start-time "$start_time" --end-time "$end_time" --period "$period" --statistics Average --query 'Datapoints[0].Average' --output text
        ```
        
    * **Alerting**:If the CPU utilization exceeds a predefined threshold (`cpu_threshold = 5%`), it sends an alert through AWS SNS publish.
        
    * If it does not exceed the threshold, it simply displays the CPU utilization percentage of the instance.
        
    
    3. **check\_s3()**
        
        * This function monitors S3 buckets in AWS.
            
        * **Listing S3 Buckets**: It retrieves a list of all S3 buckets by the below command
            
        * ```bash
              aws s3api list-buckets --query "Buckets[].Name" --output text
            ```
            
        * If any buckets are found, it will send a message to the registered email via AWS SNS publish.
            
        * **No Buckets Found**: If no buckets are found, it notifies that no buckets exist.
            
        * **Size Check**: For each bucket, it checks the current storage size using the following command
            
        * ```bash
              aws s3 ls s3://$bucket --summarize --human-readable | tail -n1
            ```
            
        * Next, it extracts the current size of the bucket using the following command:
            
        * ```bash
              local current_size=$(echo "$storage_info" | awk '{print $3}')
            ```
            
        * In this command, the "$storage\_info" variable holds the output of the `aws s3 ls` command, specifically the summarized information about the S3 bucket.
            
        * Next, it passes the input of "$storage\_info" to the awk command using the pipe command ("|").
            
        * The `awk '{print $3}'` command prints the third column from each line of input.
            
        
        **Alerting**:
        
        * If the size of any bucket exceeds a specified threshold (`100 KiB`), it sends an alert through AWS SNS publish.
            
    4. **check\_lambda()**
        
        * **Listing Lambda Functions**: It retrieves a list of all Lambda functions.
            
        * **Metrics Check**: For each function, it checks two metrics:
            
            * **Invocations**: Number of times the function has been invoked in the last 24 hours.
                
            * **Errors**: Number of errors encountered by the function in the last 24 hours.
                
        * **Alerting**: It sends alerts if:
            
        * The invocation count exceeds a predefined threshold (`invocation_threshold = 100`).
            
        * The number of errors exceeds a predefined threshold (`error_threshold = 1`).
            
    
    ### Step - 6 : Making script executable
    
    Grant execute permission to the script:
    
    ```bash
    chmod u+x aws_tracker.sh
    ```
    
    **Execute the Script**
    
    Run the script to check the output:
    
    ```bash
    ./aws_tracker.sh
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718992431132/2fe298a8-9166-405f-b878-afd590ea2784.png align="center")
    
    Since I have running instances on my AWS account, I received this automated email using my bash script.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718992539231/17d68ea2-668c-4597-b6ad-2bb8c1c37cdd.png align="center")
    
    Bucket email alert: Since I have running buckets, I received an alert to stop or terminate them if they are unused.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718992552884/34bedeac-43be-4df5-b995-c7d97cc1a404.png align="center")
    
    S3 Storage Alert: The threshold is 100 KB, but my bucket size is 269.5 KB, so I received an alert.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1718992588710/46b0ad4e-b12a-427a-830c-7a8f918ae412.png align="center")
    

### Step - 7 : automating the bash script to run at 2:00am daily

we can automate the bash script to run everyday at 2 am by using cronjob

**set up** `crontab`

1. **Open the Crontab File**:
    
    ```bash
    crontab -e
    ```
    
2. **Add the Cron Job**: Add the following line to schedule your script to run everyday at 2:00am
    
    ```bash
    0 2 * * * /root/aws_tracker.sh
    ```
    
3. **Save and Exit:**
    
    Now the bash script automatically runs daily at 2:00am
    

**Conclusion:**

In conclusion, this Bash script serves as a proactive tool for managing AWS resources efficiently. By automating the monitoring process and integrating with AWS SNS for notifications, it empowers users, particularly students, to maintain control over their cloud usage, enhance cost-effectiveness, and develop valuable skills in cloud resource management.

If you find this blog helpful, please consider sharing and liking it to help others discover the benefits of automating AWS resource management.

Thanks for reading the blog!