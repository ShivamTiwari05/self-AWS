**How to configure the AWS cloudfront + Lambda + EventBridge to make sure that it send me notification everyday for the number of instances running**


 **complete Lambda function code** that:

- **Counts running EC2 instances**
- **Sends the result via SMS** using **Amazon SNS**
- Works **whether you use an SNS topic** or **send directly to a phone number**

I'll give you **two versions**:  
- **Version A**: Sends via **SNS topic**
- **Version B**: Sends directly to **a phone number**

---

## ✅ Version A: Using SNS Topic (Recommended for multiple recipients)

```python
import boto3
import datetime
import os

# Create clients
ec2 = boto3.client('ec2')
sns = boto3.client('sns')

def lambda_handler(event, context):
    # Describe EC2 instances in 'running' state
    response = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )

    # Count the number of running instances
    running_count = sum(len(res['Instances']) for res in response['Reservations'])

    # Prepare message
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    message = f"[{timestamp}] There are currently {running_count} running EC2 instances."

    # Publish to SNS Topic
    sns.publish(
        TopicArn=os.environ['SNS_TOPIC_ARN'],
        Subject='Daily EC2 Instance Count Notification',
        Message=message
    )
```

### 👉 What you need to do:
- Set an environment variable in the Lambda:
  - Key: `SNS_TOPIC_ARN`
  - Value: Your SNS topic ARN (like `arn:aws:sns:us-east-1:123456789012:EC2InstanceSMSNotification`)

---

## ✅ Version B: Send SMS directly to a phone number

```python
import boto3
import datetime

# Create clients
ec2 = boto3.client('ec2')
sns = boto3.client('sns')

def lambda_handler(event, context):
    # Describe EC2 instances in 'running' state
    response = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )

    # Count running instances
    running_count = sum(len(res['Instances']) for res in response['Reservations'])

    # Prepare message
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    message = f"[{timestamp}] There are {running_count} EC2 instances running."

    # Send SMS directly
    sns.publish(
        PhoneNumber='+91XXXXXXXXXX',  # Replace with your verified number with country code
        Message=message
    )
```

### 📌 Notes:
- If you're still in **SMS sandbox**, go to SNS → **Text messaging (SMS)** → **Sandbox** → add & verify your number.
- To remove this restriction, request a **production SMS limit increase** from AWS.

---

### 🛡️ IAM Permissions Required for Lambda Role:

Make sure your Lambda's execution role has these permissions:

```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeInstances",
    "sns:Publish"
  ],
  "Resource": "*"
}
```

---

Let me know which version you’re going with, and I can help wire it up with **EventBridge to run at 11 PM daily** or even write the Terraform/IaC if you want!


