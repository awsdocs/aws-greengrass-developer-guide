# Module 5: Interacting with Device Shadows<a name="module5"></a>

This advanced module shows you how AWS Greengrass devices can interact with [AWS IoT device shadows](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html) in an AWS Greengrass group\. A *shadow* is a JSON document that is used to store current or desired state information for a thing\. In this module, you discover how one AWS Greengrass device \(`GG_Switch`\) can modify the state of another AWS Greengrass device \(`GG_TrafficLight`\) and how these states can be synced to the AWS Greengrass cloud:

![\[AWS Greengrass core is connected to a traffic light device shadow and a light switch device.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-077.5.png)

Before you begin, make sure that you have completed [Module 1](module1.md), [Module 2](module2.md), [Module 3 \(Part 1\)](module3-I.md), and [Module 3 \(Part 2\)](module3-II.md)\. You should also understand how to connect devices to an AWS Greengrass core \([Module 4](module4.md)\)\. You do not need other components or devices\. This module should take about 30 minutes to complete\.