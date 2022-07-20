--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Configure devices and subscriptions<a name="config-dev-subs"></a>

Shadows can be synced to AWS IoT when the AWS IoT Greengrass core is connected to the internet\. In this module, you first use local shadows without syncing to the cloud\. Then, you enable cloud syncing\.

Each client device has its own shadow\. For more information, see [Device shadow service for AWS IoT](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html) in the *AWS IoT Developer Guide*\.

1. On the group configuration page, choose the **Client devices** tab\.

1. From the **Client devices** tab, add two new client devices in your AWS IoT Greengrass group\. For detailed steps of this process, see [Create client devices in an AWS IoT Greengrass group](device-group.md)\.
   + Name the client devices **GG\_Switch** and **GG\_TrafficLight**\.
   + Generate and download the security resources for both client devices\.
   + Make a note of the certificate ID in the file names of the security resources for the client devices\. You use these values later\.

1. Create a folder on your computer for these client devices' security credentials\. Copy the certificates and keys into this folder\.

1. Make sure that the client devices are set to use local shadows and not sync with the AWS Cloud\. If not, select the client device, choose **Sync shadow**, and then choose **Disable shadow sync with cloud**\.

1. <a name="module5-subscriptions"></a>Add the subscriptions in the following table to your group\. For example, to create the first subscription:

   1. On the group configuration page, choose the **Subscriptions** tab, and then choose **Add**\.

   1. For **Source type**, choose **Client device**, and then choose **GG\_Switch**\.

   1. For **Target type**, choose **Service**, and then choose **Local Shadow Service**\.

   1. For **Topic filter**, enter **$aws/things/GG\_TrafficLight/shadow/update**

   1. Choose **Create subscription**\.

   The topics must be entered exactly as shown in the table\. Although it's possible to use wildcards to consolidate some of the subscriptions, we don't recommend this practice\. For more information, see [Shadow MQTT topics](https://docs.aws.amazon.com/iot/latest/developerguide/device-shadow-mqtt.html) in the *AWS IoT Developer Guide*\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/config-dev-subs.html)

   The new subscriptions are displayed on the **Subscriptions** tab\.
**Note**  
For information about the `$` character, see [Reserved topics](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#reserved-topics)\.

1. <a name="enable-automatic-detection"></a>Make sure that automatic detection is enabled so the Greengrass core can publish a list of its IP addresses\. Client devices use this information to discover the core\. Do the following:

   1. On the group configuration page, choose the **Lambda functions** tab\.

   1. Under **System Lambda functions**, choose **IP detector**, and then choose **Edit**\.

   1. In the **Edit IP detector settings**, choose **Automatically detect and override MQTT broker endpoints**, and then choose **Save**\.

1. Make sure that the Greengrass daemon is running, as described in [Deploy cloud configurations to a core device](configs-core.md)\.

1. <a name="console-actions-deploy"></a>On the group configuration page, choose **Deploy**\.