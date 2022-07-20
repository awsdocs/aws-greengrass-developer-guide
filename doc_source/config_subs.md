--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Configure subscriptions<a name="config_subs"></a>

In this step, you create a subscription that enables the GG\_TrafficLight shadow to send updated state information to the GG\_Car\_Aggregator Lambda function\. This subscription is added to the subscriptions that you created in [Module 5](module5.md), which are all required for this module\.

1. On the group configuration page, choose the **Subscriptions** tab, and then choose **Add**\.

1. On the **Create a subscription** page, do the following:

   1. For **Source type**, choose **Service**, and then choose **Local Shadow Service**\.

   1. For **Target type**, choose **Lambda function**, and then choose **GG\_Car\_Aggregator**\.

   1. For **Topic filter**, enter **$aws/things/GG\_TrafficLight/shadow/update/documents**

   1. Choose **Create subscription**\.

   This module requires the new subscription and the [subscriptions](config-dev-subs.md#module5-subscriptions) that you created in Module 5\.

1. Make sure that the Greengrass daemon is running, as described in [Deploy cloud configurations to a core device](configs-core.md)\.

1. <a name="console-actions-deploy"></a>On the group configuration page, choose **Deploy**\.