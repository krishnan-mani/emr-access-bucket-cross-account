 Illustrates access to S3 bucket owned by a different account from instances in an EMR cluster 

HOW-TO
===

- We assume that the bucket is created and owned on account X and the EMR cluster is provisioned in account Y
- The process of creating the bucket policy and IAM roles is illustrated using AWS CLI actions
- *NOTE*: Replace the values of ```<ACCOUNT-X>``` and ```<BUCKET-NAME>``` in respective policy document examples before use
- Create the bucket policy on the bucket in account X 

```
# Using an AWS CLI profile with credentials for account X (say, "admin-X")
# Assume the bucket name is "foobar"
$ cd policies
policies $ aws --profile admin-X s3api put-bucket-policy \
    --bucket foobar \
    --policy file://bucket-policy.json

# Copy a test file into the bucket
policies $ aws s3 cp foo.txt s3://foobar/

```

- Create the IAM instance profile and role in account Y

```
# Using an AWS CLI profile with credentials for account Y (say, "admin-Y")

# Create instance profile
policies $ aws --profile admin-Y iam create-instance-profile \
    --instance-profile-name cross-account-bucket-access

# Create role
policies $ aws --profile admin-Y iam create-role \
    --role-name cross-account-bucket-access \
    --assume-role-policy-document file://role-trust-policy.json

# Add inline policy to role 
policies $ aws --profile admin-Y iam put-role-policy \
    --role-name cross-account-bucket-access \
    --policy-name access-bucket \
    --policy-document file://role-permissions-policy.json

# Adds IAM role to instance profile
policies $ aws --profile admin-Y iam add-role-to-instance-profile \
    --instance-profile-name cross-account-bucket-access \
    --role-name cross-account-bucket-access

```

- Use a CloudFormation template to launch a stack for the EMR cluster

```
policies $ cd ../stacks/emr

# Specify values for parameters SubnetId, KeyName, and InstanceProfileName ("cross-account-bucket-access")
emr $ aws --profile admin-Y cloudformation create-stack \
    --stack-name cross-account-bucket-access-emr \
    --template-body file://template.yaml \
    --parameters \
    ParameterKey=SubnetId,ParameterValue=subnet-NULL0011 \
    ParameterKey=KeyName,ParameterValue=MyKey \
    --capabilities CAPABILITY_IAM

```

- You may verify the bucket policy, by logging on to an instance in the EMR cluster and copying objects from the bucket (use the AWS CLI or any AWS SDK: these will automatically retrieve temporary credentials from STS by virtue of the IAM instance profile associated with the instance)

References
===

- [Bucket Owner Granting Cross-Account Bucket Permissions](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-walkthroughs-managing-access-example2.html)
- [Configure IAM Roles for Amazon EMR Permissions to AWS Services](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-iam-roles.html)
