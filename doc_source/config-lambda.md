--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Configure the Lambda function for AWS IoT Greengrass<a name="config-lambda"></a>

You are now ready to configure your Lambda function for AWS IoT Greengrass\.

In this step, you:
+ Use the AWS IoT console to add the Lambda function to your Greengrass group\.
+ Configure group\-specific settings for the Lambda function\.
+ Add a subscription to the group that allows the Lambda function to publish MQTT messages to AWS IoT\.
+ Configure local log settings for the group\.

 

1. <a name="console-gg-groups"></a>In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

1. Under **Greengrass groups**, choose the group that you created in [Module 2](module2.md)\.

1. On the group configuration page, choose the **Lambda functions** tab, and then scroll down to the **My Lambda functions** section and choose **Add Lambda function**\.

1. Select the name of the Lambda function you created in the previous step \(**Greengrass\_HelloWorld**, not the alias name\)\.

1. For the version, choose **Alias: GG\_HelloWorld**\.

1. In the **Lambda function configuration** section, make the following changes:
   + Set the **System user and group** to **Use group default**\.
   + Set the **Lambda function containerization** to **Use group default**\.
   + Set **Timeout** to 25 seconds\. This Lambda function sleeps for 5 seconds before each invocation\.
   + For **Pinned**, choose **True**\.

    
**Note**  
<a name="long-lived-lambda"></a>A *long\-lived* \(or *pinned*\) Lambda function starts automatically after AWS IoT Greengrass starts and keeps running in its own container\. This is in contrast to an *on\-demand* Lambda function, which starts when invoked and stops when there are no tasks left to run\. For more information, see [Lifecycle configuration for Greengrass Lambda functions](lambda-functions.md#lambda-lifecycle)\.

1. Choose **Add Lambda function** to save your changes\. For information about Lambda function properties, see [Controlling execution of Greengrass Lambda functions by using group\-specific configuration](lambda-group-config.md)\.

   Next, create a subscription that allows the Lambda function to send [MQTT](http://mqtt.org/) messages to AWS IoT Core\.

   A Greengrass Lambda function can exchange MQTT messages with:
   + [Devices](what-is-gg.md#greengrass-devices) in the Greengrass group\.
   + [Connectors](connectors.md) in the group\.
   + Other Lambda functions in the group\.
   + AWS IoT Core\.
   + The local shadow service\. For more information, see [Module 5: Interacting with device shadows](module5.md)\.

   The group uses subscriptions to control how these entities can communicate with each other\. Subscriptions provide predictable interactions and a layer of security\.

   A subscription consists of a source, target, and topic\. The source is the originator of the message\. The target is the destination of the message\. The topic allows you to filter the data that is sent from the source to the target\. The source or target can be a Greengrass device, Lambda function, connector, device shadow, or AWS IoT Core\.
**Note**  
A subscription is directed in the sense that messages flow in a specific direction: from the source to the target\. To allow two\-way communication, you must set up two subscriptions\.
**Note**  
 Currently, the subscription topic filter does not allow more than a single `+` character in a topic\. The topic filter only allows a single `#` character at the end of a topic\. 

   The `Greengrass_HelloWorld` Lambda function sends messages only to the `hello/world` topic in AWS IoT Core, so you only need to create one subscription from the Lambda function to AWS IoT Core\. You create this in the next step\.

1. On the group configuration page, choose the **Subscriptions** tab, and then choose **Add subscription**\.

   For an example that shows you how to create a subscription using the AWS CLI, see [create\-subscription\-definition](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/greengrass/create-subscription-definition.html) in the *AWS CLI Command Reference*\.

1. In the **Source type**, choose **Lambda function** and, for the **Source**, choose **Greengrass\_HelloWorld**\.

1. For the **Target type**, choose **Service** and, for the **Target** select **IoT Cloud**\.

1. For **Topic filter**, enter **hello/world**, and then choose **Create subscription**\.

1. Configure the group's logging settings\. For this tutorial, you configure AWS IoT Greengrass system components and user\-defined Lambda functions to write logs to the file system of the core device\.

   1. On the group configuration page, choose the **Logs** tab\.

   1. In the **Local logs configuration** section, choose **Edit**\.

   1. On the **Edit local logs configuration** dialog box, keep the default values for both log levels and storage sizes, and then choose **Save**\.

   You can use logs to troubleshoot any issues you might encounter when running this tutorial\. When troubleshooting issues, you can temporarily change the logging level to **Debug**\. For more information, see [Accessing file system logs](greengrass-logs-overview.md#gg-logs-local)\.

1. <a name="disable-stream-manager-no-java"></a>If the Java 8 runtime isn't installed on your core device, you must install it or disable stream manager\.
**Note**  
This tutorial doesn't use stream manager, but it does use the **Default Group creation** workflow that enables stream manager by default\. If stream manager is enabled but Java 8 isn't installed, the group deployment fails\. For more information, see the [stream manager requirements](stream-manager.md#stream-manager-requirements)\.

   To disable stream manager:

   1. On the group settings page, choose the **Lambda functions** tab\.

   1. Under the **System Lambda functions** section, select **Stream manager** and choose **Edit**\.

   1. Choose **Disable**, and then choose **Save**\.