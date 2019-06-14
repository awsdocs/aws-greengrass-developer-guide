# Module 6: Accessing Other AWS Services<a name="module6"></a>

This advanced module shows you how AWS IoT Greengrass cores can interact with other AWS services in the cloud\. It builds on the traffic light example from [Module 5](module5.md) and adds a Lambda function that processes shadow states and uploads a summary to an Amazon DynamoDB table\.

![\[AWS IoT connected to an AWS IoT Greengrass core, which is connected to a light switch device and a traffic light device shadow. The traffic light device shadow is connected to a Lambda function, which is connected to a DynamoDB table.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-089.5.png)

Before you begin, make sure that you have completed [Module 1](module1.md) through [Module 5](module5.md)\. You do not need other components or devices\.

This module should take approximately 30 minutes to complete\.

**Note**  
This module creates and updates a table in DynamoDB\. Although most of the operations are small and fall within the AWS Free Tier, performing some of the steps in this module might result in charges to your account\. For information about pricing, see [DynamoDB pricing documentation](https://aws.amazon.com/dynamodb/pricing/)\.

**Topics**
+ [Configure IAM Roles](config-iam-roles.md)
+ [Create and Configure the Lambda Function](create-config-lambda.md)
+ [Configure Subscriptions](config_subs.md)
+ [Test Communications](comms-test.md)