# Module 5: Interacting with Device Shadows<a name="module5"></a>

This advanced module shows you how AWS IoT Greengrass devices can interact with [AWS IoT device shadows](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html) in an AWS IoT Greengrass group\. A *shadow* is a JSON document that is used to store current or desired state information for a thing\. In this module, you discover how one AWS IoT Greengrass device \(`GG_Switch`\) can modify the state of another AWS IoT Greengrass device \(`GG_TrafficLight`\) and how these states can be synced to the AWS IoT Greengrass cloud:

![\[AWS IoT Greengrass core is connected to a traffic light device shadow and a light switch device.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-077.5.png)

Before you begin, run the [Greengrass Device Setup](quick-start.md) script, or make sure that you have completed [Module 1](module1.md) and [Module 2](module2.md)\. You should also understand how to connect devices to an AWS IoT Greengrass core \([Module 4](module4.md)\)\. You do not need other components or devices\.

This module should take about 30 minutes to complete\.

**Topics**
+ [Configure Devices and Subscriptions](config-dev-subs.md)
+ [Download Required Files](file-download.md)
+ [Test Communications \(Device Syncs Disabled\)](comms-disabled.md)
+ [Test Communications \(Device Syncs Enabled\)](comms-enabled.md)