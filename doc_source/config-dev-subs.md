# Configure Devices and Subscriptions<a name="config-dev-subs"></a>

1. Create two devices in your AWS Greengrass group, **GG\_Switch** and **GG\_TrafficLight**\. Use the default security settings\. 
**Note**  
You can detach devices used in earlier modules\.  
![\[Screenshot showing the two devices, GG_TrafficLight and GG_Switch.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-078.png)

   Save the certificates for the devices to your computer – note the GUID\-like filename component for the **GG\_Switch** and **GG\_TrafficLight** devices, these will be needed later\. You can reuse the previous root CA from VeriSign or download a new one\.

   Now, each shadow can be synced to AWS IoT when the AWS Greengrass core is connected to the internet\. First, you'll use local shadows without syncing the shadows to the cloud\. Later in the module, you enable syncing\. By default, cloud syncing should be disabled\. If it's not disabled, under **Devices**, choose the ellipsis \(…\), and then choose **Local Shadow Only**\.  
![\[Screenshot with LOCAL SHADOW ONLY highlighted for both devices.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-079.png)

1. Choose **Subscriptions** and create the following subscriptions for your group \(For information on the $ sign, see [Reserved Topics](http://docs.aws.amazon.com/iot/latest/developerguide/topics.html#reserved-topics)\)\. For example, to set up the first row subscription, choose **Add Subscription**, for **Select a source** choose **Select**, choose the **Devices** tab, and then choose **GG\_Switch**\. For **Select a target** choose **Select**, choose **Local Shadow Service**, and then **Next**\. For **Optional topic filter**, type \(or copy/paste\) **$aws/things/GG\_TrafficLight/shadow/update**, choose **Next**, and then **Finish**\. Using a similar procedure, complete the remaining subscriptions:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/config-dev-subs.html)
**Note**  
Although you can use wildcards \(for example, `$aws/things/GG_TrafficLight/shadow/#`\) to consolidate some of the subscriptions, we do not recommend this practice\.

   The topic paths must be written exactly as shown in the table\. Do not include an extra **/** at the end of a topic\. You can hover your mouse over a **Topic** path to see the full path via tooltip popup:  
![\[Tabular data on the Subscriptions page. The page contains Source, Target, and Topic columns.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-080.png)
**Note**  
Each device has its own device shadow service\. For more information, see [Shadow MQTT Topics](http://docs.aws.amazon.com/iot/latest/developerguide/device-shadow-mqtt.html)\. 

1. Make sure that the AWS Greengrass daemon is running, as described in [Deploy Cloud Configurations to a Core Device](configs-core.md)\.

1. On the group configuration page, from the **Actions** menu, choose **Deploy** to deploy the updated group configuration to your AWS Greengrass core device\.  
![\[Screenshot of the Deployments page with Deploy highlighted under the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-081.png)