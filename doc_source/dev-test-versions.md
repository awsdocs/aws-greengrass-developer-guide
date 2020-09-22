# Supported versions of AWS IoT Device Tester for AWS IoT Greengrass<a name="dev-test-versions"></a>

This topic lists supported versions of IDT for AWS IoT Greengrass\. As a best practice, we recommend that you use the latest version of IDT for AWS IoT Greengrass that supports your target version of AWS IoT Greengrass\. New releases of AWS IoT Greengrass might require you to download a new version of IDT for AWS IoT Greengrass\.

**Note**  
You receive a notification when you start a test run if IDT for AWS IoT Greengrass is not compatible with the version of AWS IoT Greengrass you are using\.

By downloading the software, you agree to the [AWS IoT Device Tester License Agreement](https://d232ctwt5kahio.cloudfront.net/greengrass/AWS%20IoT%20Device%20Tester%20License%20Agreement.pdf)\.

## Latest IDT version for AWS IoT Greengrass<a name="idt-latest-version"></a>

You can use the latest version of IDT for AWS IoT Greengrass with the AWS IoT Greengrass versions listed here\. We recommend that you use the latest version of IDT if it supports your target AWS IoT Greengrass version\.

For more information about using IDT, see [Test suite versions](run-tests.md#idt-test-suite-versions) and [Test group descriptions](dt-test-groups.md)\.

**IDT v3\.2\.0 for AWS IoT Greengrass**    
Supported AWS IoT Greengrass versions: v1\.11\.x, v1\.10\.x, v1\.9\.x    
Software downloads:  
+ IDT v3\.2\.0 with test suite GGQ\_1\.2\.0 for [Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_3.2.0.zip)
+ IDT v3\.2\.0 with test suite GGQ\_1\.2\.0 for [macOS](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_3.2.0.zip)
+ IDT v3\.2\.0 with test suite GGQ\_1\.2\.0 for [Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_3.2.0.zip)
<a name="unzip-package-to-local-drive"></a>IDT does not support being run by multiple users from a shared location, such as an NFS directory or a Windows network shared folder\. Doing so may result in crashes or data corruption\. We recommend that you extract the IDT package to a local drive and run the IDT binary on your local workstation\.  
Release notes:  
+ By default, IDT runs only required tests for qualification\. To qualify for additional features, you can modify the [`device.json`](set-config.md#device-config) file\.
+ Added a port number in `device.json` that you can configure for SSH connections\.
+ Docker supports only [stream manager](stream-manager.md) and machine learning \(ML\) without containerization\. Container, Docker, and Hardware Security Integration \(HSI\) are not available for Docker devices\.
+ We merged `device-ml.json` and `device-hsm.json` into `device.json`\.  
Test suite version:    
`GGQ_1.2.0`  
+ Released 2020\.09\.16\.
+ The test suite runs only required tests for qualification, unless you set the corresponding `value` in the `features` array to `yes`\.

## Earlier IDT versions for AWS IoT Greengrass<a name="idt-prev-versions"></a>

The following earlier versions of IDT for AWS IoT Greengrass are also supported\.

**IDT v3\.1\.3 AWS IoT Greengrass versions: v1\.10\.x, v1\.9\.x, v1\.8\.x**  
Software downloads:  
+ IDT v3\.1\.3 with test suite GGQ\_1\.1\.2 for [Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_3.1.3.zip)
+ IDT v3\.1\.3 with test suite GGQ\_1\.1\.2 for [macOS](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_3.1.3.zip)
+ IDT v3\.1\.3 with test suite GGQ\_1\.1\.2 for [Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_3.1.3.zip)
Release notes:  
+ Added support for ML feature qualification for AWS IoT Greengrass v1\.10\.x and v1\.9\.x\. You can now use IDT to validate that your devices can perform ML inference locally with models stored and trained in the cloud\.
+ Added `--stop-on-first-failure` for the `run-suite` command\. You can use this option to configure IDT to stop running on the first failure\. We recommend using this option during the debugging stage at the test groups level\.
+ Added a clock drift check for MQTT tests to ensure that the device under test uses the correct system time\. The time used must be within an acceptable time range\.
+ Added `--update-idt` for the `run-suite` command\. You can use this option to set the response for the prompt to update IDT\.
+ Added `--update-managed-policy` for the `run-suite` command\. You can use this option to set the response for the prompt to update the managed policy\.
+ Added a bug fix for automatic updates of IDT test suite versions\. The fix ensures that IDT can run the latest test suites that are available for your AWS IoT Greengrass version\.

**IDT v2\.3\.0 for AWS IoT Greengrass v1\.10, v1\.9\.x, and v1\.8\.x**  
When testing on a physical device, AWS IoT Greengrass v1\.10, v1\.9\.x, and v1\.8\.x are supported\.  
When testing in a Docker container, AWS IoT Greengrass v1\.10 and v1\.9\.x are supported\.  
Software downloads:  
+ IDT for AWS IoT Greengrass: [Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_2.3.0.zip)
+ IDT for AWS IoT Greengrass: [macOS](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_2.3.0.zip)
+ IDT for AWS IoT Greengrass: [Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_2.3.0.zip)
Release notes:  
+ Added support for [Running AWS IoT Greengrass in a Docker container](run-gg-in-docker-container.md)\. You can now use IDT to qualify and validate that your devices can run AWS IoT Greengrass in a Docker container\.
+ Added an [AWS managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) \(`AWSIoTDeviceTesterForGreengrassFullAccess`\) that defines the permissions required to run AWS IoT Device Tester\. If new releases require additional permissions, AWS adds them to this managed policy so you don't have to update your IAM permissions\.
+ Introduced checks to validate that your environment \(for example, device connectivity and internet connectivity\) is set up correctly before you run the test cases\.
+ Improved the Greengrass dependency checker in IDT to make it more flexible while checking for libc on devices\.
 

**IDT v2\.2\.0 for AWS IoT Greengrass v1\.10, v1\.9\.x, and v1\.8\.x**  
Software downloads:  
+ IDT for AWS IoT Greengrass: [Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_2.2.0.zip)
+ IDT for AWS IoT Greengrass: [macOS](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_2.2.0.zip)
+ IDT for AWS IoT Greengrass: [Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_2.2.0.zip)
Release notes:  
+ Added support for AWS IoT Greengrass v1\.10\.
+ Added support for the [Greengrass Docker application deployment](docker-app-connector.md) connector\.
+ Added support for AWS IoT Greengrass [stream manager](stream-manager.md)\.
+ Added support for AWS IoT Greengrass in the China \(Beijing\) Region\.
 

**IDT v2\.1\.0 for AWS IoT Greengrass v1\.9\.x, v1\.8\.x, and v1\.7\.x**  
Software downloads:  
+ IDT for AWS IoT Greengrass: [Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_2.1.0.zip)
+ IDT for AWS IoT Greengrass: [macOS](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_2.1.0.zip)
+ IDT for AWS IoT Greengrass: [Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_2.1.0.zip)
Release notes:  
+ Added support for AWS IoT Greengrass v1\.9\.4\.
+ Added support for Linux\-ARMv6l devices\.
 

For more information, see [Support policy for AWS IoT Device Tester for AWS IoT Greengrass](idt-support-policy.md)\.