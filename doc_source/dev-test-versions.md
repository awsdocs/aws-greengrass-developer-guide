# AWS IoT Device Tester for AWS IoT Greengrass Versions<a name="dev-test-versions"></a>

The latest version of IDT for AWS IoT Greengrass supports the AWS IoT Greengrass versions listed here\. If you need to test an earlier version of AWS IoT Greengrass, see [Earlier IDT Versions for AWS IoT Greengrass](idt-prev-versions.md)\.

By downloading the software, you agree to the [AWS IoT Device Tester License Agreement](https://d232ctwt5kahio.cloudfront.net/greengrass/AWS%20IoT%20Device%20Tester%20License%20Agreement.pdf)\. New releases of AWS IoT Greengrass might require you to download a new version of IDT for AWS IoT Greengrass\. You receive a notification when you start a test run if IDT for AWS IoT Greengrass is not compatible with the version of AWS IoT Greengrass you are using\. 

**IDT v2\.0\.0 for AWS IoT Greengrass v1\.9\.3, v1\.9\.2, v\.1\.9\.1, v1\.9\.0, v1\.8\.4, v1\.8\.3, and v1\.8\.2**
+ IDT for AWS IoT Greengrass: [Linux](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_linux_2.0.0.zip)
+ IDT for AWS IoT Greengrass: [macOS](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_mac_2.0.0.zip)
+ IDT for AWS IoT Greengrass: [Windows](https://d232ctwt5kahio.cloudfront.net/greengrass/devicetester_greengrass_win_2.0.0.zip)

**Release Notes:**
+ Removed dependency on Python for device under test\.
+ Test suite execution time reduced by more than 50 percent, which makes the qualification process faster\.
+ Executable size reduced by more than 50 percent, which makes download and installation faster\.
+ Improved [timeout multiplier support](https://docs.aws.amazon.com/greengrass/latest/developerguide/idt-troubleshooting.html#test-timeout) for all test cases\.
+ Enhanced post\-diagnostics messages for faster troubleshooting of errors\.
+ Updated [Permissions Policy Template](policy-template.md) required to run IDT\.
+ Added support for AWS IoT Greengrass v1\.9\.3\.