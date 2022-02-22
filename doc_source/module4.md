--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

--------

# Module 4: Interacting with devices in an AWS IoT Greengrass group<a name="module4"></a>

This module shows you how AWS IoT devices can connect to and communicate with an AWS IoT Greengrass core device\. AWS IoT devices that connect to an AWS IoT Greengrass core are part of an AWS IoT Greengrass group and can participate in the AWS IoT Greengrass programming paradigm\. In this module, one Greengrass device sends a Hello World message to another device in the Greengrass group\.

![\[AWS IoT connected to an AWS IoT Greengrass core, which is connected to AWS IoT Greengrass device #1 and AWS IoT Greengrass device #2.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-065.5.png)

Before you begin, run the [Greengrass device setup](quick-start.md) script or complete [Module 1](module1.md) and [Module 2](module2.md)\. This module creates two simulated devices\. You do not need other components or devices\.

This module should take less than 30 minutes to complete\.

**Topics**
+ [Create AWS IoT devices in an AWS IoT Greengrass group](device-group.md)
+ [Configure subscriptions](config-subs.md)
+ [Install the AWS IoT Device SDK for Python](IoT-SDK.md)
+ [Test communications](test-comms.md)