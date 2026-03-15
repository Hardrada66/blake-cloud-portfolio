Automated Cloud TThreat Detection & Response (AWS)
Overview

This project demonstrates a cloud-native security automation pipeline built in AWS. The goal was to simulate automated incident response by detecting security findings and triggering infrastructure-level containment actions.

The architecture integrates several AWS security and automation services:

Amazon GuardDuty – threat detection

Amazon EventBridge – event routing

AWS Lambda – automated response logic

Amazon SNS – security alert notifications

EC2 Security Groups – network-level isolation

When a GuardDuty finding is generated, the system automatically triggers a response workflow designed to contain potentially compromised resources and notify security operators.

Architecture
GuardDuty Finding
        ↓
EventBridge Rule
        ↓
Lambda Function (Automated Response)
        ↓
EC2 Isolation + Instance Tagging
        ↓
SNS Email Notification
Project Goals

This project was designed to demonstrate:

• Cloud-native security detection pipelines
• Event-driven security automation
• Automated incident containment in AWS
• Integration of multiple AWS services for security operations

Infrastructure Components
1. Amazon GuardDuty

GuardDuty is AWS’s managed threat detection service.

It analyzes multiple data sources including:

CloudTrail logs

VPC Flow Logs

DNS query logs

These detections generate security findings which represent suspicious or malicious activity.

Example detections:

Credential compromise

Port scanning

Command-and-control traffic

Cryptocurrency mining

2. Amazon EventBridge

EventBridge acts as the event routing layer.

A rule was created to match:

source: aws.guardduty
detail-type: GuardDuty Finding

When a GuardDuty finding occurs, EventBridge triggers the Lambda function.

3. AWS Lambda – Automated Response

The Lambda function performs automated remediation actions when triggered.

Actions performed:

Extract the EC2 instance ID from the GuardDuty finding

Tag the instance as compromised

Replace its security group with a quarantine / isolation security group

Send a notification via SNS

This simulates automated containment of a compromised workload.

Example Lambda Logic
import boto3
import json

ec2 = boto3.client("ec2")
sns = boto3.client("sns")

def lambda_handler(event, context):

    instance_id = event.get("detail", {}) \
        .get("resource", {}) \
        .get("instanceDetails", {}) \
        .get("instanceId")

    if not instance_id:
        return

    # isolate instance
    ec2.modify_instance_attribute(
        InstanceId=instance_id,
        Groups=["ISOLATION_SECURITY_GROUP"]
    )

    # tag instance
    ec2.create_tags(
        Resources=[instance_id],
        Tags=[
            {"Key": "Compromised", "Value": "True"},
            {"Key": "Response", "Value": "GuardDutyAutoIsolation"}
        ]
    )

    # send alert
    sns.publish(
        TopicArn="SNS_TOPIC_ARN",
        Subject="GuardDuty Automated Response Triggered",
        Message=json.dumps(event)
    )
Security Controls Implemented
Automated Isolation

A dedicated isolation security group was created with:

No inbound rules

No outbound rules

Applying this security group immediately quarantines the instance from network communication.

Instance Tagging

Instances affected by security findings are tagged with:

Compromised = True
Response = GuardDutyAutoIsolation

This allows security teams to quickly identify compromised resources.

Alerting

Amazon SNS sends security notifications whenever automated containment occurs.

Alerts include:

Finding ID

Severity

Resource impacted

Response action taken

Testing the System

GuardDuty sample findings were generated to simulate malicious activity.

This triggered:

GuardDuty detection

EventBridge rule execution

Lambda automated response

SNS alert delivery

Lessons Learned

During testing, GuardDuty sample findings generated a high volume of events. Because Lambda executions were failing during early tests, AWS’s automatic retry behavior caused the same events to be processed multiple times.

This resulted in:

Repeated Lambda invocations

SNS notification flooding

This highlighted the importance of production safeguards such as:

Event filtering

Dead-letter queues

Retry limits

Idempotent Lambda logic

Future Improvements

To make this architecture production-ready, the following improvements are recommended:

• Implement EventBridge filtering to limit event scope
• Add Dead Letter Queues (DLQ) for Lambda failures
• Introduce retry limits and error handling
• Store original security groups for forensic rollback
• Implement security orchestration workflows

Skills Demonstrated

This project demonstrates hands-on experience with:

Cloud security detection

AWS event-driven architecture

Security automation

Infrastructure containment strategies

Lambda-based remediation

Multi-service AWS security integration
