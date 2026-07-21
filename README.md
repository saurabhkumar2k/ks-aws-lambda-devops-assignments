# ks-aws-lambda-devops-assignments
**AWS Cloud deployment**

**Task 1. Automated S3 Bucket Cleanup (Objects Older Than 30 Days)**

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

**Task 2:** **Automated EBS Snapshot Creation and Cleanup**
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

<img width="1888" height="815" alt="image" src="https://github.com/user-attachments/assets/2eab02a5-8f9f-4844-b72c-98d6986dd50d" />



<img width="1543" height="757" alt="image" src="https://github.com/user-attachments/assets/e4d23343-e822-4031-95fd-2fdfdec0be44" />

<img width="1896" height="875" alt="image" src="https://github.com/user-attachments/assets/06ea5d9c-e3d4-4364-b002-0a2801aa6558" />

<img width="1885" height="923" alt="image" src="https://github.com/user-attachments/assets/2d2d5d58-fde1-4e8d-9e23-ce557691c9a4" />

<img width="1890" height="848" alt="image" src="https://github.com/user-attachments/assets/0eaffc29-fc6e-46a2-8f97-367abe04685f" />

<img width="1890" height="823" alt="image" src="https://github.com/user-attachments/assets/32b219cf-a84d-42b4-8b8e-6b20d8f92534" />

<img width="1910" height="767" alt="image" src="https://github.com/user-attachments/assets/5940e2ad-be5f-4fbc-827d-4fe11f5dc42d" />

<img width="1906" height="773" alt="image" src="https://github.com/user-attachments/assets/d75350d4-f948-458e-9301-5551b05b850a" />

























