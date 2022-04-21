--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Configure subscriptions<a name="config_subs"></a>

In this step, you create a subscription that enables the GG\_TrafficLight shadow to send updated state information to the GG\_Car\_Aggregator Lambda function\. This subscription is added to the subscriptions that you created in [Module 5](module5.md), which are all required for this module\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.

1. On the **Select your source and target** page, set the following values:
   + For **Select a source**, choose **Services**, and then choose **Local Shadow Service**\.
   + For **Select a target**, choose **Lambdas**, and then choose **GG\_Car\_Aggregator**\.

   Choose **Next**\.  
![\[The Select your source and target page with source set to Local Shadow Service and target set to the GG_Car_Aggregator Lambda function.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-098.5.png)

1. On the **Filter your data with a topic** page, for **Topic filter**, enter the following topic:

   **$aws/things/GG\_TrafficLight/shadow/update/documents**  
![\[Topic filter field set to $aws/things/GG_TrafficLight/shadow/update/documents.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-098.7.png)

1. Choose **Next**, and then choose **Finish**\.

   This module requires the new subscription and the [subscriptions](config-dev-subs.md#module5-subscriptions) that you created in Module 5\.

1. Make sure that the Greengrass daemon is running, as described in [Deploy cloud configurations to a core device](configs-core.md)\.

1. <a name="console-actions-deploy"></a>On the group configuration page, from **Actions**, choose **Deploy**\.  
![\[Screenshot of the Group page with the Deploy action highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-040.png)

   This deploys the group configuration to your AWS IoT Greengrass core device\. For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.