# Configure the Lambda function for AWS IoT Greengrass<a name="config-lambda"></a>

You are now ready to configure your Lambda function for AWS IoT Greengrass\.

In this step, you:
+ Use the AWS IoT console to add the Lambda function to your Greengrass group\.
+ Configure group\-specific settings for the Lambda function\.
+ Add a subscription to the group that allows the Lambda function to publish MQTT messages to AWS IoT\.
+ Configure local log settings for the group\.

 

1. In the AWS IoT console, under **Greengrass**, choose **Groups**, and then choose the group that you created in [Module 2](module2.md)\.

1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1. Choose **Use existing Lambda**\.  
![\[Screenshot of Add a Lambda to your Greengrass Group with the Use existing Lambda button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-032.png)

1. Search for the name of the Lambda you created in the previous step \(**Greengrass\_HelloWorld**, not the alias name\), select it, and then choose **Next**:  
![\[Screenshot of Use existing Lambda with Greengrass_HelloWorld and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-033.png)

1. For the version, choose **Alias: GG\_HelloWorld**, and then choose **Finish**\. You should see the **Greengrass\_HelloWorld** Lambda function in your group, using the **GG\_HelloWorld** alias\.

1. Choose the ellipsis \(**…**\), and then choose **Edit Configuration**:  
![\[Screenshot of MyFirstGroup with the ellipsis and Edit Configuration highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-034.png)

1. On the **Group\-specific Lambda configuration** page, make the following changes:
   + Set **Timeout** to 25 seconds\. This Lambda function sleeps for 20 seconds before each invocation\.
   + For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\.

      
![\[Screenshot of the configuration page with 25 (seconds) and the Make this function long-lived and keep it running indefinitely radio button selected.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-035.png)
**Note**  
<a name="long-lived-lambda"></a>A *long\-lived* \(or *pinned*\) Lambda function starts automatically after AWS IoT Greengrass starts and keeps running in its own container\. This is in contrast to an *on\-demand* Lambda function, which starts when invoked and stops when there are no tasks left to execute\. For more information, see [Lifecycle configuration for Greengrass Lambda functions](lambda-functions.md#lambda-lifecycle)\.

1. Keep the default values for all other fields, such as **Run as**, **Containerization**, and **Input payload data type**, and choose **Update** to save your changes\. For information about Lambda function properties, see [Controlling execution of Greengrass Lambda functions by using group\-specific configuration](lambda-group-config.md)\.

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

1. On the group configuration page, choose **Subscriptions**, and then choose **Add your first Subscription**\.  
![\[Screenshot of the Group configuration page, with Subscription and Add your first Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-036.png)

   For an example that shows you how to create a subscription using the AWS CLI, see [create\-subscription\-definition](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/greengrass/create-subscription-definition.html) in the *AWS CLI Command Reference*\.

1. In **Select a source**, choose **Select**\. Then, on the **Lambdas** tab, choose **Greengrass\_HelloWorld** as the source\.   
![\[Screenshot of the Select a source page with Lambdas and Greengrass_HelloWorld highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-037.png)

1. For **Select a target**, choose **Select**\. Then, on the **Service** tab, choose **IoT Cloud**, and then choose **Next**\.  
![\[The Select a target with the Services tab, IoT Cloud and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-038.png)

1. For **Topic filter**, enter **hello/world**, and then choose **Next**\.  
![\[Screenshot with hello/world highlighted under Topic filter.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-039.png)

1. Choose **Finish**\.

1. Configure the group's logging settings\. For this tutorial, you configure AWS IoT Greengrass system components and user\-defined Lambda functions to write logs to the file system of the core device\.

   1. On the group configuration page, choose **Settings**\.

   1. For **Local logs configuration**, choose **Edit**\.

   1. On the **Configure Group logging** page, choose **Add another log type**\.

   1. For event source, choose **User Lambdas** and **Greengrass system**, and then choose **Update**\.

   1. Keep the default values for logging level and disk space limit, and then choose **Save**\.

   You can use logs to troubleshoot any issues you might encounter when running this tutorial\. When troubleshooting issues, you can temporarily change the logging level to **Debug**\. For more information, see [Accessing file system logs](greengrass-logs-overview.md#gg-logs-local)\.

1. <a name="disable-stream-manager-no-java"></a>If the Java 8 runtime isn't installed on your core device, you must install it or disable stream manager\.
**Note**  
This tutorial doesn't use stream manager, but it does use the **Default Group creation** workflow that enables stream manager by default\. If stream manager is enabled but Java 8 isn't installed, the group deployment fails\. For more information, see the [stream manager requirements](stream-manager.md#stream-manager-requirements)\.

   To disable stream manager:

   1. On the group settings page, under **Stream manager**, choose **Edit**\.

   1. Choose **Disable**, and then choose **Save**\.