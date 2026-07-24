# ks-aws-lambda-devops-assignments
**AWS Cloud deployment**
## **Task 1.**
## ** Automated S3 Bucket Cleanup (Objects Older Than 30 Days)**

**Step 1:** As recommended, region selected after login in AWS account is **us-east-1**
<img width="1890" height="287" alt="image" src="https://github.com/user-attachments/assets/5572688e-1d52-4291-8ebd-b8155a88a2f7" />

**Step 2:** Created budget alert for $1.
<img width="1731" height="901" alt="image" src="https://github.com/user-attachments/assets/0a54bdb0-5a0b-419c-a296-ddc4b32581ae" />
<img width="1540" height="136" alt="image" src="https://github.com/user-attachments/assets/723a0d16-3bc8-4f4e-b610-c8fadf237f57" />

**Step 3:** Created new repository **"saurabhkumar2k/ks-aws-lambda-devops-assignments.git"**. Clone the same in my local.
**Step 4:** Ceated question wise folders.
**Step 5:** Created S3 bucket
<img width="1877" height="803" alt="image" src="https://github.com/user-attachments/assets/5b42e6bd-7839-463e-baea-00dd920c00f5" />
**Step 6:** Upload few text files.
<img width="1876" height="905" alt="image" src="https://github.com/user-attachments/assets/5374d00b-3136-4441-96c8-619d719fdf70" />
<img width="1883" height="902" alt="image" src="https://github.com/user-attachments/assets/b7899bd2-6393-464b-93f5-1a5814a78e8a" />
**Step 7:** Created IAM role having inline policy.
<img width="1867" height="907" alt="image" src="https://github.com/user-attachments/assets/68a2b8c6-4def-4f3d-b18e-fbfb52a1d5fb" />

The inline policy implemented as under:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:ListBucket"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::devops-cleanup-bucket"
    },
    {
      "Action": [
        "s3:DeleteObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::devops-cleanup-bucket/*"
    }
  ]
}
<img width="1893" height="916" alt="image" src="https://github.com/user-attachments/assets/bcc6056e-564e-4eed-8f23-14d875e95d24" />

<img width="1878" height="908" alt="image" src="https://github.com/user-attachments/assets/21b26e66-1ebf-4cc2-9b49-c094637c3559" />

**Step 8:** Created Lambda function using Boto3

<img width="1882" height="912" alt="image" src="https://github.com/user-attachments/assets/212a6896-5374-428f-bac0-5feb6c5c6912" />

Sharing Lambda code that is being implemented while creating function:-
import boto3
from datetime import datetime, timezone, timedelta

s3 = boto3.client('s3')

BUCKET_NAME = "devops-cleanup-bucket"

def lambda_handler(event, context):

    # Delete objects older than 5 minutes
    cutoff_time = datetime.now(timezone.utc) - timedelta(minutes=5)

    paginator = s3.get_paginator('list_objects_v2')

    deleted = []

    for page in paginator.paginate(Bucket=BUCKET_NAME):

        for obj in page.get('Contents', []):

            if obj['LastModified'] < cutoff_time:

                s3.delete_object(
                    Bucket=BUCKET_NAME,
                    Key=obj['Key']
                )

                deleted.append(obj['Key'])

    print("Deleted Objects:", deleted)

    return {
        "statusCode": 200,
        "deleted_count": len(deleted),
        "deleted": deleted
    }

<img width="1888" height="922" alt="image" src="https://github.com/user-attachments/assets/f176f2d8-4bb4-40fd-823d-355c570f9a46" />

<img width="1881" height="912" alt="image" src="https://github.com/user-attachments/assets/95ad1992-e058-4657-8270-9ceef35e26e6" />

<img width="1893" height="927" alt="image" src="https://github.com/user-attachments/assets/593c5a70-f040-4c0a-82da-035b04a10414" />


I tested my bucket for its emptyness after 5 minutes because I have given age_minutes=5 instead of age_days=30.
<img width="1912" height="913" alt="image" src="https://github.com/user-attachments/assets/1ebdf93e-323e-492e-a3a2-deb76ccd6c2c" />


