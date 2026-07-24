
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
