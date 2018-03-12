# Test Communications<a name="comms-test"></a>

1. On your computer, open two [command\-line](https://en.wikipedia.org/wiki/Command-line_interface) windows\. Just as in [Module 5](module5.md), one window will be for the GG\_Switch device and the other for the GG\_TrafficLight device\.
**Note**  
These are the same commands that you ran in Module 5\.

   Run the following commands for the GG\_Switch device:

   ```
   cd path-to-certs-folder
   python lightController.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert switch.cert.pem --key switch.private.key --thingName GG_TrafficLight --clientId GG_Switch
   ```

   Run the following commands for the GG\_TrafficLight device:

   ```
   cd path-to-certs-folder
   python trafficLight.py --endpoint AWS_IOT_ENDPOINT --rootCA root-ca-cert.pem --cert light.cert.pem --key light.private.key --thingName GG_TrafficLight --clientId GG_TrafficLight
   ```

   Every 20 seconds, the switch updates the shadow state to G, Y, and R, and the light displays its new state\.

1. On every third green light \(every 3 minutes\), the function handler of the Lambda function is triggered, and a new DynamoDB record is created\. After `lightController.py` and `trafficLight.py` have run for three minutes, go to the AWS Management Console, search for and open the DynamoDB console\. Make sure that the **N\. Virgina** \(`us-east-1`\) region is selected, then choose **Tables** and choose the **CarStats** table\. Next, choose the **Items** tab:  
![\[Screenshots of DynamoDB console with CarStats and Items tab highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-099.png)

   You should see entries with basic statistics on cars passed \(one entry for every three minutes\)\. You may need to choose the refresh button \(two circular arrows\) to view updates to the **CarStats** table:  
![\[DynamoDB CarStats screenshot showing multiple record entries.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-100.png)
**Note**  
If necessary, you can troubleshoot issues by viewing the AWS Greengrass core logs, particularly `router.log`:  

   ```
   cd /greengrass/ggc/var/log
   sudo cat system/router.log | more
   ```
For more information, see [Troubleshooting AWS Greengrass Applications](gg-troubleshooting.md)\.

You have reached the end of this tutorial and should now understand the AWS Greengrass programming model and its fundamental concepts, including AWS Greengrass cores, groups, subscriptions, and the deployment process for Lambda functions running at the edge\.

You can delete the DynamoDB table and stop communications between the AWS Greengrass core device and the AWS IoT cloud\. To stop communications, open a terminal on the AWS Greengrass core device and run **one** of the following commands:

+ To shut down the AWS Greengrass core device:

  ```
  sudo halt
  ```

+ To stop the AWS Greengrass daemon:

  ```
  cd /greengrass/ggc/packages/1.3.0/
  sudo ./greengrassd stop
  ```