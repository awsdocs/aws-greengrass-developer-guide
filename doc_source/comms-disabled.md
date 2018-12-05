# Test Communications \(Device Syncs Disabled\)<a name="comms-disabled"></a>

1. <a name="repeated-step"></a>Open two [command\-line](https://en.wikipedia.org/wiki/Command-line_interface) windows on your computer \(not the AWS IoT Greengrass core device\)\. One command\-line window is for the GG\_Switch device and the other is for the GG\_TrafficLight device\. Both scripts, when executed for the first time, run the AWS IoT Greengrass discovery service to connect to the AWS IoT Greengrass core \(through the internet\)\. After a device has discovered and successfully connected to the AWS IoT Greengrass core, future operations can be executed locally\. Before you run the following commands, make sure that your computer and the AWS IoT Greengrass core are connected to the internet using the same network\.

   For the GG\_Switch command\-line window, run the following:

   ```
   cd path-to-certs-folder
   python lightController.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert switch.cert.pem --key switch.private.key --thingName GG_TrafficLight --clientId GG_Switch
   ```

   For the GG\_TrafficLight command\-line window, run the following:

   ```
   cd path-to-certs-folder
   python trafficLight.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert light.cert.pem --key light.private.key --thingName GG_TrafficLight --clientId GG_TrafficLight
   ```
**Note**  
To find the endpoint to use for *AWS\_IOT\_ENDPOINT*, open the AWS IoT console and choose **Settings**\.

   Every 20 seconds, the switch updates the shadow state to G, Y, and R, and the light displays its new state, as shown next\.

   GG\_Switch output:  
![\[Screenshot of the output associated with GG_Switch.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-083.png)

   GG\_TrafficLight output:  
![\[Screenshot of the output associated with GG_TrafficLight.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-084.png)

1. In the AWS IoT console, choose your AWS IoT Greengrass group, choose **Devices**, and then choose **GG\_TrafficLight**:  
![\[Screenshot of Devices page with GG_TrafficLIght highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-085.png)

   Choose **Shadow**\. After the GG\_Switch changes states, there should not be any updates to this shadow topic in **Shadow State**\. That's because the GG\_TrafficLight is set to **LOCAL SHADOW ONLY** as opposed to **SHADOW SYNCING TO CLOUD**\.

1. Press Ctrl \+ C in the GG\_Switch \(`lightController.py`\) command\-line window\. You should see that the GG\_TrafficLight \(`trafficLight.py`\) window stops receiving state change messages\.