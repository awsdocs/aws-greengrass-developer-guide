# Configure Devices and Subscriptions<a name="config-dev-subs"></a>

1. Create two devices in your AWS IoT Greengrass group, **GG\_Switch** and **GG\_TrafficLight**\. Use the default security settings\.   
![\[Screenshot showing the two devices, GG_TrafficLight and GG_Switch.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-078.png)

   Save the certificates for the devices to your computer\. Make a note of the GUID\-like filename component for the GG\_Switch and GG\_TrafficLight devices\. You need them later in this module\. You can reuse the root CA that you downloaded in the previous module\.

   Each shadow can be synced to AWS IoT when the AWS IoT Greengrass core is connected to the internet\. First, you use local shadows without syncing the shadows to the cloud\. Later in the module, you enable syncing\. By default, cloud syncing should be disabled\. If it's not disabled, under **Devices**, choose the ellipsis \(**…**\), and then choose **Local Shadow Only**\.  
![\[Screenshot with LOCAL SHADOW ONLY highlighted for both devices.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-079.png)

1. Choose **Subscriptions** and create the following subscriptions for your group\. see For example, to set up the first row subscription, choose **Add Subscription**\. For **Select a source**, choose **Select**\. Choose the **Devices** tab, and then choose **GG\_Switch**\. For **Select a target**, choose **Select**, choose **Local Shadow Service**, and then choose **Next**\. For **Optional topic filter**, enter **$aws/things/GG\_TrafficLight/shadow/update**, choose **Next**, and then choose **Finish**\. Use the following table to create the remaining subscriptions:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/config-dev-subs.html)

   For information about the $ sign, see [Reserved Topics](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html#reserved-topics)\.
**Note**  
Although you can use wildcards \(for example, `$aws/things/GG_TrafficLight/shadow/#`\) to consolidate some of the subscriptions, we do not recommend this practice\.

   The topic paths must be written exactly as shown in the table\. Do not include an extra **/** at the end of a topic\. To see the full path, hover your mouse in the **Topic** column\.  
![\[Tabular data on the Subscriptions page. The page contains Source, Target, and Topic columns.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-080.png)
**Note**  
Each device has its own device shadow service\. For more information, see [Shadow MQTT Topics](https://docs.aws.amazon.com/iot/latest/developerguide/device-shadow-mqtt.html)\. 

1. Make sure that the AWS IoT Greengrass daemon is running, as described in [Deploy Cloud Configurations to a Core Device](configs-core.md)\.

1. On the group configuration page, from **Actions**, choose **Deploy** to deploy the updated group configuration to your AWS IoT Greengrass core device\.  
![\[Screenshot of the Deployments page with Deploy highlighted under the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-081.png)

   For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.