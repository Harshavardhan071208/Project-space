# Project-space
🔥 How S3 Connects to DynamoDB:
S3 itself doesn’t directly connect to DynamoDB. The connection happens through an AWS Lambda function triggered by S3 events.

🚀 Flow:
✅ Video is uploaded to S3.

✅ S3 triggers Lambda (via event notification).

✅ Lambda function reads the S3 object details (bucket name, file name, size, etc.).

✅ Lambda writes metadata (file name, user info, upload time, etc.) into DynamoDB.

🛠️ Step-by-Step Setup:
Step 1: Create DynamoDB Table
Go to DynamoDB → Create table.

Example Table:
| Primary Key: | VideoID (String) or FileName (String) |
| Attributes: | Title, UploaderID, UploadDate, S3Bucket, S3Key, etc. |

Step 2: Create an S3 Bucket for Videos
Enable event notifications.

Configure it to trigger a Lambda function on Object Created (PUT) events.

Step 3: Create Lambda Function
This function will be triggered by S3 when a new file is uploaded.

Step 4: Set S3 Event Trigger on Lambda
In the Lambda console → Add trigger → Select S3.

Configure event type: "PUT (Object Created)".

Step 5: Lambda Code Example (Python)
python
Copy
Edit
import json
import boto3
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('YourDynamoDBTableName')

def lambda_handler(event, context):
    # Get bucket name and file key from S3 event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    size = event['Records'][0]['s3']['object']['size']
    
    # Example metadata (can be extended)
    item = {
        'FileName': key,
        'Bucket': bucket,
        'Size': size,
        'UploadDate': datetime.utcnow().isoformat(),
        'Status': 'Uploaded'
    }
    
    # Put item into DynamoDB
    table.put_item(Item=item)
    
    return {
        'statusCode': 200,
        'body': json.dumps('Metadata saved to DynamoDB!')
    }
Step 6: Permissions (IAM Role)
Lambda needs permissions to:
✅ Read from S3: s3:GetObject
✅ Write to DynamoDB: dynamodb:PutItem

Attach a policy like:

json
Copy
Edit
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["dynamodb:PutItem"],
            "Resource": "arn:aws:dynamodb:*:*:table/YourDynamoDBTableName"
        },
        {
            "Effect": "Allow",
            "Action": ["s3:GetObject"],
            "Resource": "arn:aws:s3:::YourS3BucketName/*"
        }
    ]
}
🎯 Summary:
Connection happens via Lambda.

S3 triggers → Lambda → Lambda writes into DynamoDB.

This is the standard, reliable pattern used in serverless architectures.

