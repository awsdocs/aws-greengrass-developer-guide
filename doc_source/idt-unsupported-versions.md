# Unsupported versions of AWS IoT Device Tester for AWS IoT Greengrass<a name="idt-unsupported-versions"></a>

This topic lists unsupported versions of IDT for AWS IoT Greengrass\. Unsupported versions do not receive bug fixes or updates\. For more information, see [Support policy for AWS IoT Device Tester for AWS IoT Greengrass](idt-support-policy.md)\.

**IDT v3\.0\.1 for AWS IoT Greengrass**    
Release notes:  
+ Added support for AWS IoT Greengrass v1\.10\.1\.
+ Automatic updates of IDT test suite versions\. IDT can download the latest test suites that are available for your AWS IoT Greengrass version\. With this feature:
  + Test suites are versioned using a `major.minor.patch` format\. The initial test suite version is `GGQ_1.0.0`\.
  + You can download new test suites interactively in the command line interface or set the `upgrade-test-suite` flag when you start IDT\.

  For more information, see [IDT for AWS IoT Greengrass test suite versions](run-tests.md#idt-test-suite-versions)\.
+ Added `list-supported-products`\. You can use this command to list the AWS IoT Greengrass and test suite versions that are supported by the installed version of IDT\.
+ Added `list-test-cases`\. You can use this command to list the test cases that are available in a test group\.
+ Added `test-id` for the `run-suite` command\. You can use this option to run individual test cases in a test group\.  
Test suite version:    
`GGQ_1.0.0`  <a name="ggq-1.0.0"></a>
+ Released 2020\.04\.02\.
+ Applied new version numbering format\.
 

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