--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Verify the Lambda function is running on the core device<a name="lambda-check"></a>

1. From the navigation pane of the [AWS IoT console](https://console.aws.amazon.com/iot/), choose **Test**\.  
![\[Screenshot of AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/console-test.png)

1. Choose **Subscribe to topic**, and configure the following fields:
   + For **Subscription topic**, enter **hello/world**\. \(Don't choose **Subscribe** yet\.\)
   + For **Quality of Service**, choose **0**\.
   + For **MQTT payload display**, choose **Display payloads as strings**\.

      
![\[Screenshot of MQTT test client page with topic subscription information.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-044.png)

1. Under Subscriptions, choose the **hello/world** topic\.

Assuming the Lambda function is running on your device, it publishes messages similar to the following to the `hello/world` topic:

![\[Screenshot of message sent to the hello/world topic with message highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-045.png)

Although the Lambda function continues to send MQTT messages to the `hello/world` topic, don't stop the AWS IoT Greengrass daemon\. The remaining modules are written with the assumption that it's running\.

You can delete the function and subscription from the group:
+ From the **Lambdas** page, choose the ellipsis \(**…**\), and then choose **Remove function**\.
+ From the **Subscriptions** page, choose the ellipsis \(**…**\), and then choose **Delete**\.

The function and subscription are removed from the core during the next group deployment\.