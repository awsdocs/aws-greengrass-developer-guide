# Verify the Lambda function is running on the core device<a name="lambda-check"></a>

1. From the navigation pane of the [AWS IoT console](https://console.aws.amazon.com/iot/), choose **Test**\.  
![\[Screenshot of AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-test.png)

1. Choose **Subscribe to topic**, and configure the following fields:
   + For **Subscription topic**, enter **hello/world**\. \(Don't choose **Subscribe to topic** yet\.\)
   + For **Quality of Service**, choose **0**\.
   + For **MQTT payload display**, choose **Display payloads as strings**\.

      
![\[Screenshot of Subscriptions test page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-044.png)

1. Choose **Subscribe to topic**\.

Assuming the Lambda function is running on your device, it publishes messages similar to the following to the `hello/world` topic:

![\[Screenshot of message sent to the hello/world topic with message highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-045.png)

Although the Lambda function continues to send MQTT messages to the `hello/world` topic, don't stop the AWS IoT Greengrass daemon\. The remaining modules are written with the assumption that it's running\.

You can delete the function and subscription from the group:
+ From the **Lambdas** page, choose the ellipsis \(**…**\), and then choose **Remove function**\.
+ From the **Subscriptions** page, choose the ellipsis \(**…**\), and then choose **Delete**\.

The function and subscription are removed from the core during the next group deployment\.