# Configure Subscriptions<a name="config-subs"></a>

In this step, you enable the **HelloWorld\_Publisher** device to send a `HelloWorld` message to the **HelloWorld\_Subscriber** device\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\. For **Select a source**, choose **Select**, **Devices**, and **HelloWorld\_Publisher**\. Similarly, for **Select a target**, choose **HelloWorld\_Subscriber**\. Lastly, choose **Next**:  
![\[Screenshot of the Select your source and target page with HelloWorld_Publisher, HelloWorld_Subscriber, and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-072.png)

   For **Optional topic filter**, type **hello/world/pubsub**:  
![\[Screenshot of the Optional topic filter field set to hello/world/pubsub.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-073.png)

   Then choose **Next** followed by **Finish**\.
**Note**  
You may delete subscriptions from the earlier modules\. To do so, choose **Subscriptions**, choose an ellipsis \(**…**\) associated with a subscription, and then choose **Delete**\.

1. Make sure that the AWS Greengrass daemon is running, as described in [Deploy Cloud Configurations to a Core Device](configs-core.md)\.

1. On the group configuration page, from the **Actions** menu, choose **Deploy** to deploy the updated group configuration to your AWS Greengrass core device:  
![\[On the Deployments page, the Actions drop-down menu with Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-074.png)

   To confirm a successful deployment, choose **Deployments** and you should see a **Successfully completed** message near the time you initiated the deployment\.

   For help troubleshooting any issues that you encounter, see [Troubleshooting AWS Greengrass Applications](gg-troubleshooting.md)\.