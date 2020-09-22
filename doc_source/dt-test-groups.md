# Test group descriptions<a name="dt-test-groups"></a>

------
#### [ IDT v2\.0\.0 and later ]

**Required Test Groups for Core Qualification**  
These test groups are required to qualify your AWS IoT Greengrass device for the AWS Partner Device Catalog\.    
AWS IoT Greengrass Core Dependencies  
Validates that your device meets all software and hardware requirements for the AWS IoT Greengrass Core software\.  
The `Software Packages Dependencies` test case in this test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Deployment  
Validates that Lambda functions can be deployed on your device\.  
MQTT  
Verifies the AWS IoT Greengrass message router functionality by checking local communication between the Greengrass core and AWS IoT devices\.  
Over\-the\-Air \(OTA\)  
Validates that your device can successfully perform an OTA update of the AWS IoT Greengrass Core software\.  
<a name="n-a-docker"></a>This test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Version  
Checks that the version of AWS IoT Greengrass provided is compatible with the AWS IoT Device Tester version you are using\.

**Optional Test Groups**  
These test groups are optional\. If you choose to qualify for optional tests, your device is listed with additional capabilities in the AWS Partner Device Catalog\.    
Container Dependencies  
<a name="description-container"></a>Validates that the device meets all of the software and hardware requirements to run Lambda functions in container mode on a Greengrass core\.  
<a name="n-a-docker"></a>This test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Deployment Container  
<a name="description-deployment-container"></a>Validates that Lambda functions can be deployed on the device and run in container mode on a Greengrass core\.  
<a name="n-a-docker"></a>This test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Docker Dependencies \(Supported for IDT v2\.2\.0 and later\)  
<a name="description-docker"></a>Validates that the device meets all the required technical dependencies to use the Greengrass Docker application deployment connector to run containers  
<a name="n-a-docker"></a>This test group is not applicable when testing in a [Docker container](docker-config-setup.md)\.  
Hardware Security Integration \(HSI\)  
<a name="description-hsi"></a>Verifies that the provided HSI shared library can interface with the hardware security module \(HSM\) and implements the required PKCS\#11 APIs correctly\. The HSM and shared library must be able to sign a CSR, perform TLS operations, and provide the correct key lengths and public key algorithm\.  
Stream Manager Dependencies \(Supported for IDT v2\.2\.0 and later\)  
<a name="description-sm"></a>Validates that the device meets all of the required technical dependencies to run AWS IoT Greengrass stream manager\.  
Machine Learning Dependencies \(Supported for IDT v3\.1\.0 and later\)  
<a name="description-ml"></a>Validates that the device meets all of the required technical dependencies to perform ML inference locally\.  
Machine Learning Inference Tests \(Supported for IDT v3\.1\.0 and later\)  
<a name="description-mlit"></a>Validates that ML inference can be performed on the given device under test\. For more information, see [Optional: Configuring your device for ML qualification](idt-ml-qualification.md)\.  
Machine Learning Inference Container Tests \(Supported for IDT v3\.1\.0 and later\)  
<a name="description-mlict"></a>Validates that ML inference can be performed on the given device under test and run in container mode on a Greengrass core\. For more information, see [Optional: Configuring your device for ML qualification](idt-ml-qualification.md)\.

------
#### [ IDT v1\.3\.3 and earlier ]

**Required Test Groups for Core Qualification**  
These tests are required to qualify your AWS IoT Greengrass device for the AWS Partner Device Catalog\.    
AWS IoT Greengrass Core Dependencies  
Validates that your device meets all software and hardware requirements for the AWS IoT Greengrass Core software\.  
Combination \(Device Security Interaction\)  
Verifies the functionality of the device certificate manager and IP detection on the Greengrass core device by changing connectivity information on the Greengrass group in the cloud\. The test group rotates the AWS IoT Greengrass server certificate and verifies that AWS IoT Greengrass allows connections\.  
Deployment \(Required for IDT v1\.2 and earlier\)  
Validates that Lambda functions can be deployed on your device\.  
Device Certificate Manager \(DCM\)  
Verifies that the AWS IoT Greengrass device certificate manager can generate a server certificate on startup and rotate certificates if they are close to expiration\.  
IP Detection \(IPD\)  
Verifies that core connectivity information is updated when there are IP address changes in a Greengrass core device\. For more information, see [Activate automatic IP detection](gg-core.md#ip-auto-detect)\.  
Logging  
Verifies that the AWS IoT Greengrass logging service can write to a log file using a user Lambda function written in Python\.  
MQTT  
Verifies the AWS IoT Greengrass message router functionality by sending messages on a topic that is routed to two Lambda functions\.   
Native  
Verifies that AWS IoT Greengrass can run native \(compiled\) Lambda functions\.  
Over\-the\-Air \(OTA\)  
Validates that your device can successfully perform a OTA update of the AWS IoT Greengrass Core software\.  
Penetration  
Validates that the AWS IoT Greengrass Core software fails to start if hard link/soft link protection and [seccomp](https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt) are not enabled\. It is also used to verify other security\-related features\.  
Shadow  
Verifies local shadow and shadow cloud\-syncing functionality\.  
Spooler  
Validates that the MQTT messages are queued with the default spooler configuration\.  
Token Exchange Service \(TES\)  
Verifies that AWS IoT Greengrass can exchange its core certificate for valid AWS credentials\.  
Version  
Checks that the version of AWS IoT Greengrass provided is compatible with the AWS IoT Device Tester version you are using\.

**Optional Test Groups**  
These tests are optional\. If you choose to qualify for optional tests, your device is listed with additional capabilities in the AWS Partner Device Catalog\.    
Container Dependencies  
Checks that the device meets all of the required dependencies to run Lambda functions in container mode\.  
Hardware Security Integration \(HSI\)  
Verifies that the provided HSI shared library can interface with the hardware security module \(HSM\) and implements the required PKCS\#11 APIs correctly\. The HSM and shared library must be able to sign a CSR, perform TLS operations, and provide the correct key lengths and public key algorithm\.  
Local Resource Access  
Verifies the local resource access \(LRA\) feature of AWS IoT Greengrass by providing access to local files and directories owned by various Linux users and groups to containerized Lambda functions through AWS IoT Greengrass LRA APIs\. Lambda functions should be allowed or denied access to local resources based on local resource access configuration\.  
Network  
Verifies that socket connections can be established from a Lambda function\. These socket connections should be allowed or denied based on the Greengrass core configuration\.

------