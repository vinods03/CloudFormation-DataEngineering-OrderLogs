# CloudFormation-DataEngineering-OrderLogs
An end-to-end Data Engineering project on AWS along with the CloudFormation stack to provision all the required resources


What is this project about ?

This is an extension of the project in https://github.com/vinods03/AWS-DataEngineering-OrderLogs
Order logs data that our system receives on an EC2 instance, is processed near real-time into Redshift cluster.
The major enhancement here is the automation of the entire resource provisiong through a CloudFormation Stack.

A minor enhancement is the addition of an SQS queue as the notification target for the S3 landing area.
The lambda function that processes the data from S3 landing area to S3 staging area will now read from the SQS queue instead of reading directly from the S3 bucket.
In case, lambda throttles or for some reason unable to process a huge number of files coming in to the S3 landing area, having a SQS queue will ensure notifications are not lost.

How to use the stack ?

Please go through the "CloudFormation Stack details.docx" and create the resources that are not part of the stack (ex: Redshift cluster, DynamoDB table etc).
Make sure you change the lambda script and glue script as per your S3 Staging area bucket name, Glue crawler name and Secrets manager name.
Make sure you copy order_logs_lambda_processor.zip and order_logs_glue_script.py into an appropriate S3 bucket and reference these paths in the CloudFormation Stack, instead of my code repository. 
Now create the stack using cfn-order-logs.yml.


Once the stack is created, log on to the EC2 instance and execute the below steps to generate data that needs to be processed through the various layers into the Redshift cluster.

cd /etc/aws-kinesis
sudo nano agent.json -> Give the source path - the path where logs will be sent / generated and the target firehose name. 
Update the firehose.endpoint if region is other than us-east-
The agent.json should look like this:

{
  "cloudwatch.emitMetrics": true,
  "kinesis.endpoint": "",
  "firehose.endpoint": "firehose.us-east-1.amazonaws.com",

  "flows": [
    {
      "filePattern": "/var/log/cadabra/*",
      "deliveryStream": "order-logs-firehose"
    }
  ]
}

Restart the kinesis agent using below command:
sudo service aws-kinesis-agent stop
sudo service aws-kinesis-agent start

Then generate the logs using the command after cd ~ (to go to home directory): 
sudo ./LogGenerator.py 250000 (note: 250000 represents the number of orders)

You should be able to see the data flowing through the S3 landing area, S3 staging area, Redshift cluster and the DynamoDB audit table.
You should also be able to query the landing area data in Athena as per our design.
