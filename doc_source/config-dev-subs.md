# Configure devices and subscriptions<a name="config-dev-subs"></a>

Shadows can be synced to AWS IoT when the AWS IoT Greengrass core is connected to the internet\. In this module, you first use local shadows without syncing to the cloud\. Then, you enable cloud syncing\.

Each device has its own shadow\. For more information, see [Device shadow service for AWS IoT](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html) in the *AWS IoT Developer Guide*\.

1. From the **Devices** page, add two new devices in your AWS IoT Greengrass group\. For detailed steps of this process, see [Create AWS IoT devices in an AWS IoT Greengrass group](device-group.md)\.
   + Name the devices **GG\_Switch** and **GG\_TrafficLight**\.
   + Generate and download the 1\-Click default security resources for both devices\.
   + Make a note of the hash component in the file names of the security resources for the devices\. You use these values later\.

      
![\[Screenshot showing the two devices, GG_TrafficLight and GG_Switch.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-078.png)

1. Decompress the downloaded certificates and keys for both devices into a single folder on your computer\. For example, run the following command for each `.tar.gz` file\.

   ```
   tar -xzf hash-setup.tar.gz
   ```
**Note**  
On Windows, you can decompress `.tar.gz` files using a tool such as [7\-Zip](http://www.7-zip.org/) or [WinZip](http://www.winzip.com/)\.

1. Copy the `root-ca-cert.pem` file that you downloaded in the [previous module](device-group.md#root-ca-device) to this folder\.

1. Make sure that the devices are set to use local shadows\. If not, choose the ellipsis \(**…**\), and then choose **Make local only**\.  
![\[Screenshot with LOCAL SHADOW ONLY highlighted for both devices.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-079.png)

1. <a name="configure-ip-address"></a>The function code used in this module requires that you manually configure the core's endpoint\.

   1. On the group configuration page, choose **Settings**\. 

   1. For **Local connection detection**, choose **Manually manage connection information**, and then choose **View Cores for specific endpoint information**\.

   1. Choose your core, and then choose **Connectivity**\.

   1. Choose **Edit** and make sure that you have only one endpoint value\. This value must be the IP address endpoint for port 8883 of your AWS IoT Greengrass core device \(for example, `192.168.1.4`\)\.

   1. Choose **Update**\.

1. <a name="module5-subscriptions"></a>Add the subscriptions in the following table to your group\. For example, to create the first subscription:

   1. On the group configuration page, choose **Subscriptions**, and then choose **Add subscription**\.

   1. Under **Select a source**, choose **Devices**, and then choose **GG\_Switch**\.

   1. Under **Select a target**, choose **Services**, and then choose **Local Shadow Service**\.

   1. Choose **Next**\.

   1. For **Topic filter**, enter **$aws/things/GG\_TrafficLight/shadow/update**

   1. Choose **Next**, and then choose **Finish**\.

   The topics must be entered exactly as shown in the table\. Although it's possible to use wildcards to consolidate some of the subscriptions, we don't recommend this practice\. For more information, see [Shadow MQTT topics](https://docs.aws.amazon.com/iot/latest/developerguide/device-shadow-mqtt.html) in the *AWS IoT Developer Guide*\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/config-dev-subs.html)

   The new subscriptions are displayed on the **Subscriptions** page\. To see the full topic path of a subscription, hover your mouse over the **Topic** column\.  
![\[Tabular data on the Subscriptions page. The page contains Source, Target, and Topic columns.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-080.png)
**Note**  
For information about the `$` character, see [Reserved topics](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#reserved-topics)\.

1. <a name="enable-automatic-detection"></a>Make sure that automatic detection is enabled so the Greengrass core can publish a list of its IP addresses\. Devices use this information to discover the core\.

   1. On the group configuration page, choose **Settings**\.

   1. Under **Core connectivity information**, for **Local connection detection**, choose **Automatically detect and override connection information**\.

1. Make sure that the Greengrass daemon is running, as described in [Deploy cloud configurations to a core device](configs-core.md)\.

1. <a name="console-actions-deploy"></a>On the group configuration page, from **Actions**, choose **Deploy**\.  
![\[Screenshot of the Group page with the Deploy action highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-040.png)

   This deploys the group configuration to your AWS IoT Greengrass core device\. For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.