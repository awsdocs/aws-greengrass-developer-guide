--------

You are viewing the documentation for AWS IoT Greengrass Version 1\. AWS IoT Greengrass Version 2 is the latest major version of AWS IoT Greengrass\. For more information about using AWS IoT Greengrass V2, see the [https://docs.aws.amazon.com/greengrass/v2/developerguide](https://docs.aws.amazon.com/greengrass/v2/developerguide)\.

--------

# Configure subscriptions<a name="config-subs"></a>

In this step, you enable the HelloWorld\_Publisher device to send MQTT messages to the HelloWorld\_Subscriber device\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.

1. Configure the subscription\.
   + Under **Select a source**, choose **Devices**, and then choose **HelloWorld\_Publisher**\.
   + Under **Select a target**, choose **Devices**, and then choose **HelloWorld\_Subscriber**\.
   + Choose **Next**\.

      
![\[Screenshot of the Select your source and target page with HelloWorld_Publisher, HelloWorld_Subscriber, and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-072.png)

1. For **Topic filter**, enter **hello/world/pubsub**, choose **Next**, and then choose **Finish**\.
**Note**  
You can delete subscriptions from the previous modules\. On the group's **Subscriptions** page, choose the ellipsis \(**…**\) associated with a subscription, and then choose **Delete**\.

1. <a name="enable-automatic-detection"></a>Make sure that automatic detection is enabled so the Greengrass core can publish a list of its IP addresses\. Devices use this information to discover the core\.

   1. On the group configuration page, choose **Settings**\.

   1. Under **Core connectivity information**, for **Local connection detection**, choose **Automatically detect and override connection information**\.

1. Make sure that the Greengrass daemon is running, as described in [Deploy cloud configurations to a core device](configs-core.md)\.

1. <a name="console-actions-deploy"></a>On the group configuration page, from **Actions**, choose **Deploy**\.  
![\[Screenshot of the Group page with the Deploy action highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-040.png)

   This deploys the group configuration to your AWS IoT Greengrass core device\. For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

The deployment status is displayed below the group name on the page header\. To see deployment details, choose **Deployments**\.