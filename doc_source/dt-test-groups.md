# Test Group Descriptions<a name="dt-test-groups"></a>

------
#### [ IDT v2\.0\.0 and later ]

Docker Dependencies \(Supported for IDT v2\.2\.0 and later\)  
This test group checks if the device meets all the required technical dependencies to use the Greengrass Docker application deployment connector to run containers\.

Stream Manager Dependencies \(Supported for IDT v2\.2\.0 and later\)  
This test group checks if the device meets all the required technical dependencies to run AWS IoT Greengrass stream manager\.

Container Dependencies  
This test group checks if the device meets all the software and hardware requirements to run Lambda functions in container mode on a AWS IoT Greengrass core core\.

Deployment  
This test group validates Lambda functions can be deployed on your device\.

Deployment Container  
This test group validates that Lambda functions can be deployed on the device and run in container mode on a AWS IoT Greengrass core\.

AWS IoT Greengrass core Dependencies  
This test group checks if your device meets all software and hardware requirements for the AWS IoT Greengrass core software\.

Hardware Security Integration \(HSI\)  
This test group verifies that the provided HSI shared library can interface with the hardware security module \(HSM\) and implements the required PKCS\#11 APIs correctly\. The HSM and shared library must be able to sign a CSR, perform TLS operations, and provide the correct key lengths and public key algorithm\.

MQTT  
his test group is used to verify the AWS IoT Greengrass message router functionality by checking local communication between AWS IoT Greengrass core and AWS IoT devices\.

Over\-the\-Air \(OTA\)  
This test group validates that your device can successfully perform a AWS IoT Greengrass core OTA update\.

Version  
This test group checks that the version of AWS IoT Greengrass provided is compatible with the AWS IoT Device Tester version you are using\.

------
#### [ IDT v1\.3\.3 and earlier ]

Combination \(Device Security Interaction\)  
 This test group verifies the functionality of the device certificate manager and IP detection on the AWS IoT Greengrass core device by changing connectivity information on the AWS IoT Greengrass group in the cloud\. The test group rotates the AWS IoT Greengrass server certificate and verifies that AWS IoT Greengrass allows connections\.

Container Dependencies  
The container dependencies test group checks if the device meets all the required dependencies to run Lambda functions in container mode\.

Deployment  
This test group validates Lambda functions can be deployed on your device\.

Device Certificate Manager \(DCM\)  
This test group is used to verify that the AWS IoT Greengrass device certificate manager can generate a server certificate on startup and rotate certificates if they are close to expiration\.

AWS IoT Greengrass core Dependencies  
This test group checks if your device meets all software and hardware requirements for the AWS IoT Greengrass Core software\.

Hardware Security Integration \(HSI\)  
This test group verifies that the provided HSI shared library can interface with the hardware security module \(HSM\) and implements the required PKCS\#11 APIs correctly\. The HSM and shared library must be able to sign a CSR, perform TLS operations, and provide the correct key lengths and public key algorithm\.

IP Detection \(IPD\)  
This test group is used to verify that core connectivity information is updated when there are IP address changes in an AWS IoT Greengrass core device\. For more information, see [Activate Automatic IP Detection](gg-core.md#ip-auto-detect)\.

Local Resource Access  
This test group is used to verify the local resource access feature of AWS IoT Greengrass by providing access to local files and directories owned by various Linux users and groups to containerized Lambda functions through AWS IoT Greengrass LRA APIs\. Lambda functions should be allowed or denied access to local resources based on local resource access configuration\.

Logging  
This test group is used to verify that the AWS IoT Greengrass logging service can write to a log file using a user Lambda function written in Python\.

MQTT  
This test group is used to verify the AWS IoT Greengrass message router functionality by sending messages on a topic that is routed to two Lambda functions\. 

Native  
This test group is used to verify that AWS IoT Greengrass can run native \(compiled\) Lambda functions\.

Network  
This test group is used to verify that socket connections can be established from a Lambda function\. These socket connections should be allowed or denied based on AWS IoT Greengrass core configuration\.

Over\-the\-Air \(OTA\)  
This test group validates that your device can successfully perform a AWS IoT Greengrass core OTA update\.

Penetration  
This test group checks if the AWS IoT Greengrass Core software fails to start if hard link/soft link protection and [seccomp](https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt) are not enabled\. It is also used to verify other security\-related features\.

Shadow  
This test group is used to verify local shadow and shadow cloud\-syncing functionality\.

Spooler  
This test group is used to validate that the MQTT messages are queued with the default spooler configuration\.

Token Exchange Service \(TES\)  
This test group is used to verify that AWS IoT Greengrass can exchange its core certificate for valid AWS credentials\.

Version  
This test group checks that the version of AWS IoT Greengrass provided is compatible with the AWS IoT Device Tester version you are using\.

------