## **Task 2.**
## **Automated EBS Snapshot Creation and Cleanup**
**Step 1** : Open EC2 and navigate to Volumes in Elastic Block Store
<img width="1907" height="907" alt="image" src="https://github.com/user-attachments/assets/1b5c9bfe-bad6-4ed7-adb1-f29bd987d157" />

**Step 2** : Created volume with some details as under in scrennshot
<img width="1812" height="821" alt="image" src="https://github.com/user-attachments/assets/a85e6f5d-2299-45ec-8a6e-32c0c93c46dd" />
Got the volume ID
<img width="1618" height="447" alt="image" src="https://github.com/user-attachments/assets/5d6a2a02-b865-40b5-a5f1-d960fa4bea6b" />

**Step 3** : Created IAM role with inline policy
<img width="1898" height="832" alt="image" src="https://github.com/user-attachments/assets/ac744aee-a419-44df-8535-871042611041" />

<img width="1872" height="916" alt="image" src="https://github.com/user-attachments/assets/ecc71fdc-70ce-47a0-b3ef-c4ae3e55058f" />

<img width="1892" height="916" alt="image" src="https://github.com/user-attachments/assets/5e3a534b-1267-40c1-9fed-64bc7d4f61f7" />

**Step 4** : Lambda configuration has neen initiated. Added lambda code with:
import boto3
from datetime import datetime, timezone, timedelta

ec2 = boto3.client("ec2")

VOLUME_ID = "YOUR_VOLUME_ID"

def lambda_handler(event, context):

    snapshot = ec2.create_snapshot(
        VolumeId=VOLUME_ID,
        Description="Lambda Automated Backup"
    )

    snapshot_id = snapshot["SnapshotId"]

    ec2.create_tags(
        Resources=[snapshot_id],
        Tags=[
            {
                "Key": "CreatedBy",
                "Value": "Lambda-Backup"
            }
        ]
    )

    print(f"Created Snapshot: {snapshot_id}")

    cutoff_date = datetime.now(timezone.utc) - timedelta(days=30)

    snapshots = ec2.describe_snapshots(
        OwnerIds=["self"]
    )["Snapshots"]

    for snap in snapshots:

        tags = {
            tag["Key"]: tag["Value"]
            for tag in snap.get("Tags", [])
        }

        if tags.get("CreatedBy") == "Lambda-Backup":

            if snap["StartTime"] < cutoff_date:

                ec2.delete_snapshot(
                    SnapshotId=snap["SnapshotId"]
                )

                print(
                    f"Deleted Snapshot: {snap['SnapshotId']}"
                )

    return {
        "status": "success",
        "snapshot": snapshot_id
    }

  **Step 5** : Deploy and tested Lambda configuration.
    Got errors like 
   <img width="1412" height="722" alt="image" src="https://github.com/user-attachments/assets/ec4d25ce-f5ec-4f16-862f-a44d8479a4f1" />
   
**Step 6** : Troubleshooting has started with JSON Policy. Updated JSON like below as in screenshot
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SnapshotManagement",
      "Effect": "Allow",
      "Action": [
        "ec2:CreateSnapshot",
        "ec2:DescribeSnapshots",
        "ec2:DeleteSnapshot",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    }
  ]
}
   

<img width="1888" height="815" alt="image" src="https://github.com/user-attachments/assets/2eab02a5-8f9f-4844-b72c-98d6986dd50d" />



<img width="1543" height="757" alt="image" src="https://github.com/user-attachments/assets/e4d23343-e822-4031-95fd-2fdfdec0be44" />

<img width="1896" height="875" alt="image" src="https://github.com/user-attachments/assets/06ea5d9c-e3d4-4364-b002-0a2801aa6558" />

**Step 7** : Now open IAM role and created inline policy with JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SnapshotManagement",
      "Effect": "Allow",
      "Action": [
        "ec2:CreateSnapshot",
        "ec2:DescribeSnapshots",
        "ec2:DeleteSnapshot",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    }
  ]
}

<img width="1885" height="923" alt="image" src="https://github.com/user-attachments/assets/2d2d5d58-fde1-4e8d-9e23-ce557691c9a4" />

