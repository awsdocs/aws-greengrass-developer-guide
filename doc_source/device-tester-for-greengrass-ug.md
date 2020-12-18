# Using AWS IoT Device Tester for AWS IoT Greengrass<a name="device-tester-for-greengrass-ug"></a>

AWS IoT Device Tester \(IDT\) is a downloadable testing framework that lets you validate IoT devices\. You can use IDT for AWS IoT Greengrass to run the AWS IoT Greengrass qualification suite, and create and run custom test suites for your devices\.

IDT for AWS IoT Greengrass runs on your host computer \(Windows, macOS, or Linux\) connected to the device to be tested\. It runs tests and aggregates results\. It also provides a command line interface to manage the testing process\.

## AWS IoT Greengrass qualification suite<a name="gg-qual-suite"></a>

Use IDT for AWS IoT Greengrass to verify that the AWS IoT Greengrass Core software runs on your hardware and can communicate with the AWS Cloud\. It also performs end\-to\-end tests with AWS IoT Core\. For example, it verifies that your device can send and receive MQTT messages and process them correctly\. 

If you want to add your hardware to the AWS Partner Device Catalog, run the AWS IoT Greengrass qualification suite to generate test reports that you can submit to AWS IoT\. For more information, see [AWS Device Qualification Program](https://aws.amazon.com/partners/dqp/)\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/devicetester_gg.png)

 organizes tests using the concepts of *test suites* and *test groups*\.<a name="idt-test-suites-groups"></a>
+ A test suite is the set of test groups used to verify that a device works with particular versions of AWS IoT Greengrass\.
+ A test group is the set of individual tests related to a particular feature, such as Greengrass group deployments and MQTT messaging\.

 For more information, see [Use IDT to run the AWS IoT Greengrass qualification suite](idt-gg-qualification.md)\.

## Custom test suites<a name="custom-test-suite"></a>

<a name="idt-byotc"></a>Starting in IDT v4\.0\.0, IDT for AWS IoT Greengrass combines a standardized configuration setup and result format with a test suite environment that enables you to develop custom test suites for your devices and device software\. You can add custom tests for your own internal validation or provide them to your customers for device verification\.

How a test writer configures a custom test suite determines the settings configurations that are required to run custom test suites\. For more information, see [Use IDT to develop and run your own test suites](idt-custom-tests.md)\.