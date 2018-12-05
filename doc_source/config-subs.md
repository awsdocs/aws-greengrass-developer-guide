# Configure Subscriptions<a name="config-subs"></a>

In this step, you enable the HelloWorld\_Publisher device to send a Hello World message to the HelloWorld\_Subscriber device\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\. For **Select a source**, choose **Select**\. In the expanded section, choose **Devices**, and then choose **HelloWorld\_Publisher**\. Similarly, for **Select a target**, choose **HelloWorld\_Subscriber**\. Choose **Next**\.  
![\[Screenshot of the Select your source and target page with HelloWorld_Publisher, HelloWorld_Subscriber, and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-072.png)

   For **Optional topic filter**, enter **hello/world/pubsub**:  
![\[Screenshot of the Optional topic filter field set to hello/world/pubsub.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-073.png)

   Choose **Next**, and then choose **Finish**\.
**Note**  
To delete subscriptions from the previous modules, choose **Subscriptions**, choose the ellipsis \(**…**\) associated with a subscription, and then choose **Delete**\.

1. Make sure that the AWS IoT Greengrass daemon is running, as described in [Deploy Cloud Configurations to a Core Device](configs-core.md)\.

1. On the group configuration page, from **Actions**, choose **Deploy** to deploy the updated group configuration to your AWS IoT Greengrass core device\.  
![\[On the Deployments page, the Actions drop-down menu with Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-074.png)

   To confirm the deployment was successful, choose **Deployments**\. In the **Status** column, you should see a **Successfully completed** message\.

   For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.