**Step 8**: Now back to Lamda. While deploying created a test event.

Response I got is like as under:
Response: { "errorMessage": "An error occurred (UnauthorizedOperation) when calling the CreateSnapshot operation: You are not authorized to perform this operation. User: arn:aws:sts::985818273957:assumed-role/EBSSnapshotAutomation-role-6dsldlka/EBSSnapshotAutomation is not authorized to perform: ec2:CreateSnapshot on resource: arn:aws:ec2:us-east-1::snapshot/* because no identity-based policy allows the ec2:CreateSnapshot action. Encoded authorization failure message.

**Step 9** : Started role verification
<img width="1890" height="848" alt="image" src="https://github.com/user-attachments/assets/0eaffc29-fc6e-46a2-8f97-367abe04685f" />

**Step 10** Checked attached policy in permission tab. And I replaced the policy with:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SnapshotManagement",
      "Effect": "Allow",
      "Action": [
        "ec2:CreateSnapshot",
        "ec2:DescribeSnapshots",
        "ec2:DeleteSnapshot",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    }
  ]
}
<img width="1890" height="823" alt="image" src="https://github.com/user-attachments/assets/32b219cf-a84d-42b4-8b8e-6b20d8f92534" />

**Step 11** Try creating a snapshot manually:
EC2 Console
Volumes
Select volume:vol-066452e6332aa69bb
Actions → Create Snapshot

**Step 12** Back to Lambda test run and found
<img width="1718" height="717" alt="image" src="https://github.com/user-attachments/assets/b9b6159c-1611-4950-a3ea-e8557eb9b071" />


**Next Step: Verify Snapshot in EC2 Console**
<img width="1910" height="767" alt="image" src="https://github.com/user-attachments/assets/5940e2ad-be5f-4fbc-827d-4fe11f5dc42d" />

<img width="1906" height="773" alt="image" src="https://github.com/user-attachments/assets/d75350d4-f948-458e-9301-5551b05b850a" />

## **Task 3.**
## **Auto-Tagging EC2 Instances on Launch**

Whenever a new EC2 instance enters the Running state, as per the instruction we need to add tag LaunchDate=<current date>, add tag Environment=Dev and 
log the action in CloudWatch.

**Step 1**: Navigate to Lambda -> Create function
<img width="1887" height="892" alt="image" src="https://github.com/user-attachments/assets/004087c3-43b2-41a5-befe-7219c7ce0464" />

<img width="1893" height="870" alt="image" src="https://github.com/user-attachments/assets/27b9a61f-6216-419e-9e63-b13c18ef213b" />

**Step 2**: Creating IAM policy.
Navigated to Configuration -> Permission
<img width="1888" height="912" alt="image" src="https://github.com/user-attachments/assets/27123289-b537-43e4-ae59-91e21ad1625f" />

Go to Execution Role -> Add Permission -> Inline Policy ->JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2Tagging",
      "Effect": "Allow",
      "Action": [
        "ec2:CreateTags",
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}

<img width="1876" height="922" alt="image" src="https://github.com/user-attachments/assets/3528900d-21c0-43f5-9dd4-b85d12bdeab5" />

<img width="1891" height="906" alt="image" src="https://github.com/user-attachments/assets/e9a5b7d0-0642-4a7e-9089-26b8c7674d50" />

**Step 3**:
Navigate to the Lambda Code Editor
Go to AWS Console -> Lambda -> Functions ->EC2AutoTagger:
**Code for the lambda_functions.py**
import boto3
from datetime import datetime

ec2 = boto3.client("ec2")

def lambda_handler(event, context):

    instance_id = event["detail"]["instance-id"]

    launch_date = datetime.utcnow().strftime("%Y-%m-%d")

    ec2.create_tags(
        Resources=[instance_id],
        Tags=[
            {
                "Key": "LaunchDate",
                "Value": launch_date
            },
            {
                "Key": "Environment",
                "Value": "Dev"
            }
        ]
    )

    print(f"Tagged Instance {instance_id}")

    return {
        "status": "success",
        "instance": instance_id
    }

