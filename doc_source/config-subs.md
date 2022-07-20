--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Configure subscriptions<a name="config-subs"></a>

In this step, you enable the HelloWorld\_Publisher client device to send MQTT messages to the HelloWorld\_Subscriber client device\.

1. On the group configuration page, choose the **Subscriptions** tab, and then choose **Add**\.

1. On the **Create a subscription** page, do the following to configure the subscription:

   1. For **Source type**, choose **Client device**, and then choose **HelloWorld\_Publisher**\.

   1. Under **Target type**, choose **Client device**, and then choose **HelloWorld\_Subscriber**\.

   1. For **Topic filter**, enter **hello/world/pubsub**\.
**Note**  
You can delete subscriptions from the previous modules\. On the group's **Subscriptions** page, select the subscriptions to delete, and then choose **Delete**\.

   1. Choose **Create subscription**\.

1. <a name="enable-automatic-detection"></a>Make sure that automatic detection is enabled so the Greengrass core can publish a list of its IP addresses\. Client devices use this information to discover the core\. Do the following:

   1. On the group configuration page, choose the **Lambda functions** tab\.

   1. Under **System Lambda functions**, choose **IP detector**, and then choose **Edit**\.

   1. In the **Edit IP detector settings**, choose **Automatically detect and override MQTT broker endpoints**, and then choose **Save**\.

1. Make sure that the Greengrass daemon is running, as described in [Deploy cloud configurations to a core device](configs-core.md)\.

1. <a name="console-actions-deploy"></a>On the group configuration page, choose **Deploy**\.

The deployment status is displayed below the group name on the page header\. To see deployment details, choose the **Deployments** tab\.