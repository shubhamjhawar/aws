# STEPS

## Question 1

Create S3 bucket from AWS CLI <br>
<b>a.</b> Create an IAM role with S3 full access

```
aws iam create-role --role-name BucketMaker  --assume-role-policy-document file:///trustpolicy.json

```

Here the trust policy json contains the following information

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

```

Providing the role with S3 full access

```
aws iam attach-role-policy --role-name  BucketMaker  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

```

<b>b.</b>Create an EC2 instance with above role
Creating an instance profile

```python3

import boto3

# Specify the region where you want to launch the instance
region = 'us-east-1'

# Specify the AMI ID of the Amazon Linux 2 AMI
ami_id = 'ami-0889a44b331db0194'

# Specify the instance type
instance_type = 't2.micro'

# Specify the security group IDs
security_group_ids = ['sg-066205334fd658af6']

# Specify the IAM role ARN
iam_role_arn = 'arn:aws:iam::574344495913:instance-profile/ProfileShubham'

# Create a client to interact with EC2 service
ec2_client = boto3.client('ec2', region_name=region)

# Launch the instance with the IAM role specified
response = ec2_client.run_instances(
    ImageId=ami_id,
    InstanceType=instance_type,
    SecurityGroupIds=security_group_ids,
    IamInstanceProfile={
        'Arn': iam_role_arn
    },
    MinCount=1,
    MaxCount=1
)

# Get the instance ID of the launched instance
instance_id = response['Instances'][0]['InstanceId']

# Print the instance ID
print('Launched instance with ID:', instance_id)


```
<img width="385" alt="Screenshot 2023-05-10 at 8 59 52 AM" src="https://github.com/shubhamjhawar/Snowflake/assets/66582610/f163e4ca-c008-4d18-810a-387d15d4b7ac">

<br>


<img width="1188" alt="Screenshot 2023-05-10 at 9 00 41 AM" src="https://github.com/shubhamjhawar/Snowflake/assets/66582610/ce5b1a5d-a643-4a9d-ad8c-d0ab9f5cf941">



<b>c.</b>
Creating the bucket

```
 aws s3api create-bucket --bucket bucketfromcli03  --region us-east-1

```


## Question 2

Put files in S3 bucket from lambda <br>

Creating clients

```
import boto3
import json
from botocore.exceptions import ClientError

```

<b>a.</b>Create custom role for AWS lambda which will only have put object access
Creating policy for put object access

```

iam = boto3.client('iam')

#  Put files in S3 bucket from lambda
#  a. Create custom role for AWS lambda which will have only put object access

policy_document = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::myawsbucketshubham/*"
            ]
        }
    ]
}

create_policy_response = iam.create_policy(
        PolicyName='put-object-policy',
        PolicyDocument=json.dumps(policy_document)
)

policy_arn = create_policy_response['Policy']['Arn']

role_name = 'put-object-lambda-role'
create_role_response = iam.create_role(
    RoleName=role_name,
    AssumeRolePolicyDocument=json.dumps({
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "Service": "lambda.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
            }
        ]
    })
)

create_role_response = iam.attach_role_policy(
    RoleName=role_name,
    PolicyArn=policy_arn
)

```

Attach the policy to the role

```
create_role_response = iam.attach_role_policy(
    RoleName=role_name,
    PolicyArn=policy_arn
)

```

<b>b.</b>Add role to generate and access Cloudwatch logs
Creating cloudwatch policy

```
cloudwatch_logs_policy_document = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "logs:GetLogEvents"
            ],
            "Resource": "*"
        }
    ]
}

cloudwatch_logs_policy_response = iam.create_policy(
    PolicyName='CloudWatchLogsPolicy',
    PolicyDocument=json.dumps(cloudwatch_logs_policy_document)
)

cloudwatch_logs_policy_arn = cloudwatch_logs_policy_response['Policy']['Arn']
```

Attach the policy to the role

```
iam.attach_role_policy(
    RoleName=role_name ,
    PolicyArn=cloudwatch_logs_policy_arn
)
```

<img width="1123" alt="Screenshot 2023-05-10 at 9 03 23 AM" src="https://github.com/shubhamjhawar/Snowflake/assets/66582610/d3638e3f-b3bf-4b13-b62b-ad495fbf23db">



<b></b>In python script, generate json in given format and save .json file in bucket created<br>
<b></b>Created a lambda_handler function to save the file in json in format and upload it to the bucket<br>
<b></b>Schedule the job to run every minute. Stop execution after 3 runs
<b></b>Using amazon lambda, created a function and uploaded the zip file created above

<img width="1277" alt="Screenshot 2023-05-10 at 9 12 04 AM" src="https://github.com/shubhamjhawar/Snowflake/assets/66582610/dbd16a03-dad2-4837-86ed-878be89dcbf1">

<img width="1334" alt="Screenshot 2023-05-10 at 9 08 39 AM" src="https://github.com/shubhamjhawar/Snowflake/assets/66582610/68c01d7a-8809-4e18-acf9-b1c3b3cf2d8c">

<br>
<br>
<br>
The log Files are below


<img width="1128" alt="Screenshot 2023-05-10 at 9 11 39 AM" src="https://github.com/shubhamjhawar/Snowflake/assets/66582610/933e5137-0824-435b-bf01-ae52141e9b91">


## Question 3

<b>a.</b> Modify lambda function to accept parameters and return file name
Modified lambda function is :
<b>b.</b>Create a POST API from API Gateway, pass parameters as request body to Lambda
    job. Return filename and status code as response.
<b>c.</b>Consume API from local machine and pass unique data to lambda.
<b>d.</b>Check if cloud watch logs are generated

<br>

To create a post API to feed to lambda job these steps were followed 
```
Go to the API Gateway console and click "Create API".
Select "REST API" and click "Build".
Choose "New API" and enter a name for your API. Click "Create API".
Click "Create Resource" to create a new resource under your API.
Enter a name for your resource and click "Create Resource".
Click "Create Method" and select "POST" from the dropdown.
Select "Lambda Function" and check the "Use Lambda Proxy integration" box.
Enter the name of your Lambda function in the "Lambda Function" field and click "Save".
Deploy your API by clicking "Actions" > "Deploy API". Select "New Stage" and enter a name for your stage. Click "Deploy".
Note the URL of your API endpoint

```


<img width="726" alt="Screenshot 2023-05-10 at 9 17 41 AM" src="https://github.com/shubhamjhawar/Snowflake/assets/66582610/d307364c-4a00-4485-846d-46bfe6dd91de">

<br>

For the sending the file using local machine i used curl and did 

```
curl -X POST -H "Content-Type: application/json" -d '{"transaction_id": 12345, "payment_mode": "card", "amount": 200.0, "customer_id": 101}'  https://axtk2nk25e.execute-api.us-east-1.amazonaws.com/prod/aws-assignment



```

<br>


<img width="1368" alt="Screenshot 2023-05-10 at 9 26 57 AM" src="https://github.com/shubhamjhawar/python_algorithms/assets/66582610/55d0c506-ec0a-4297-918f-b696c4a5f295">



