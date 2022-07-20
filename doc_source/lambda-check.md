--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Verify the Lambda function is running on the core device<a name="lambda-check"></a>

1. From the navigation pane of the [AWS IoT console](https://console.aws.amazon.com/iot/), under **Test**, choose **MQTT test client**\.

1. Choose the **Subscribe to topic** tab\.

1. Enter **hello/world** into the **Topic filter** and expand the **Additional configuration**\.

1. Enter the information listed in each of the following fields:
   + For **Quality of Service**, choose **0**\.
   + For **MQTT payload display**, choose **Display payloads as strings**\.

    

1. Choose **Subscribe**\.

Assuming the Lambda function is running on your device, it publishes messages similar to the following to the `hello/world` topic:

![\[Screenshot of message sent to the hello/world topic with message highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-045.png)

Although the Lambda function continues to send MQTT messages to the `hello/world` topic, don't stop the AWS IoT Greengrass daemon\. The remaining modules are written with the assumption that it's running\.

You can delete the function and subscription from the group:
+ On the groups configuration page, under the **Lambda functions** tab, select the Lambda function you want to remove and choose **Remove**\.
+ On the groups configuration page, under the **Subscriptions** tab, choose the subscription, and then choose **Delete**\.

The function and subscription are removed from the core during the next group deployment\.