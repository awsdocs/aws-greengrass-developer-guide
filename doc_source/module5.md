# Module 5: Interacting with device shadows<a name="module5"></a>

This advanced module shows you how AWS IoT Greengrass devices can interact with [AWS IoT device shadows](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html) in an AWS IoT Greengrass group\. A *shadow* is a JSON document that is used to store current or desired state information for a thing\. In this module, you discover how one AWS IoT Greengrass device \(`GG_Switch`\) can modify the state of another AWS IoT Greengrass device \(`GG_TrafficLight`\) and how these states can be synced to the AWS IoT Greengrass cloud:

![\[AWS IoT Greengrass core is connected to a traffic light device shadow and a light switch device.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-077.5.png)

Before you begin, run the [Greengrass device setup](quick-start.md) script, or make sure that you have completed [Module 1](module1.md) and [Module 2](module2.md)\. You should also understand how to connect devices to an AWS IoT Greengrass core \([Module 4](module4.md)\)\. You do not need other components or devices\.

This module should take about 30 minutes to complete\.

**Topics**
+ [Configure devices and subscriptions](config-dev-subs.md)
+ [Download required files](file-download.md)
+ [Test communications \(device syncs disabled\)](comms-disabled.md)
+ [Test communications \(device syncs enabled\)](comms-enabled.md)