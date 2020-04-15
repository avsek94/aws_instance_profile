Learning Objectives
Open the labreferences.txt File
WINDOWS USERS: The Instant Terminal is recommended for this lab.
Solution

Log in to the AWS console using the cloud_user credentials provided. Once inside the AWS account, make sure you are using us-east-1 (N. Virginia) as the selected region.

Hint: When copying and pasting code into Vim from the lab guide, first enter :set paste (and then i to enter insert mode) to avoid adding unnecessary spaces and hashes.

Open the labreferences.txt File

Navigate to S3.
Open the bucket containing the text s3bucketlookupfiles.
Download and open the labreferences.txt file. This will be referenced through other portions of this lab.
Create the DEV IAM Role via the AWS CLI
Set the AWS CLI Region and Output Type

aws configure
Leave the AWS Access Key ID and AWS Secret Access Key blank. Enter us-east-1 as the Default region name and json as the Default output format.

Create IAM trust Policy for an EC2 Role

Use a terminal editor (e.g., vi or Nano) to create a file called trust_policy_ec2.json with the following content:
    {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }
     ]
    }
Create the DEV IAM Role by running the following command from aws cli.
aws iam create-role --role-name DEV_ROLE --assume-role-policy-document file://trust_policy_ec2.json
Create the IAM Policyfor the S3 DEV Bucket Read Access via the AWS CLI.
Use a terminal editor (e.g., vi or Nano) to create a file called dev_s3_read_access.json with the following content (be sure to replace <DEV_S3_BUCKET_NAME> with the bucket name provided in the labreferences.txt file before saving the file):

{
"Version": "2012-10-17",
"Statement": [
    {
      "Sid": "AllowUserToSeeBucketListInTheConsole",
      "Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::*"]
    },
    {
        "Effect": "Allow",
        "Action": [
            "s3:Get*",
            "s3:List*"
        ],
        "Resource": [
            "arn:aws:s3:::<DEV_S3_BUCKET_NAME>/*",
            "arn:aws:s3:::<DEV_S3_BUCKET_NAME>"
        ]
    }
    ]
}

Create the Manged Policy:

aws iam create-policy --policy-name DevS3ReadAccess --policy-document file://dev_s3_read_access.json
Attach Permissions to the DEV_ROLE via the AWS CLI
Run the following AWS CLI command, and be sure to replace <POLICY_ARN_FROM_LAST_STEP> with the ARN shown as output from the last command:

aws iam attach-role-policy --role-name DEV_ROLE --policy-arn "<POLICY_ARN_FROM_LAST_STEP>"
Verify the settings

Verify the managed policy was attached:
aws iam list-attached-role-policies --role-name DEV_ROLE
