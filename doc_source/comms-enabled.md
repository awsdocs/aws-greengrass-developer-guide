--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Test communications \(device syncs enabled\)<a name="comms-enabled"></a>

For this test, you configure the GG\_TrafficLight device shadow to sync to AWS IoT\. You run the same commands as in the previous test, but this time the shadow state in the cloud is updated when GG\_Switch sends an update request\.

1. In the AWS IoT console, choose your AWS IoT Greengrass group, and then choose the **Client devices** tab\.

1. Select the GG\_TrafficLight device, choose **Sync shadow**, and then choose **Enable shadow sync with cloud**\.

   You should receive a notification that the device shadow sync status was updated\.

1. <a name="console-actions-deploy"></a>On the group configuration page, choose **Deploy**\.

1. In your two command\-line windows, run the commands from the previous test for the [GG\_Switch](comms-disabled.md#run-switch-device) and [GG\_TrafficLight](comms-disabled.md#run-trafficlight-device) client devices\.

1. Now, check the shadow state in the AWS IoT console\. Choose your AWS IoT Greengrass group, choose the **Client devices** tab, choose **GG\_TrafficLight**, choose the **Device Shadows** tab, and then choose **Classic Shadow**\.

   Because you enabled sync of the GG\_TrafficLight shadow to AWS IoT, the shadow state in the cloud should be updated whenever GG\_Switch sends an update\. This functionality can be used to expose the state of a client device to AWS IoT\.
**Note**  
If necessary, you can troubleshoot issues by viewing the AWS IoT Greengrass core logs, particularly `runtime.log`:  

   ```
   cd /greengrass/ggc/var/log
   sudo cat system/runtime.log | more
   ```
 You can also view `GGShadowSyncManager.log` and `GGShadowService.log`\. For more information, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\. 

Keep the client devices and subscriptions set up\. You use them in the next module\. You also run the same commands\.