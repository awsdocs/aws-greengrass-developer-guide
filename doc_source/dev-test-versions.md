# Supported Versions of AWS IoT Device Tester for AWS IoT Greengrass<a name="dev-test-versions"></a>

This topic lists supported versions of IDT for AWS IoT Greengrass\. As a best practice, we recommend that you use the latest version of IDT for AWS IoT Greengrass that supports your target version of AWS IoT Greengrass\. New releases of AWS IoT Greengrass might require you to download a new version of IDT for AWS IoT Greengrass\.

**Note**  
You receive a notification when you start a test run if IDT for AWS IoT Greengrass is not compatible with the version of AWS IoT Greengrass you are using\.

By downloading the software, you agree to the [AWS IoT Device Tester License Agreement](https://d232ctwt5kahio.cloudfront.net/greengrass/AWS%20IoT%20Device%20Tester%20License%20Agreement.pdf)\.

## Latest IDT Version for AWS IoT Greengrass<a name="idt-latest-version"></a>

The latest version of IDT for AWS IoT Greengrass can be used with the AWS IoT Greengrass versions listed here\. We recommend that you use the latest version of IDT if it supports your target AWS IoT Greengrass version\.

**IDT v2\.3\.0 for AWS IoT Greengrass v1\.10, v1\.9\.x, and v1\.8\.x**  
When testing on a physical device, AWS IoT Greengrass v1\.10, v1\.9\.x, and v1\.8\.x are supported\.  
When testing in a Docker container, AWS IoT Greengrass v1\.10 and v1\.9\.x are supported\.  
Software downloads:  
+ IDT for AWS IoT Greengrass: [Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_2.3.0.zip)
+ IDT for AWS IoT Greengrass: [macOS](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_2.3.0.zip)
+ IDT for AWS IoT Greengrass: [Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_2.3.0.zip)
Release notes:  
+ Added support for [Running AWS IoT Greengrass in a Docker Container](run-gg-in-docker-container.md)\. You can now use IDT to qualify and validate that your devices can run AWS IoT Greengrass in a Docker container\.
+ Added an [AWS managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies) \(`AWSIoTDeviceTesterForGreengrassFullAccess`\) that defines the permissions required to run AWS IoT Device Tester\. If new releases require additional permissions, AWS adds them to this managed policy so you don't have to update your IAM permissions\.
+ Introduced checks to validate that your environment \(for example, device connectivity and internet connectivity\) is set up correctly before you run the test cases\.
+ Improved the Greengrass dependency checker in IDT to make it more flexible while checking for libc on devices\.

## Earlier IDT Versions for AWS IoT Greengrass<a name="idt-prev-versions"></a>

The following earlier versions of IDT for AWS IoT Greengrass are also supported\.

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
 

**IDT v2\.0\.0 for AWS IoT Greengrass v1\.9\.3, v1\.9\.2, v\.1\.9\.1, v1\.9\.0, v1\.8\.4, v1\.8\.3, and v1\.8\.2**  
Software downloads:  
+ IDT for AWS IoT Greengrass: [Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_2.0.0.zip)
+ IDT for AWS IoT Greengrass: [macOS](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_2.0.0.zip)
+ IDT for AWS IoT Greengrass: [Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_2.0.0.zip)
Release notes:  
+ Removed dependency on Python for device under test\.
+ Test suite execution time reduced by more than 50 percent, which makes the qualification process faster\.
+ Executable size reduced by more than 50 percent, which makes download and installation faster\.
+ Improved [timeout multiplier support](idt-troubleshooting.md#test-timeout) for all test cases\.
+ Enhanced post\-diagnostics messages for faster troubleshooting of errors\.
+ Updated the permissions policy template required to run IDT\.
+ Added support for AWS IoT Greengrass v1\.9\.3\.
 

 

For more information, see [Support Policy for AWS IoT Device Tester for AWS IoT Greengrass](idt-support-policy.md)\.