---
title: "AWS Cloud Cost Optimization using Lambda Function"
datePublished: Tue Jul 30 2024 16:42:35 GMT+0000 (Coordinated Universal Time)
cuid: clz8nc5o3000908lbfneq7r2r
slug: aws-cloud-cost-optimization-using-lambda-function
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722357648766/8d69786d-e3de-41ad-8bf7-866777e75ae7.png
tags: cloud, aws, devops, aws-lambda, cloudcostmanagement

---

## Introduction to Serverless Computing

Today, we're going to embark on an exciting journey into the world of serverless computing and explore AWS Lambda, a powerful service offered by Amazon Web Services.

So, what exactly is "serverless computing"? Don't worry; it's not about getting rid of servers entirely. Instead, serverless computing is a cloud computing model where you, as a developer, don't have to manage servers directly. You get to focus on writing and deploying your code, while the cloud provider handles all the underlying infrastructure for you.

### AWS Lambda:

AWS Lambda is a part of the compute family of services and addresses the need for serverless computing.

The main difference between EC2 and Lambda functions is that Lambda functions follow serverless principles. This means that AWS Lambda automatically manages the necessary infrastructure such as operating system, instance type, and storage. When you deploy a Lambda function, it automatically provisions the required resources to run your code and scales them according to the workload. When the code is not running, Lambda automatically scales down, effectively removing the resources and thus reducing costs.

In contrast, with EC2, you are responsible for provisioning and managing the virtual servers, including selecting the OS, instance type, and managing the scaling and termination of instances. After your application has finished running on EC2, you need to manually terminate the instances to stop incurring charges. Lambda abstracts away this manual management, making it a more efficient and cost-effective solution for many use cases.

### Scenario:

Let's say a DevOps engineer has created an EC2 instance and a volume, and their organization feels that the application running on the EC2 instance contains important/sensitive information. To safeguard this data, the DevOps engineer takes daily snapshots (backups) of the volume. One day, the organization decides that the usage of the application is no longer needed, so they delete the EC2 instance and volumes. However, they forget to delete the EBS snapshots. Since the EC2 instance and volumes are deleted, the EBS snapshots are no longer useful, and if they are not deleted, AWS will continue to charge for them. To resolve this issue, a Lambda function can be used to automatically identify and delete snapshots that are not associated with any existing EC2 instances or volumes.

### **Step 1: Create an EC2 Instance and EBS Volume**

To get started, we need to create an EC2 instance with ubuntu as an operating system. We will be using a `t2.small` instance type.

1. Login to your aws console --&gt; Select EC2 --&gt; Click on launch a new instance
    

**Configuring EC2 Instance:**

1. Choose Ubuntu as the operating system (OS).
    
2. Select t2.large as the instance type.
    
3. Set up a key pair for SSH access to the EC2 instance. If you don't have a key pair, click "Create new key pair".
    

Click on the "Launch instance" button to proceed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722348686981/af6e2fb4-5416-484d-9a5d-7eb80a33b412.png align="center")

The instance is up and running, now check the EBS volume attached to this instance.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722348735329/61cfdc13-0283-40b4-83e9-5d8ea1a36ca2.png align="center")

### **Step 2: Create a Snapshot for the EBS Volume**

In the EC2 dashboard, head over to the Snapshots section and make a snapshot for the new EBS volume.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722348878253/2c6d64e3-2e9a-415e-a01a-29e223f8817d.png align="center")

Now, check if the snapshot was created.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722348927128/7cf3cddd-4e8a-41cc-8188-10618bfa547a.png align="center")

### **Step 3: Create a Lambda Function**

Next, let's create a Lambda function to find and delete old EBS snapshots. **Head over to AWS Lambda**: Go to Services -&gt; Lambda -&gt; Create Function.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722348989650/74ff9258-3142-48cd-bd81-2e24d5f4b291.png align="center")

Now, at the "Create Function" screen, choose "Author from scratch." Fill in the basic information and select Python as the runtime.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349057933/7e137330-bd83-43bf-9ac1-e557cf3c15f8.png align="center")

select "Create a new role with basic Lambda permissions" and click on create function

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349110951/592b0873-5a7e-4c00-9260-0849d93dabc4.png align="center")

Now copy and paste the python code in the code source section

