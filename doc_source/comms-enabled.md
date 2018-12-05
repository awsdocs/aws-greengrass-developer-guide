# Test Communications \(Device Syncs Enabled\)<a name="comms-enabled"></a>

1. In the AWS IoT console, choose your AWS IoT Greengrass group, and then choose **Devices**\. For the GG\_TrafficLight device, choose the ellipsis \(**…**\), and then choose **Sync to the Cloud**:  
![\[Screenshot with Sync to the Cloud highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-087.png)

   You should receive a notification that the device shadow has been updated\.

1. From the **Deployments** page, deploy the updated configuration to your AWS IoT Greengrass core device\.

1. Repeat the [step in which you create two command\-line windows](comms-disabled.md#repeated-step)\.

1. In the AWS IoT console, choose your AWS IoT Greengrass group, choose **Devices**, choose **GG\_TrafficLight**, and then choose **Shadow**\.

   Because you enabled syncs of the GG\_TrafficLight shadow to AWS IoT, the shadow state in the cloud should be updated whenever GG\_Switch sends an update\. This functionality can be used to expose the state of an AWS IoT Greengrass device to the AWS IoT cloud\.  
![\[Shadow state showing "G" for the desired property and the reported property.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-088.png)  
![\[After 20 seconds, shadow state showing "Y" for the desired property and the reported property.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-089.png)
**Note**  
If necessary, you can troubleshoot issues by viewing the AWS IoT Greengrass core logs, particularly `runtime.log`:  

   ```
   cd /greengrass/ggc/var/log
   sudo cat system/runtime.log | more
   ```
 You can also view `GGShadowSyncManager.log` and `GGShadowService.log`\. For more information, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\. 