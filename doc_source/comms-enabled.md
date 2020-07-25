# Test communications \(device syncs enabled\)<a name="comms-enabled"></a>

For this test, you configure the GG\_TrafficLight device shadow to sync to AWS IoT\. You run the same commands as in the previous test, but this time the shadow state in the cloud is updated when GG\_Switch sends an update request\.

1. In the AWS IoT console, choose your AWS IoT Greengrass group, and then choose **Devices**\.

1. For the GG\_TrafficLight device, choose the ellipsis \(**…**\), and then choose **Sync to the Cloud**\.  
![\[Screenshot with Sync to the Cloud highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-087.png)

   You should receive a notification that the device shadow was updated\.

1. <a name="console-actions-deploy"></a>On the group configuration page, from **Actions**, choose **Deploy**\.  
![\[Screenshot of the Group page with the Deploy action highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-040.png)

   This deploys the group configuration to your AWS IoT Greengrass core device\. For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

1. In your two command\-line windows, run the commands from the previous test for the [GG\_Switch](comms-disabled.md#run-switch-device) and [GG\_TrafficLight](comms-disabled.md#run-trafficlight-device) devices\.

1. Now, check the shadow state in the AWS IoT console\. Choose your AWS IoT Greengrass group, choose **Devices**, choose **GG\_TrafficLight**, and then choose **Shadow**\.

   Because you enabled sync of the GG\_TrafficLight shadow to AWS IoT, the shadow state in the cloud should be updated whenever GG\_Switch sends an update\. This functionality can be used to expose the state of a Greengrass device to AWS IoT\.  
![\[Shadow state showing "G" for the desired property and the reported property.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-088.png)  
![\[After 20 seconds, shadow state showing "Y" for the desired property and the reported property.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-089.png)
**Note**  
If necessary, you can troubleshoot issues by viewing the AWS IoT Greengrass core logs, particularly `runtime.log`:  

   ```
   cd /greengrass/ggc/var/log
   sudo cat system/runtime.log | more
   ```
 You can also view `GGShadowSyncManager.log` and `GGShadowService.log`\. For more information, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\. 

Keep the devices and subscriptions set up\. You use them in the next module\. You also run the same commands\.