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

