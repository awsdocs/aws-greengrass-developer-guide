# Using AWS IoT Device Tester for AWS IoT Greengrass<a name="device-tester-for-greengrass-ug"></a>

You can use AWS IoT Device Tester \(IDT\) for AWS IoT Greengrass to verify that the AWS IoT Greengrass Core software runs on your hardware and can communicate with the AWS Cloud\. It also performs end\-to\-end tests with AWS IoT Core\. For example, it verifies your device can send and receive MQTT messages and process them correctly\. IDT for AWS IoT Greengrass generates test reports that you can submit to AWS IoT to add your hardware to the AWS Partner Device Catalog\. For more information, see [AWS Device Qualification Program](https://aws.amazon.com/partners/dqp/)\. 

IDT for AWS IoT Greengrass runs on your host computer \(Windows, macOS, or Linux\) connected to the device to be tested\. It runs tests and aggregates results\. It also provides a command line interface to manage the testing process\.

In addition to testing devices, IDT for AWS IoT Greengrass creates resources \(for example, AWS IoT things, AWS IoT Greengrass groups, Lambda functions, and so on\) in your AWS account to facilitate the qualification process\.

<a name="idt-aws-credentials"></a>To create these resources, IDT for AWS IoT Greengrass uses the AWS credentials configured in the `config.json` file to make API calls on your behalf\. These resources are provisioned at various times during a test\.

When you run IDT for AWS IoT Greengrass on your host computer, it performs the following steps:

1. Loads and validates your device and credentials configuration\.

1. Performs selected tests with the required local and cloud resources\.

1. Cleans up local and cloud resources\.

1. Generates tests reports that indicate if your board passed the tests required for qualification\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/devicetester_gg.png)

IDT for AWS IoT Greengrass organizes tests using the concepts of *test suites* and *test groups*\.<a name="idt-test-suites-groups"></a>
+ A test suite is the set of test groups used to verify that a device works with particular versions of AWS IoT Greengrass\.
+ A test group is the set of individual tests related to a particular feature, such as Greengrass group deployments and MQTT messaging\.

 For more information, see [Test suite versions](run-tests.md#idt-test-suite-versions) and [Test group descriptions](dt-test-groups.md)\.