**Step 4** : Launch new EC2 instance
Go to EC2 → Instances -> Click Launch Instance

**Launching new instance with configuration**
Name: TestServer
AMI: Amazon Linux 2023
Instance Type: t2.micro (Free Tier)
Used existing key pair
Launch Instance
<img width="1912" height="926" alt="image" src="https://github.com/user-attachments/assets/510f57c5-dd1d-4e58-84c8-aba2a48346e9" />

**Step 5** : Created test event named **EC2RunningEvent**
After saving the event name, AWS will open a JSON editor.
{
  "detail": {
    "instance-id": "i-0f1a7590ad052e22a"
  }
}

**Note** i-0f1a7590ad052e22a is the new EC2 instance ID.

**Step 5** Now then Test and got response as under:
<img width="1903" height="917" alt="image" src="https://github.com/user-attachments/assets/92bc593a-d568-48fb-bdc1-5ee47641acf0" />

**Step 6** : Open EC2 instance and navigated to Tags tab and found EC2 auto tagging is working
<img width="1872" height="796" alt="image" src="https://github.com/user-attachments/assets/9de8c543-37fa-4e6d-b81f-7a99c6b1df13" />

**Step 7** : Creating EventBridge Rule
Go to Amazon EventBridge and Create Rule

<img width="1897" height="878" alt="image" src="https://github.com/user-attachments/assets/76bd22e2-6b8e-424b-8812-58e87abbbdc9" />

<img width="1881" height="827" alt="image" src="https://github.com/user-attachments/assets/14621686-18b7-4e64-98c9-82aa2e9fbe8f" />

<img width="1877" height="803" alt="image" src="https://github.com/user-attachments/assets/3666771c-3203-4e11-9eb7-b44c359f9e38" />

<img width="1881" height="935" alt="image" src="https://github.com/user-attachments/assets/a3d47294-72d6-4659-8d99-8bbc30bdd2b0" />

<img width="1913" height="833" alt="image" src="https://github.com/user-attachments/assets/61059797-fd7e-4957-a9dc-1dac2eef40dd" />

<img width="1907" height="826" alt="image" src="https://github.com/user-attachments/assets/92824e18-7e76-4722-ac63-fdf7ef57b37f" />

<img width="1901" height="900" alt="image" src="https://github.com/user-attachments/assets/e21d39d3-51f3-4114-94f9-dd6a0546b460" />

<img width="1887" height="927" alt="image" src="https://github.com/user-attachments/assets/1ba541b9-7b1b-47c8-b2b1-6e90222ecda0" />


## **Task 4.**
## **Daily AWS Cost Alert Using Cost Explorer API and SNS**

**Step 1** : Creating SNS topic
Amazon Simple Notification Service - Create Topic
<img width="1917" height="968" alt="image" src="https://github.com/user-attachments/assets/cb9632be-3222-41c9-82a7-032239e092fe" />

<img width="1881" height="907" alt="image" src="https://github.com/user-attachments/assets/4e92ed33-17ce-4a2c-83a3-d2318bc463c4" />

**Step 2** : Create Subscription
<img width="1893" height="795" alt="image" src="https://github.com/user-attachments/assets/d2694afa-c391-4a63-8dba-c8cc9f8c6665" />
<img width="1877" height="877" alt="image" src="https://github.com/user-attachments/assets/97cd1423-1691-4fe4-9c66-70db71cd1ab2" />

**Step 3** : Confirm email subscription
<img width="1835" height="787" alt="image" src="https://github.com/user-attachments/assets/bbf73976-eaa4-4909-ae3d-49b70315ec00" />
<img width="953" height="540" alt="image" src="https://github.com/user-attachments/assets/7073ac30-d5c9-4b3a-894c-b9002d8891c2" />

**Step 4** : Creating Lambda function
<img width="1888" height="930" alt="image" src="https://github.com/user-attachments/assets/d7d1d654-793d-4392-9583-93bf15da3a7f" />

