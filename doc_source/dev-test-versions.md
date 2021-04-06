--------

You are viewing the documentation for AWS IoT Greengrass Version 1\. AWS IoT Greengrass Version 2 is the latest major version of AWS IoT Greengrass\. For more information about using AWS IoT Greengrass Version 2, see the [https://docs.aws.amazon.com/greengrass/v2/developerguide](https://docs.aws.amazon.com/greengrass/v2/developerguide)\.

--------

# Supported versions of AWS IoT Device Tester for AWS IoT Greengrass V1<a name="dev-test-versions"></a>

This topic lists supported versions of IDT for AWS IoT Greengrass V1\. We recommend that you use the latest version of IDT for AWS IoT Greengrass V1 that supports your target version of AWS IoT Greengrass V1\. New releases of AWS IoT Greengrass might require you to download a new version of IDT for AWS IoT Greengrass\. 

For information about IDT for AWS IoT Greengrass V2, see [Using AWS IoT Device Tester for AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/device-tester-for-greengrass-ug.html) in the *AWS IoT Greengrass V2 Developer Guide*\. 

**Note**  
You receive a notification when you start a test run if IDT for AWS IoT Greengrass is not compatible with the version of AWS IoT Greengrass you are using\.

By downloading the software, you agree to the [AWS IoT Device Tester License Agreement](https://docs.aws.amazon.com/greengrass/latest/developerguide/idt-license.html)\.



## Latest IDT version for AWS IoT Greengrass<a name="idt-latest-version"></a>

You can use the latest version of IDT for AWS IoT Greengrass with the AWS IoT Greengrass versions listed here\. We recommend that you use the latest version of IDT if it supports your target AWS IoT Greengrass version\.

**IDT v4\.0\.2 for AWS IoT Greengrass**     
Supported AWS IoT Greengrass versions: v1\.11\.1, v1\.11\.0, v1\.10\.3    
Software downloads:  
+ IDT v4\.0\.2 with test suite GGQ\_1\.2\.2 for [Linux](https://docs.aws.amazon.com/greengrass/latest/developerguide/download-idt-gg-linux-4.0.2.html)
+ IDT v4\.0\.2 with test suite GGQ\_1\.2\.2 for [macOS](https://docs.aws.amazon.com/greengrass/latest/developerguide/download-idt-gg-mac-4.0.2.html)
+ IDT v4\.0\.2 with test suite GGQ\_1\.2\.2 for [Windows](https://docs.aws.amazon.com/greengrass/latest/developerguide/download-idt-gg-win-4.0.2.html)
<a name="unzip-package-to-local-drive"></a>IDT does not support being run by multiple users from a shared location, such as an NFS directory or a Windows network shared folder\. We recommend that you extract the IDT package to a local drive and run the IDT binary on your local workstation\.  
Release notes:  
+ Fixed an issue that caused IDT to mask Hardware Security Integration \(HSI\) errors\.
+ Enables you to develop and run your custom test suites using AWS IoT Device Tester for AWS IoT Greengrass\. For more information, see [Use IDT to develop and run your own test suites](idt-custom-tests.md)\.
+ Provides code signed IDT applications for macOS and Windows\. In macOS, if a security warning message displays, you might need to grant a security exception for IDT\. For more information, see [Security exception on macOS](idt-troubleshooting.md#macos-notarization-exception)\.
AWS IoT Greengrass doesn't provide a Dockerfile or a Docker image for version 1\.11\.1 of the AWS IoT Greengrass core software\. To test your device for Docker qualification, use an earlier version of AWS IoT Greengrass core software\.  
Test suite version:    
`GGQ_1.2.2`  
+ Released 2021\.03\.29\.
+ Bug fixes and improvements\.

## Earlier IDT versions for AWS IoT Greengrass<a name="idt-prev-versions"></a>

The following earlier versions of IDT for AWS IoT Greengrass are also supported\.

**IDT v3\.2\.0 for AWS IoT Greengrass**    
Supported AWS IoT Greengrass versions: v1\.11\.0, v1\.10\.1, v1\.10\.0    
Software downloads:  
+ IDT v3\.2\.0 with test suite GGQ\_1\.2\.0 for [Linux](https://docs.aws.amazon.com/greengrass/latest/developerguide/download-idt-gg-linux-3.2.0.html)
+ IDT v3\.2\.0 with test suite GGQ\_1\.2\.0 for [macOS](https://docs.aws.amazon.com/greengrass/latest/developerguide/download-idt-gg-mac-3.2.0.html)
+ IDT v3\.2\.0 with test suite GGQ\_1\.2\.0 for [Windows](https://docs.aws.amazon.com/greengrass/latest/developerguide/download-idt-gg-win-3.2.0.html)
<a name="unzip-package-to-local-drive"></a>IDT does not support being run by multiple users from a shared location, such as an NFS directory or a Windows network shared folder\. We recommend that you extract the IDT package to a local drive and run the IDT binary on your local workstation\.  
Release notes:  
+ By default, IDT runs only required tests for qualification\. To qualify for additional features, you can modify the [`device.json`](set-config.md#device-config) file\.
+ Added a port number in `device.json` that you can configure for SSH connections\.
+ Docker supports only [stream manager](stream-manager.md) and machine learning \(ML\) without containerization\. Container, Docker, and Hardware Security Integration \(HSI\) are not available for Docker devices\.
+ We merged `device-ml.json` and `device-hsm.json` into `device.json`\.

For more information, see [Support policy for AWS IoT Device Tester for AWS IoT Greengrass](idt-support-policy.md)\.

## Unsupported IDT versions for for AWS IoT Greengrass<a name="idt-unsupported-versions"></a>

This topic lists unsupported versions of IDT for AWS IoT Greengrass\. Unsupported versions do not receive bug fixes or updates\. For more information, see [Support policy for AWS IoT Device Tester for AWS IoT Greengrass](idt-support-policy.md)\.

**IDT v3\.1\.3 AWS IoT Greengrass versions: v1\.10\.x, v1\.9\.x, v1\.8\.x**  
Release notes:  
+ Added support for ML feature qualification for AWS IoT Greengrass v1\.10\.x and v1\.9\.x\. You can now use IDT to validate that your devices can perform ML inference locally with models stored and trained in the cloud\.
+ Added `--stop-on-first-failure` for the `run-suite` command\. You can use this option to configure IDT to stop running on the first failure\. We recommend using this option during the debugging stage at the test groups level\.
+ Added a clock drift check for MQTT tests to ensure that the device under test uses the correct system time\. The time used must be within an acceptable time range\.
+ Added `--update-idt` for the `run-suite` command\. You can use this option to set the response for the prompt to update IDT\.
+ Added `--update-managed-policy` for the `run-suite` command\. You can use this option to set the response for the prompt to update the managed policy\.
+ Added a bug fix for automatic updates of IDT test suite versions\. The fix ensures that IDT can run the latest test suites that are available for your AWS IoT Greengrass version\.
 

**IDT v3\.0\.1 for AWS IoT Greengrass**    
Release notes:  
+ Added support for AWS IoT Greengrass v1\.10\.1\.
+ Automatic updates of IDT test suite versions\. IDT can download the latest test suites that are available for your AWS IoT Greengrass version\. With this feature:
  + Test suites are versioned using a `major.minor.patch` format\. The initial test suite version is `GGQ_1.0.0`\.
  + You can download new test suites interactively in the command line interface or set the `upgrade-test-suite` flag when you start IDT\.

  For more information, see [Test suite versions](idt-gg-qualification.md#idt-test-suite-versions)\.
+ Added `list-supported-products`\. You can use this command to list the AWS IoT Greengrass and test suite versions that are supported by the installed version of IDT\.
+ Added `list-test-cases`\. You can use this command to list the test cases that are available in a test group\.
+ Added `test-id` for the `run-suite` command\. You can use this option to run individual test cases in a test group\.  
Test suite version:    
`GGQ_1.0.0`  <a name="ggq-1.0.0"></a>
+ Released 2020\.04\.02\.
+ Applied new version numbering format\.
 

**IDT v2\.3\.0 for AWS IoT Greengrass v1\.10, v1\.9\.x, and v1\.8\.x**  
When testing on a physical device, AWS IoT Greengrass v1\.10, v1\.9\.x, and v1\.8\.x are supported\.  
When testing in a Docker container, AWS IoT Greengrass v1\.10 and v1\.9\.x are supported\.  
Release notes:  
+ Added support for [Running AWS IoT Greengrass in a Docker container](run-gg-in-docker-container.md)\. You can now use IDT to qualify and validate that your devices can run AWS IoT Greengrass in a Docker container\.
+ Added an [AWS managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) \(`AWSIoTDeviceTesterForGreengrassFullAccess`\) that defines the permissions required to run AWS IoT Device Tester\. If new releases require additional permissions, AWS adds them to this managed policy so you don't have to update your IAM permissions\.
+ Introduced checks to validate that your environment \(for example, device connectivity and internet connectivity\) is set up correctly before you run the test cases\.
+ Improved the Greengrass dependency checker in IDT to make it more flexible while checking for libc on devices\.
 

**IDT v2\.2\.0 for AWS IoT Greengrass v1\.10, v1\.9\.x, and v1\.8\.x**  
Release notes:  
+ Added support for AWS IoT Greengrass v1\.10\.
+ Added support for the [Greengrass Docker application deployment](docker-app-connector.md) connector\.
+ Added support for AWS IoT Greengrass [stream manager](stream-manager.md)\.
+ Added support for AWS IoT Greengrass in the China \(Beijing\) Region\.
 

**IDT v2\.1\.0 for AWS IoT Greengrass v1\.9\.x, v1\.8\.x, and v1\.7\.x**  
Release notes:  
+ Added support for AWS IoT Greengrass v1\.9\.4\.
+ Added support for Linux\-ARMv6l devices\.
 

**IDT v2\.0\.0 for AWS IoT Greengrass v1\.9\.3, v1\.9\.2, v\.1\.9\.1, v1\.9\.0, v1\.8\.4, v1\.8\.3, and v1\.8\.2**  
Release notes:  
+ Removed dependency on Python for device under test\.
+ Test suite execution time reduced by more than 50 percent, which makes the qualification process faster\.
+ Executable size reduced by more than 50 percent, which makes download and installation faster\.
+ Improved [timeout multiplier support](idt-troubleshooting.md#test-timeout) for all test cases\.
+ Enhanced post\-diagnostics messages to troubleshoot errors faster\.
+ Updated the permissions policy template required to run IDT\.
+ Added support for AWS IoT Greengrass v1\.9\.3\.
 

**IDT v1\.3\.3 for AWS IoT Greengrass v1\.9\.2, v1\.9\.1, v1\.9\.0, v1\.8\.3, and v1\.8\.2**  
Release notes:  
+ Added support for Greengrass v1\.9\.2 and v1\.8\.3\.
+ Added support for Greengrass OpenWrt\.
+ Added SSH user name and password device sign\-in\.
+ Added native test bug fix for OpenWrt\-ARMv7l platform\.
 

**IDT v1\.2 for AWS IoT Greengrass v1\.8\.1**  
Release notes:  
+ Added a configurable timeout multiplier to address and troubleshoot timeout issues \(for example, low bandwidth connections\)\.
 

**IDT v1\.1 for AWS IoT Greengrass v1\.8\.0**  
Release notes:  
+ Added support for AWS IoT Greengrass Hardware Security Integration \(HSI\)\.
+ Added support for AWS IoT Greengrass container and no container\.
+ Added automated AWS IoT Greengrass service role creation\.
+ Improved test resource cleanup\.
+ Added test execution summary report\.
 

**IDT v1\.1 for AWS IoT Greengrass v1\.7\.1**  
Release notes:  
+ Added support for AWS IoT Greengrass Hardware Security Integration \(HSI\)\.
+ Added support for AWS IoT Greengrass container and no container\.
+ Added automated AWS IoT Greengrass service role creation\.
+ Improved test resource cleanup\.
+ Added test execution summary report\.
 

**IDT v1\.0 for AWS IoT Greengrass v1\.6\.1**  
Release notes:  
+ Added OTA test bug fix for future AWS IoT Greengrass version compatibility\.
If you're using IDT v1\.0 for AWS IoT Greengrass v1\.6\.1, you must create a [Greengrass service role](service-role.md)\. In later versions, IDT creates the service role for you\.