```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # Get all EBS snapshots
    response = ec2.describe_snapshots(OwnerIds=['self'])

    # Get all active EC2 instance IDs
    instances_response = ec2.describe_instances(Filters=[{'Name': 'instance-state-name', 'Values': ['running']}])
    active_instance_ids = set()

    for reservation in instances_response['Reservations']:
        for instance in reservation['Instances']:
            active_instance_ids.add(instance['InstanceId'])

    # Iterate through each snapshot and delete if it's not attached to any volume or the volume is not attached to a running instance
    for snapshot in response['Snapshots']:
        snapshot_id = snapshot['SnapshotId']
        volume_id = snapshot.get('VolumeId')

        if not volume_id:
            # Delete the snapshot if it's not attached to any volume
            ec2.delete_snapshot(SnapshotId=snapshot_id)
            print(f"Deleted EBS snapshot {snapshot_id} as it was not attached to any volume.")
        else:
            # Check if the volume still exists
            try:
                volume_response = ec2.describe_volumes(VolumeIds=[volume_id])
                if not volume_response['Volumes'][0]['Attachments']:
                    ec2.delete_snapshot(SnapshotId=snapshot_id)
                    print(f"Deleted EBS snapshot {snapshot_id} as it was taken from a volume not attached to any running instance.")
            except ec2.exceptions.ClientError as e:
                if e.response['Error']['Code'] == 'InvalidVolume.NotFound':
                    # The volume associated with the snapshot is not found (it might have been deleted)
                    ec2.delete_snapshot(SnapshotId=snapshot_id)
                    print(f"Deleted EBS snapshot {snapshot_id} as its associated volume was not found.")
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349234451/d3284fbb-25ec-40db-b1ff-e9b312c98c95.png align="center")

Next, click on "Test" to create an event. Then, in the test event action section, select "Create new event."

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349292392/41543cc7-eb09-4fcf-a94c-fc849bb85354.png align="center")

Now lets configure the Lambda function, set the runtime timeout to 10 seconds to minimize costs, as the longer a Lambda function runs, the more you will be charged.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349370771/cb4ca0b4-4b37-4cbf-8116-c9debdfcc1ef.png align="center")

Now, if you try to test the Lambda function, you will receive an error stating that you are not authorized to perform this operation. This is because we didn't assign the roles for this Lambda function. A role is essentially a set of permissions that allow a service to communicate with other services.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349452523/90f415ad-d658-4864-a941-7d5bac9dc803.png align="center")

to resolve the above issue we need to assign a role to the created lambda function.

to do that

1. Go to Configuration -&gt; Permissions.
    
2. Click on the role name to open it in a new tab.
    
3. Add the following inline policy:
    

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeSnapshots",
                "ec2:DeleteSnapshot",
                "ec2:DescribeVolumes"
            ],
            "Resource": "*"
        }
    ]
}

```

The above role grants the following permissions to the Lambda function:

"ec2:DescribeInstances,ec2:DescribeSnapshots,ec2:DeleteSnapshot, ec2:DescribeVolumes"

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349609761/b2bfe817-b5ff-4707-8c22-5bdc3a411a15.png align="center")

Now, if you test the Lambda function again, it will run successfully. However, it will not delete any snapshots because there is currently an EC2 instance with its volume running.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349708929/096c7544-f05e-46d7-9e75-e294229fa565.png align="center")

Delete the EC2 instance and make sure its volume is also gone.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349808664/40c9eeef-7812-49bc-9c57-06b0ec137dc8.png align="center")

Run the Lambda function again to check that the snapshot is now marked as stale and gets deleted..

The function has been successfully executed and now let’s see if the Snapshot has been deleted.

You will notice that an EBS volume is deleted because its volume is not found.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349858563/0058d556-093f-4870-b6b0-4a2ba9a6c9b1.png align="center")

Let's verify whether the snapshot has been deleted.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722349915437/f107a5be-1744-4f92-ad38-f00b3ef4119f.png align="center")

The Snapshot has been successfully deleted.

**Conclusion**

Congratulations! You have successfully created an AWS Lambda function to optimize storage costs by identifying and deleting stale EBS snapshots. This automated approach not only helps maintain an efficient cloud infrastructure but also reduces unnecessary storage expenses. Great job!

This is just a demonstration of one service that Lambda can handle. The possibilities with AWS Lambda are virtually limitless—you can create Lambda functions for a wide variety of tasks depending on your requirements and the number of services you are using.