**Step 6**: Create IAM Policy
<img width="1877" height="912" alt="image" src="https://github.com/user-attachments/assets/24c72bde-f02e-4e92-8308-a2196a4b6aa9" 

  Adding inline policy with JSON code in policy editor
  {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CostExplorer",
      "Effect": "Allow",
      "Action": [
        "ce:GetCostAndUsage"
      ],
      "Resource": "*"
    },
    {
      "Sid": "SNSPublish",
      "Effect": "Allow",
      "Action": [
        "sns:Publish"
      ],
      "Resource": "arn:aws:sns:us-east-1:985818273957:CostAlertTopic:a9a14a74-27b1-4083-a409-d147e0fa31ab"
    }
  ]
}


  <img width="1888" height="901" alt="image" src="https://github.com/user-attachments/assets/2d673cab-dccc-4f70-b6e8-03c12b26382d" />

<img width="1882" height="900" alt="image" src="https://github.com/user-attachments/assets/50974af4-9acd-43aa-beaa-a0d302594ab4" />

**Step 7** : Navigae to Lambda function -- DailyCostAlert

lambda_function.py will be updated with code:
import boto3
from datetime import date

ce = boto3.client('ce')
sns = boto3.client('sns')

SNS_TOPIC_ARN = "arn:aws:sns:us-east-1:985818273957:CostAlertTopic:a9a14a74-27b1-4083-a49-d147e031ab"

THRESHOLD = 0.01   # testing
# THRESHOLD = 50    # production


def lambda_handler(event, context):

    today = date.today()

    start_date = today.replace(day=1).strftime('%Y-%m-%d')
    end_date = today.strftime('%Y-%m-%d')

    response = ce.get_cost_and_usage(
        TimePeriod={
            'Start': start_date,
            'End': end_date
        },
        Granularity='MONTHLY',
        Metrics=['UnblendedCost']
    )

    amount = float(
        response['ResultsByTime'][0]
        ['Total']['UnblendedCost']['Amount']
    )

    print(f"Current MTD Cost: ${amount}")

    if amount > THRESHOLD:

        message = (
            f"AWS Cost Alert\n\n"
            f"Current Month-To-Date Cost: ${amount}\n"
            f"Threshold: ${THRESHOLD}"
        )

        sns.publish(
            TopicArn=SNS_TOPIC_ARN,
            Subject='AWS Cost Alert',
            Message=message
        )

        print("Alert Sent")

    else:
        print("Threshold Not Reached")

    return {
        "statusCode": 200,
        "cost": amount
    }

    <img width="1876" height="907" alt="image" src="https://github.com/user-attachments/assets/aa4add53-b5e9-43f3-9994-b9a0007437e0" />

  **Step 8** : Deploy
  **Step 9** : Create Test Event
  <img width="487" height="660" alt="image" src="https://github.com/user-attachments/assets/807f2013-6919-4a3f-ae5f-9e76ee01e51b" />

  While Testing I got permission error.
  <img width="857" height="525" alt="image" src="https://github.com/user-attachments/assets/35ac33f3-207e-4986-be4a-fd4200fcd05d" />

  **Step 10** : Need to rectify: 
  **Remedy: Mistakenly wrong arn has been placed in lambda_function.py**
  **Resolved:**
  <img width="1860" height="833" alt="image" src="https://github.com/user-attachments/assets/e5573e25-0bb0-4383-a249-3cc741d02901" />

  **At last unsubscrition mail has been received**
  <img width="1571" height="808" alt="image" src="https://github.com/user-attachments/assets/f8c8d4ae-4932-4b80-882c-56d02110aad0" />


## **Task 5.**
## **Restore an EC2 Instance from the Latest Snapshot**

**Step 1** : Creating lambda function 
<img width="1893" height="903" alt="image" src="https://github.com/user-attachments/assets/1f9ded14-8a33-44df-af56-421d766d6980" />
**Step 2** Inline policy in IAM role : Lambda → Configuration → Permissions → Execution Role
JSON Code for inline policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SnapshotAccess",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeSnapshots",
        "ec2:DescribeImages",
        "ec2:RegisterImage",
        "ec2:RunInstances",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    }
  ]
}

Policy has been created

