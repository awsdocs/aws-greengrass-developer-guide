# Verify the Lambda Function Is Running on the Device<a name="lambda-check"></a>

From the navigation pane of the AWS IoT console, choose **Test**\.

![\[Screenshot of AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-test.png)

In **Subscription topic**, enter **hello/world**\. \(Don't choose **Subscribe to topic** yet\.\) For **Quality of Service**, choose **0**\. For **MQTT payload display**, choose **Display payloads as strings \(more accurate\)**\. 

![\[Screenshot of Subscriptions.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-044.png)

Next, choose **Subscribe to topic**\.

Assuming the Lambda function is running on your device, it publishes messages similar to the following to the `hello/world` topic:

![\[Screenshot of message sent to the hello/world topic with message highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-045.png)

**Note**  
Although the Lambda function running on the AWS IoT Greengrass core device continues to send MQTT messages to the `hello/world` topic in the AWS IoT cloud, don't stop the AWS IoT Greengrass daemon\. The remaining modules are written with the assumption that it's running\.