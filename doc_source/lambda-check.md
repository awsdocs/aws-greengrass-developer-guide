# Verify the Lambda Function Is Running on the Device<a name="lambda-check"></a>

From the left pane of the AWS IoT console, choose **Test**\.

![\[Screenshot of AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-043.png)

In **Subscription topic** field, type **hello/world** \(don't choose **Subscribe to topic** yet\)\. For **Quality of Service**, select **0**\. For **MQTT payload display**, select **Display payloads as strings \(more accurate\)**\. 

![\[Screenshot of Subscriptions.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-044.png)

Next, choose **Subscribe to topic**\.

Assuming the Lambda function is running on your device, it will publish messages to the **hello/world** topic similar to the following:

![\[Screenshot of message sent to the hello/world topic with message highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-045.png)

**Note**  
Although the Lambda function running on the AWS Greengrass core device continues to send MQTT messages to the **hello/world** topic in the AWS IoT cloud, don't stop the AWS Greengrass daemon because the remaining modules assume that it's running\.