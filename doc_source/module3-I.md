# Module 3 \(Part 1\): Lambda Functions on AWS IoT Greengrass<a name="module3-I"></a>

This module shows you how to create and deploy a Lambda function that sends MQTT messages from your AWS IoT Greengrass core device\. The module describes Lambda function configurations, subscriptions used to allow MQTT messaging, and deployments to a core device\.

[Module 3 \(Part 2\)](module3-II.md) covers the differences between on\-demand and long\-lived Lambda functions running on the AWS IoT Greengrass core\.

Before you begin, make sure that you have completed [Module 1](module1.md) and [Module 2](module2.md) and have a running AWS IoT Greengrass core device\.

**Tip**  
Or, to use a script that sets up your core device for you, see [Quick Start: Greengrass Device Setup](quick-start.md)\. The script can also create and deploy the Lambda function used in this module\.

This module should take about 30 minutes to complete\.

**Topics**
+ [Create and Package a Lambda Function](create-lambda.md)
+ [Configure the Lambda Function for AWS IoT Greengrass](config-lambda.md)
+ [Deploy Cloud Configurations to an AWS IoT Greengrass Core Device](configs-core.md)
+ [Verify the Lambda Function Is Running on the Core Device](lambda-check.md)