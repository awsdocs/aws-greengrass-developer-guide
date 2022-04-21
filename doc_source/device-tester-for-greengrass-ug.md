--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Using AWS IoT Device Tester for AWS IoT Greengrass V1<a name="device-tester-for-greengrass-ug"></a>

AWS IoT Device Tester \(IDT\) is a downloadable testing framework that lets you validate IoT devices\. Because AWS IoT Greengrass Version 1 has been moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html), IDT for AWS IoT Greengrass V1 no longer generates signed qualification reports\. You will no longer be able to qualify new AWS IoT Greengrass V1 devices to list in the [AWS Partner Device Catalog](https://devices.amazonaws.com/) through the [AWS Device Qualification Program](https://aws.amazon.com/partners/dqp/)\. However, you can continue to use IDT for AWS IoT Greengrass V1 to test your Greengrass V1 devices\. We recommend that you use [IDT for AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/device-tester-for-greengrass-ug.html) to qualify and list Greengrass devices in the [AWS Partner Device Catalog](https://devices.amazonaws.com/)\.

IDT for AWS IoT Greengrass runs on your host computer \(Windows, macOS, or Linux\) connected to the device to be tested\. It runs tests and aggregates results\. It also provides a command line interface to manage the testing process\.

## AWS IoT Greengrass qualification suite<a name="gg-qual-suite"></a>

Use IDT for AWS IoT Greengrass to verify that the AWS IoT Greengrass Core software runs on your hardware and can communicate with the AWS Cloud\. It also performs end\-to\-end tests with AWS IoT Core\. For example, it verifies that your device can send and receive MQTT messages and process them correctly\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/devicetester_gg.png)

AWS IoT Device Tester for AWS IoT Greengrass organizes tests using the concepts of *test suites* and *test groups*\.<a name="idt-test-suites-groups"></a>
+ A test suite is the set of test groups used to verify that a device works with particular versions of AWS IoT Greengrass\.
+ A test group is the set of individual tests related to a particular feature, such as Greengrass group deployments and MQTT messaging\.

 For more information, see [Use IDT to run the AWS IoT Greengrass qualification suite](idt-gg-qualification.md)\.

## Custom test suites<a name="custom-test-suite"></a>

<a name="idt-byotc"></a>Starting in IDT v4\.0\.0, IDT for AWS IoT Greengrass combines a standardized configuration setup and result format with a test suite environment that enables you to develop custom test suites for your devices and device software\. You can add custom tests for your own internal validation or provide them to your customers for device verification\.

How a test writer configures a custom test suite determines the settings configurations that are required to run custom test suites\. For more information, see [Use IDT to develop and run your own test suites](idt-custom-tests.md)\.