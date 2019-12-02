# Configure Subscriptions<a name="config-subs"></a>

In this step, you enable the HelloWorld\_Publisher device to send MQTT messages to the HelloWorld\_Subscriber device\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.

1. Configure the subscription\.
   + Under **Select a source**, choose **Devices**, and then choose **HelloWorld\_Publisher**\.
   + Under **Select a target**, choose **Devices**, and then choose **HelloWorld\_Subscriber**\.
   + Choose **Next**\.

      
![\[Screenshot of the Select your source and target page with HelloWorld_Publisher, HelloWorld_Subscriber, and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-072.png)

1. For **Topic filter**, enter **hello/world/pubsub**, choose **Next**, and then choose **Finish**\.
**Note**  
You can delete subscriptions from the previous modules\. On the group's **Subscriptions** page, choose the ellipsis \(**…**\) associated with a subscription, and then choose **Delete**\.

1. Make sure that the Greengrass daemon is running, as described in [Deploy Cloud Configurations to a Core Device](configs-core.md)\.

1. <a name="console-actions-deploy"></a>On the group configuration page, from **Actions**, choose **Deploy**\.  
![\[Screenshot of the Group page with the Deploy action highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-040.png)

   This deploys the group configuration to your AWS IoT Greengrass core device\. For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

The deployment status is displayed below the group name on the page header\. To see deployment details, choose **Deployments**\.