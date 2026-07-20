# ks-aws-lambda-devops-assignments
**AWS Cloud deployment**

**Question 1. Automated S3 Bucket Cleanup (Objects Older Than 30 Days)**

Step 1: As recommended, region selected after login in AWS account is **us-east-1**
<img width="1890" height="287" alt="image" src="https://github.com/user-attachments/assets/5572688e-1d52-4291-8ebd-b8155a88a2f7" />

Step 2: Created budget alert for $1.
<img width="1731" height="901" alt="image" src="https://github.com/user-attachments/assets/0a54bdb0-5a0b-419c-a296-ddc4b32581ae" />
<img width="1540" height="136" alt="image" src="https://github.com/user-attachments/assets/723a0d16-3bc8-4f4e-b610-c8fadf237f57" />

Step 3: Created new repository **"saurabhkumar2k/ks-aws-lambda-devops-assignments.git"**. Clone the same in my local.
Step 4: Ceated question wise folders.
Step 5: Created S3 bucket
<img width="1877" height="803" alt="image" src="https://github.com/user-attachments/assets/5b42e6bd-7839-463e-baea-00dd920c00f5" />
Step 6: Upload few text files.
<img width="1876" height="905" alt="image" src="https://github.com/user-attachments/assets/5374d00b-3136-4441-96c8-619d719fdf70" />
<img width="1883" height="902" alt="image" src="https://github.com/user-attachments/assets/b7899bd2-6393-464b-93f5-1a5814a78e8a" />
Step 7: Created IAM role having inline policy.
<img width="1867" height="907" alt="image" src="https://github.com/user-attachments/assets/68a2b8c6-4def-4f3d-b18e-fbfb52a1d5fb" />
<img width="1893" height="916" alt="image" src="https://github.com/user-attachments/assets/bcc6056e-564e-4eed-8f23-14d875e95d24" />
<img width="1878" height="908" alt="image" src="https://github.com/user-attachments/assets/21b26e66-1ebf-4cc2-9b49-c094637c3559" />
Step 8: Created Lambda function
<img width="1882" height="912" alt="image" src="https://github.com/user-attachments/assets/212a6896-5374-428f-bac0-5feb6c5c6912" />
<img width="1888" height="922" alt="image" src="https://github.com/user-attachments/assets/f176f2d8-4bb4-40fd-823d-355c570f9a46" />
<img width="1881" height="912" alt="image" src="https://github.com/user-attachments/assets/95ad1992-e058-4657-8270-9ceef35e26e6" />

<img width="1893" height="927" alt="image" src="https://github.com/user-attachments/assets/593c5a70-f040-4c0a-82da-035b04a10414" />


I tested my bucket for its emptyness after 5 minutes because I have given age_minutes=5 instead of age_days=30.
<img width="1912" height="913" alt="image" src="https://github.com/user-attachments/assets/1ebdf93e-323e-492e-a3a2-deb76ccd6c2c" />











