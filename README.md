# CloudWatch Log Exporter

## Introduction
CloudWatch Log Exporter is a serverless application designed to automate the process of exporting CloudWatch log groups to S3 buckets. It uses AWS Step Functions to orchestrate the export process, making it scalable and resilient.

The application performs the following key functions:
* Lists CloudWatch log groups across specified AWS regions
* Creates export tasks for each log group
* Monitors the status of export tasks
* Updates a DynamoDB table with the export status
* Notifies about failed exports via SNS

## Architecture Diagram

```

+----------------+     +-------------------+     +------------------+
|  EventBridge   |---->| Step Functions    |---->|  Lambda Function |
|    (Trigger)   |     | State Machine     |     |                  |
+----------------+     +-------------------+     +------------------+
                                 |                   |    |    |
                                 |                   |    |    |
                                 v                   v    v    v
                      +------------------+    +-----+----+----+-----+
                      |    DynamoDB      |    | CloudWatch | S3     |
                      |    (Task Status) |    |    Logs    |        |
                      +------------------+    +------------+--------+
                                                     |
                                                     v
                                              +-----------+
                                              |    SNS    |
                                              | (Failures)|
                                              +-----------+
```


## Installation

### Prerequisites
* AWS CLI configured with appropriate permissions
* Node.js and npm installed
* AWS CDK CLI installed (npm install -g aws-cdk)

### Steps
1. Clone the repository:

```
git clone https://github.com/zhang1980s/cloudwatch-log-exporter.git
cd cloudwatch-log-exporter
```


2. Install dependencies:

```
npm install
```

3. Deploy the CDK stack:

```
./cdk-deploy-to.sh <account id> <region>
```

4. During deployment, you'll be prompted to enter values for the following parameters:
* logPrefix: The prefix for exported logs in the S3 bucket
* ScheduleParameter: The cron schedule for running the export process
* ExportDaysParameter: The number of days of logs to export


5. After deployment, the CDK will output the ARN of the Step Functions state machine. You can use this ARN to manually trigger the export process if needed.


## Configuration
The application uses an SSM Parameter to store the mapping of regions to S3 buckets. You can modify this mapping by updating the SSM Parameter named /cloudwatch-log-exporter/region-bucket-map.
The format of the parameter value should be:
```
region1,s3://bucket1
region2,s3://bucket2
```

## Monitoring and Troubleshooting
* Check the CloudWatch Logs for the Lambda function to see detailed logs of the export process.
* The DynamoDB table stores the status of each export task. You can query this table to see the current state of exports.
* Failed exports will trigger an SNS notification. Make sure to subscribe to the SNS topic to receive these notifications.

