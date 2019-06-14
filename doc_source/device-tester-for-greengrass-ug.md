# Using AWS IoT Device Tester for AWS IoT Greengrass<a name="device-tester-for-greengrass-ug"></a>

You can use AWS IoT Device Tester \(IDT\) for AWS IoT Greengrass to verify that the AWS IoT Greengrass Core software runs on your hardware and can communicate with the AWS IoT Cloud\. It also performs end\-to\-end tests with AWS IoT Core\. For example, it verifies your device can send and receive MQTT messages and process them correctly\. IDT for AWS IoT Greengrass generates test reports that you can submit to AWS IoT to add your hardware to the AWS Partner Device Catalog\. For more information, see [AWS Device Qualification Program](https://aws.amazon.com/partners/dqp/)\. 

IDT for AWS IoT Greengrass runs on your host computer \(Windows, macOS, or Linux\) connected to the device to be tested\. It runs tests and aggregates results\. It also provides a command line interface to manage the testing process\.

In addition to testing devices, IDT for AWS IoT Greengrass creates resources \(for example, AWS IoT things, AWS IoT Greengrass groups, Lambda functions, and so on\) to facilitate the qualification process\.

To create these resources, IDT for AWS IoT Greengrass uses the AWS credentials configured in the `config.json` to make API calls on your behalf\. These resources are provisioned at various times during a test\.

When you run IDT for AWS IoT Greengrass on your host computer, it performs the following steps:

1. Loads and validates your device and credentials configuration\.

1. Performs selected tests with the required local and cloud resources\.

1. Cleans up local and cloud resources\.

1. Generates tests reports that indicate if your board passed the tests required for qualification\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/devicetester_gg.png)

IDT for AWS IoT Greengrass organizes tests using the concepts of a *test suite* and test groups\.

The test suite is the collection of all test groups used to verify a device works with AWS IoT Greengrass\.

A test group consists of all the individual tests related to a feature being tested\. For more information, see [Test Group Descriptions](dt-test-groups.md)\.

## AWS IoT Device Tester for AWS IoT Greengrass Versions<a name="dev-test-versions"></a>

The latest version of IDT for AWS IoT Greengrass supports the AWS IoT Greengrass versions listed here\. If you need to test an earlier version of AWS IoT Greengrass, see [Earlier IDT Versions for AWS IoT Greengrass](idt-prev-versions.md)\.