<img width="1897" height="920" alt="image" src="https://github.com/user-attachments/assets/ccd75344-a38a-454c-967e-d22b8a420d31" />

**Step 3** : Get volume ID from EC2 instance which is required to restore.
<img width="1902" height="913" alt="image" src="https://github.com/user-attachments/assets/649fc045-7962-437d-907c-40c973863180" />

**Step 4** : Go to the lambda function -> Code
Update code with :
import boto3
from datetime import datetime

ec2 = boto3.client('ec2')

VOLUME_ID = "vol-0b75424c3d99583"

def lambda_handler(event, context):

    snapshots = ec2.describe_snapshots(
        Filters=[
            {
                'Name': 'volume-id',
                'Values': [VOLUME_ID]
            }
        ],
        OwnerIds=['self']
    )['Snapshots']

    if not snapshots:
        raise Exception("No snapshots found")

    latest_snapshot = sorted(
        snapshots,
        key=lambda x: x['StartTime'],
        reverse=True
    )[0]

    snapshot_id = latest_snapshot['SnapshotId']

    print(f"Latest Snapshot: {snapshot_id}")

    ami_name = f"restored-ami-{datetime.utcnow().strftime('%Y%m%d%H%M%S')}"

    response = ec2.register_image(
        Name=ami_name,
        RootDeviceName='/dev/xvda',
        VirtualizationType='hvm',
        Architecture='x86_64',
        BlockDeviceMappings=[
            {
                'DeviceName': '/dev/xvda',
                'Ebs': {
                    'SnapshotId': snapshot_id,
                    'DeleteOnTermination': True,
                    'VolumeType': 'gp3'
                }
            }
        ]
    )

    ami_id = response['ImageId']

    print(f"AMI Created: {ami_id}")

    instance = ec2.run_instances(
        ImageId=ami_id,
        InstanceType='t2.micro',
        MinCount=1,
        MaxCount=1
    )

    instance_id = instance['Instances'][0]['InstanceId']

    ec2.create_tags(
        Resources=[instance_id],
        Tags=[
            {
                'Key': 'RestoredFrom',
                'Value': snapshot_id
            }
        ]
    )

    print(f"Instance Created: {instance_id}")

    return {
        "statusCode": 200,
        "snapshot": snapshot_id,
        "ami": ami_id,
        "instance": instance_id
    }

    <img width="1870" height="905" alt="image" src="https://github.com/user-attachments/assets/94130f64-f133-4125-90e1-991c89fda3ee" />

  **Step 5** : Deploy
  <img width="1857" height="895" alt="image" src="https://github.com/user-attachments/assets/45f249ce-e123-4b5f-b243-c53fb6548eb4" />

  **Step 6** : Test

  Result is as under
  <img width="1906" height="936" alt="image" src="https://github.com/user-attachments/assets/2ff2ce3f-5085-460a-b2ff-d776e38efe5b" />

  Now screenshot of 2 EC2 instances
  i-0f1a7590ad052e22a → TestServer (your original instance)
  i-020e0a6f9da9a2432 → likely the restored instance created by Restore an EC2 Instance from the Latest Snapshot
  
  <img width="1910" height="950" alt="image" src="https://github.com/user-attachments/assets/3e35e38d-0c15-406e-b5c9-fafab18b9798" />


## **Task 6.** :Audit S3 Buckets for Public Access and Notify

**Step 1** : Creating  topic :Standard type

<img width="1880" height="905" alt="image" src="https://github.com/user-attachments/assets/500b335f-49f3-416b-abbb-2d8ec98b1509" />

**Step 2** : Creating email subscription
<img width="1912" height="898" alt="image" src="https://github.com/user-attachments/assets/e8e281db-f785-4ffa-b929-905b130c2c91" />

<img width="1867" height="787" alt="image" src="https://github.com/user-attachments/assets/00e27bf5-3bd0-4fc8-9ff0-d9d26ee05095" />

**Step 3** :Got an email
<img width="1420" height="407" alt="image" src="https://github.com/user-attachments/assets/2dfba1c2-c162-42ee-ae39-862109eaaaf7" />

