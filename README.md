#🚀 Event-Driven S3 Compliance Automation
📌 Project Overview

This project implements an event-driven automation framework that enforces security and compliance policies on Amazon S3 buckets in real time.

Whenever a new S3 bucket is created, the system automatically:

Blocks public access
Enables versioning
Adds compliance tags

🏗️ Architecture Flow
S3 Bucket Creation
        ↓
CloudTrail
        ↓
EventBridge Rule
        ↓
Lambda Function
        ↓
Apply Compliance (Security Enforcement)

⚙️ Step-by-Step Implementation

🔹 Step 1: Create CloudTrail
Go to AWS Console → CloudTrail
Click Create Trail

Configuration:

Trail Name: S3-compliance-trail
Storage: Create new S3 bucket
Encryption: Disabled
Log File Validation: Enabled

Events:

Select Management Events
Enable Read

Click → Create Trail

🔹 Step 2: Create IAM Role for Lambda
Go to IAM → Roles → Create Role
Select Lambda

Attach Policies:

AmazonS3FullAccess
CloudWatchLogsFullAccess

Role Name:
S3ComplianceLambdaRole

🔹 Step 3: Create Lambda Function
Go to Lambda → Create Function

Configuration:

Name: S3-compliance-enforcer
Runtime: Python 3.11
Architecture: x86_64

Execution Role:

Use existing role → S3ComplianceLambdaRole
🔹 Lambda Code
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):

    print("Event received:", json.dumps(event))

    bucket_name = event['detail']['requestParameters']['bucketName']

    try:
        # Block public access
        s3.put_public_access_block(
            Bucket=bucket_name,
            PublicAccessBlockConfiguration={
                'BlockPublicAcls': True,
                'IgnorePublicAcls': True,
                'BlockPublicPolicy': True,
                'RestrictPublicBuckets': True
            }
        )

        # Enable versioning
        s3.put_bucket_versioning(
            Bucket=bucket_name,
            VersioningConfiguration={
                'Status': 'Enabled'
            }
        )

        # Add compliance tag
        s3.put_bucket_tagging(
            Bucket=bucket_name,
            Tagging={
                'TagSet': [
                    {'Key': 'Compliance', 'Value': 'Enforced'},
                    {'Key': 'ManagedBy', 'Value': 'Lambda'}
                ]
            }
        )

        print(f"Compliance enforced for bucket: {bucket_name}")

    except Exception as e:
        print(f"Error processing bucket {bucket_name}: {str(e)}")

    return {
        'statusCode': 200,
        'body': json.dumps('S3 compliance check completed')
    }
🔹 Step 4: Create EventBridge Rule
Go to EventBridge → Create Rule

Rule Name:
S3-compliance-lambda-rule

🔹 Event Pattern (Important)
{
  "source": ["aws.s3"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["s3.amazonaws.com"],
    "eventName": ["CreateBucket"]
  }
}

Explanation:

Detects S3 bucket creation events
Triggered via CloudTrail logs
Invokes Lambda automatically
🔹 Add Target
Select Lambda Function
Choose: S3-compliance-enforcer

🧪 Step 5: Test the Automation
1. Create a New S3 Bucket
Disable:
Block Public Access
Versioning
2. After Creation, Verify:

✅ Versioning → Automatically Enabled
✅ Tags → Automatically Added
✅ Public Access → Automatically Blocked

🔐 Final Result

Whenever an S3 bucket is created:

Public access is blocked automatically
Versioning is enabled automatically
Compliance tags are added automatically

This ensures security, governance, and compliance enforcement across all S3 buckets.
