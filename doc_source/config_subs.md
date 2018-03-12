# Configure Subscriptions<a name="config_subs"></a>

In this step, you create a subscription that enables the GG\_TrafficLight shadow to send updated states to the GG\_Car\_Aggregator Lambda function\. This subscription is in addition to the subscriptions that you created in [Module 5](module5.md), which are all required for this module\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.

1. On the **Select your source and target** page, set the following values:

   + **Select a source** \- choose **Services** and then choose **Local Shadow Service**\.

   + **Select a target** \- choose **Lambdas** and then choose **GG\_Car\_Aggregator**\.

   Choose **Next**\.  
![\[The Select your source and target webpage with source set to Local Shadow Service and target set to the GG_Car_Aggregator Lambda function.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-098.5.png)

1. On the **Filter your data with a topic** page, for **Optional topic filter**, type **$aws/things/GG\_TrafficLight/shadow/update/documents**\.

   Choose **Next**, and then choose **Finish**\.  
![\[Optional topic filter field set to $aws/things/GG_TrafficLight/shadow/update/documents.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-098.7.png)
**Note**  
On the **Subscriptions** page, the target displays the names of the function version and the alias: **GG\_Car\_Aggregator:GG\_CarAggregator**\.

   The following table shows the complete list of subscriptions that this module requires\. The new shadow subscription appears in the last row of the table\. You created the other subscriptions in [Module 5](module5.md)\.  
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/config_subs.html)
**Note**  
Except for [Module 5](module5.md), you can delete the subscriptions from earlier modules that are not included in the table\.

1. Make sure that the AWS Greengrass daemon is running, as described in [Deploy Cloud Configurations to a Core Device](configs-core.md)\.

1. On the group configuration page, from the **Actions** menu, choose **Deploy** to deploy the updated group configuration to your AWS Greengrass core device\.