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