By downloading the software, you agree to the [AWS IoT Device Tester License Agreement](https://d232ctwt5kahio.cloudfront.net/greengrass/AWS%20IoT%20Device%20Tester%20License%20Agreement.pdf)\. New releases of AWS IoT Greengrass might require you to download a new version of IDT for AWS IoT Greengrass\. IDT for AWS IoT Greengrass notifies you if it is not compatible with the version of AWS IoT Greengrass you are using when you start a test run\. 

**IDT v1\.3\.1 for AWS IoT Greengrass v1\.9\.1, v1\.9\.0, and v1\.8\.2**
+ IDT for AWS IoT Greengrass: [Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_1.3.1.zip)
+ IDT for AWS IoT Greengrass: [macOS](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_1.3.1.zip)
+ IDT for AWS IoT Greengrass: [Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_1.3.1.zip)

**Release Notes:**
+ Added support for Greengrass v1\.9\.1

## Understanding Results and Logs<a name="results-logs"></a>

This section describes how to view and interpret IDT result reports and logs\. 

### Viewing Results<a name="view-results"></a>

While running, IDT writes errors to the console, log files, and test reports\. After IDT completes the qualification test suite, it generates two test reports\. These reports can be found in `<devicetester-extract-location> /results/<execution-id>/`\. Both reports capture the results from the qualification test suite execution\.

The `awsiotdevicetester_report.xml` is the qualification test report that you submit to AWS to list your device in the AWS Partner Device Catalog\. The report contains the following elements:
+ The IDT version\.
+ The AWS IoT Greengrass version that was tested\.
+ The SKU and the device pool name specified in the `device.json` file\.
+ The features of the device pool specified in the `device.json` file\.
+ The aggregate summary of test results\.
+ A breakdown of test results by libraries that were tested based on the device features \(for example, local resource access, shadow, MQTT, and so on\)\.

The `GGQ_Result.xml` report is in [JUnit XML format](https://llg.cubic.org/docs/junit/)\. You can integrate it into continuous integration/deployment platforms like [Jenkins](https://jenkins.io/), [Bamboo](https://www.atlassian.com/software/bamboo), and so on\. The report contains the following elements:
+ Aggregate summary of test results\.
+ Breakdown of test results by AWS IoT Greengrass functionality that was tested\.

#### Interpreting AWS IoT Device Tester Results<a name="interpreting-results-gg"></a>

The report section in `awsiotdevicetester_report.xml` or `awsiotdevicetester_report.xml` lists the tests that were run and the results\.

The first XML tag `<testsuites>` contains the summary of the test execution\. For example:

```
<testsuites name="GGQ results" time="2299" tests="28" failures="0" errors="0" disabled="0">
```Attributes used in the `<testsuites>` tag

`name`  
The name of the test suite\.

`time`  
The time, in seconds, it took to run the qualification suite\.

`tests`  
The number of tests executed\.

`failures`  
The number of tests that were run, but did not pass\.

`errors`  
The number of tests that IDT couldn't execute\.

`disabled`  
This attribute is not used and can be ignored\.

The `awsiotdevicetester_report.xml` file contains an `<awsproduct>` tag that contains information about the product being tested and the product features that were validated after running a suite of tests\.Attributes used in the `<awsproduct>` tag

`name`  
The name of the product being tested\.

`version`  
The version of the product being tested\.

`features`  
The features validated\. Features marked as `required` are required to submit your board for qualification\. The following snippet shows how this information appears in the `awsiotdevicetester_report.xml` file\.  

```
<feature name="aws-iot-greengrass-no-container" value="supported" type="required"><
/feature>
```
Features marked as `optional` are not required for qualification\. The following snippets show optional features\.  

```
<feature name="aws-iot-greengrass-container" value="supported" type="optional"></feature> 
<feature name="aws-iot-greengrass-hsi" value="not-supported" type="optional"></feature>
```

If there are no test failures or errors for the required features, your device meets the technical requirements to run AWS IoT Greengrass and can interoperate with AWS IoT services\. If you want to list your device in the AWS Partner Device Catalog, you can use this report as qualification evidence\.

In the event of test failures or errors, you can identify the test that failed by reviewing the `<testsuites>` XML tags\. The `<testsuite>` XML tags inside the `<testsuites>` tag show the test result summary for a test group\. For example:

```
<testsuite name="combination" package="" tests="1" failures="0" time="161" disabled="0" errors="0" skipped="0">
```

The format is similar to the `<testsuites>` tag, but with a `skipped` attribute that is not used and can be ignored\. Inside each `<testsuite>` XML tag, there are `<testcase>` tags for each executed test for a test group\. For example:

```
<testcase classname="Security Combination (IPD + DCM) Test Context" name="Security Combination IP Change Tests sec4_test_1: Should rotate server cert when IPD disabled and following changes are made:Add CIS conn info and Add another CIS conn info" attempts="1"></testcase>>
```Attributes used in the `<testcase>` tag

`name`  
The name of the test\.

`attempts`  
The number of times IDT executed the test case\.

When a test fails or an error occurs, `<failure>` or `<error>` tags are added to the `<testcase>` tag with information for troubleshooting\. For example:

```
<testcase classname="mcu.Full_MQTT" name="AFQP_MQTT_Connect_HappyCase" attempts="1">
	<failure type="Failure">Reason for the test failure</failure>
	<error>Reason for the test execution error</error>
</testcase>
```

#### Viewing Logs<a name="view-logs-gg"></a>

IDT generates logs from test execution in `<devicetester-extract-location>/results/<execution-id>/logs`\. Two sets of logs are generated:

`test_manager.log`  
Logs generated from the Test Manager component of AWS IoT Device Tester \(for example, logs related to configuration, test sequencing, and report generation\)\.

`<test_case_id>.log (for example, ota.log)`  
Logs of the test group, including logs from the device under test\. When a test fails, a tar\.gz file that contains the logs of the device under test for the test is created \(for example, `ota_prod_test_1_ggc_logs.tar.gz`\)\.

For more information, see [IDT for AWS IoT Greengrass Troubleshooting](idt-troubleshooting.md)\.