**Step 4** :Confirmed subscription
<img width="771" height="315" alt="image" src="https://github.com/user-attachments/assets/ce789261-644e-41e0-b3e7-d5fa18a43179" />

**Step 5** :Checked the status for "Confirmed"
<img width="1462" height="487" alt="image" src="https://github.com/user-attachments/assets/443fb1af-9e85-4c96-a7a0-ae25d071627c" />

**Step 6** : Creating a lambda function
<img width="1863" height="903" alt="image" src="https://github.com/user-attachments/assets/cbe6dcb4-46d9-4993-9b3b-3a7ce6c9782a" />

**Step 7** : Now adding inline policy
Navigated to Lambda → S3PublicAccessAudit → Configuration → Permissions → Click Role Name → Add Inline Policy

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3Audit",
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:GetBucketPublicAccessBlock",
        "s3:GetBucketPolicyStatus",
        "s3:GetBucketAcl"
      ],
      "Resource": "*"
    },
    {
      "Sid": "SNSPublish",
      "Effect": "Allow",
      "Action": [
        "sns:Publish"
      ],
      "Resource": "arn:aws:sns:us-east-1:985818273957:S3PublicAccessAlert"
    }
  ]
}

Policy has been created
<img width="1882" height="952" alt="image" src="https://github.com/user-attachments/assets/c304c264-f922-40d6-a5bc-1512fe033edd" />

**Step 8**
As per the assignment specifically requires a Lambda Function (Boto3) that:
Lists all S3 buckets
Checks Public Access Block settings
Checks bucket policy status (IsPublic)
Checks ACL permissions
Sends an SNS notification if a bucket is public

We need to go Lambda → S3PublicAccessAudit → Code
So lambda_function.py be like as
import boto3

s3 = boto3.client('s3')
sns = boto3.client('sns')

SNS_TOPIC_ARN = "arn:aws:sns:us-east-1:985818273957:S3PublicAccessAlert"

def lambda_handler(event, context):

    buckets = s3.list_buckets()['Buckets']
    public_buckets = []

    for bucket in buckets:
        bucket_name = bucket['Name']

        try:
            public = False

            # Check Block Public Access
            try:
                pab = s3.get_public_access_block(
                    Bucket=bucket_name
                )

                config = pab['PublicAccessBlockConfiguration']

                if not all(config.values()):
                    public = True

            except Exception:
                public = True

            # Check Bucket Policy Status
            try:
                policy_status = s3.get_bucket_policy_status(
                    Bucket=bucket_name
                )

                if policy_status['PolicyStatus']['IsPublic']:
                    public = True

            except Exception:
                pass

            # Check ACL Grants
            try:
                acl = s3.get_bucket_acl(
                    Bucket=bucket_name
                )

                for grant in acl['Grants']:
                    grantee = grant.get('Grantee', {})

                    if 'URI' in grantee and 'AllUsers' in grantee['URI']:
                        public = True

            except Exception:
                pass

            if public:
                public_buckets.append(bucket_name)

        except Exception as e:
            print(f"Error checking {bucket_name}: {e}")

    if public_buckets:

        message = (
            "The following S3 buckets may be publicly accessible:\n\n"
            + "\n".join(public_buckets)
        )

        sns.publish(
            TopicArn=SNS_TOPIC_ARN,
            Subject="S3 Public Access Alert",
            Message=message
        )

    print("Public buckets:", public_buckets)

    return {
        "statusCode": 200,
        "publicBuckets": public_buckets
    }

  **Step 9** : Deploy
  <img width="1896" height="920" alt="image" src="https://github.com/user-attachments/assets/38df2c4f-45bd-4ede-b7e4-28f0cc078f99" />

  **Step 10** Test status got Success
  <img width="1907" height="930" alt="image" src="https://github.com/user-attachments/assets/d7307073-c292-442c-ab3a-cabec2e2fe8e" />

  **Step 11** Bucket available in S3 buckets
  <img width="1878" height="900" alt="image" src="https://github.com/user-attachments/assets/0ffb8ed3-1d7e-4f60-9e5a-b6db1c85f19a" />























  




  
    


















































