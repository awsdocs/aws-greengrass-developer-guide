# Configure the Lambda Function for AWS IoT Greengrass<a name="config-lambda"></a>

You are now ready to configure your Lambda function for AWS IoT Greengrass\. In this step, you also configure a subscription that allows the Lambda function to communicate with AWS IoT and configure local logging for the Greengrass group\.

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
<a name="long-lived-lambda"></a>A *long\-lived* \(or *pinned*\) Lambda function starts automatically after AWS IoT Greengrass starts and keeps running in its own container\. This is in contrast to an *on\-demand* Lambda function, which starts when invoked and stops when there are no tasks left to execute\. For more information, see [Lifecycle Configuration for Greengrass Lambda Functions](lambda-functions.md#lambda-lifecycle)\.

1. Keep the default values for all other fields, such as **Run as**, **Containerization**, and **Input payload data type**, and choose **Update** to save your changes\. For information about Lambda function properties, see [Controlling Execution of Greengrass Lambda Functions by Using Group\-Specific Configuration](lambda-group-config.md)\.

   Next, create a subscription that allows the Lambda to send [MQTT](http://mqtt.org/) messages to AWS IoT\.

   A Greengrass Lambda function can exchange MQTT messages with:
   + [Devices](what-is-gg.md#greengrass-devices) in the Greengrass group\.
   + [Connectors](connectors.md) in the group\.
   + Other Lambda functions in the group\.
   + AWS IoT\.
   + The local shadow service\. For more information, see [Module 5: Interacting with Device Shadows](module5.md)\.

   The group uses subscriptions to control how these entities can communicate with each other\. Subscriptions provide predictable interactions and a layer of security\.

   A subscription consists of a source, target, and topic\. The source is the originator of the message\. The target is the destination of the message\. The topic allows you to filter the data that is sent from the source to the target\. The source or target can be a Greengrass device, Lambda function, connector, device shadow, or AWS IoT\.
**Note**  
A subscription is directed in the sense that messages flow in a specific direction: from the source to the target\. To allow two\-way communication, you must set up two subscriptions\.

   The `Greengrass_HelloWorld` Lambda function sends messages only to the `hello/world` topic in AWS IoT, so you only need to create one subscription from the Lambda function to AWS IoT\. You create this in the next step\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add your first Subscription**\.  
![\[Screenshot of the Group configuration page, with Subscription and Add your first Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-036.png)

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

   You can use logs to troubleshoot any issues you might encounter when running this tutorial\. For more information, see [Accessing File System Logs](greengrass-logs-overview.md#gg-logs-local)\.