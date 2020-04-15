# Learning Objectives
### Open the labreferences.txt File 

* WINDOWS USERS: The Instant Terminal is recommended for this lab. 

## Solution
Log in to the AWS console using the cloud_user credentials provided. Once inside the AWS account, make sure you are using us-east-1 (N. Virginia) as the selected region.

**_Hint: When copying and pasting code into Vim from the lab guide, first enter :set paste (and then i to enter insert mode) to avoid adding unnecessary spaces and hashes._**

### Open the labreferences.txt File

* Navigate to S3.
* Open the bucket containing the text s3bucketlookupfiles.
* Download and open the labreferences.txt file. This will be referenced through other portions of this lab. 

### Create the DEV IAM Role via the AWS CLI Set the AWS CLI Region and Output Type

```bash 
aws configure 
```
Leave the AWS Access Key ID and AWS Secret Access Key blank. Enter us-east-1 as the Default region name and json as the Default output format.

### Create IAM trust Policy for an EC2 Role

* Use a terminal editor (e.g., vi or Nano) to create a file called trust_policy_ec2.json with the following content: 
```bash
{ "Version": "2012-10-17", 
"Statement": [ 
    { "Effect": "Allow", 
    "Principal": {"Service": "ec2.amazonaws.com"}, 
    "Action": "sts:AssumeRole" 
    } 
   ] 
} 
```
### Create the DEV IAM Role 
* Run the following command from aws cli. 
```bash
aws iam create-role --role-name DEV_ROLE --assume-role-policy-document file://trust_policy_ec2.json 
```

### Create the IAM Policyfor the S3 DEV Bucket Read Access via the AWS CLI. 
* Use a terminal editor (e.g., vi or Nano) to create a file called dev_s3_read_access.json with the following content (be sure to replace <DEV_S3_BUCKET_NAME> with the bucket name provided in the labreferences.txt file before saving the file):
```json
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
```


Create the Manged Policy:
```bash
aws iam create-policy --policy-name DevS3ReadAccess --policy-document file://dev_s3_read_access.json
```


### Attach Permissions to the DEV_ROLE via the AWS CLI 

* Run the following AWS CLI command, and be sure to replace 
<POLICY_ARN_FROM_LAST_STEP> with the ARN shown as output from the last command:

```bash
aws iam attach-role-policy --role-name DEV_ROLE --policy-arn "<POLICY_ARN_FROM_LAST_STEP>" 
```

### Verify the settings
* Verify the managed policy was attached: 
```bash
aws iam list-attached-role-policies --role-name DEV_ROLE
```
* Get the policy details, including the current version (be sure to replace <POLICY_ARN_FROM_LAST_STEP> with the policy ARN from earlier):
```bash
aws iam get-policy --policy-arn "<POLICY_ARN_FROM_LAST_STEP>" 
```
* Get the permissions associated with the current policy version (be sure to replace the <POLICY_ARN> and <DEFAULT_VERSION_ID> with the output of the get-policy command):

```bash
aws iam get-policy-version --policy-arn "<POLICY_ARN>" --version-id "<DEFAULT_VERSION_ID>"
```

### Create the DEV Instance Profile and add the DEV_ROLE via the AWS CLI

* Create instance profile:

```bash
aws iam create-instance-profile --instance-profile-name DEV_PROFILE
```
* Add role to the new instance profile:
```bash 
aws iam add-role-to-instance-profile --instance-profile-name DEV_PROFILE --role-name DEV_ROLE
```
* Verify the configuration:
```bash
aws iam get-instance-profile --instance-profile-name DEV_PROFILE
```
### Attach the DEV_PROFILE to an Instance

* Attach the DEV_PROFILE to an EC2 instance (be sure to replace the <LAB_WEB_SERVER_INSTANCE_ID> with the instance ID of the web server in your lab):
```bash
aws ec2 associate-iam-instance-profile --instance-id <LAB_WEB_SERVER_INSTANCE_ID> --iam-instance-profile Name="DEV_PROFILE"
```
* Verify the configuration (be sure to replace the <LAB_WEB_SERVER_INSTANCE_ID> with the instance ID of the web server in your lab):
```bash
aws ec2 describe-instances --instance-ids <LAB_WEB_SERVER_INSTANCE_ID> 
```
Test DEV_ROLE Permissions

* Log In to the Web Server Instance via SSH
    1. Open a new terminal.
    2. Log in to the web server via SSH using the credentials provided. Be sure to use the public IP address for the web server.

Test the Configuration

* Run the following command to determine the identity currently used in the order:
```bash
aws sts get-caller-identity
```

* Verify access to the <DEV_S3_BUCKET_NAME> in your lab (be sure to replace the <DEV_S3_BUCKET_NAME> with the value provided in the labreferences.txt file):
```bash
aws s3 ls
aws s3 ls s3://<DEV_S3_BUCKET_NAME>
```

* Verify access is denied to the <SECRET_S3_BUCKET_NAME> (be sure to replace the <SECRET_S3_BUCKET_NAME> with the value provided in the labreferences.txt file):
```bash 
aws s3 ls s3://<SECRET_S3_BUCKET_NAME>
```