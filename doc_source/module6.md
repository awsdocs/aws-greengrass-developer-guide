--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

--------

# Module 6: Accessing other AWS services<a name="module6"></a>

This advanced module shows you how AWS IoT Greengrass cores can interact with other AWS services in the cloud\. It builds on the traffic light example from [Module 5](module5.md) and adds a Lambda function that processes shadow states and uploads a summary to an Amazon DynamoDB table\.

![\[AWS IoT connected to an AWS IoT Greengrass core, which is connected to a light switch device and a traffic light device shadow. The traffic light device shadow is connected to a Lambda function, which is connected to a DynamoDB table.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-089.5.png)

Before you begin, run the [Greengrass device setup](quick-start.md) script, or make sure that you have completed [Module 1](module1.md) and [Module 2](module2.md)\. You should also complete [Module 5](module5.md)\. You do not need other components or devices\.

This module should take about 30 minutes to complete\.

**Note**  
This module creates and updates a table in DynamoDB\. Although most of the operations are small and fall within the Amazon Web Services Free Tier, performing some of the steps in this module might result in charges to your account\. For information about pricing, see [DynamoDB pricing documentation](https://aws.amazon.com/dynamodb/pricing/)\.

**Topics**
+ [Configure the group role](config-iam-roles.md)
+ [Create and configure the Lambda function](create-config-lambda.md)
+ [Configure subscriptions](config_subs.md)
+ [Test communications](comms-test.md)