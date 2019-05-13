# Test Group Descriptions<a name="dt-test-groups"></a>

Combination \(Device Security Interaction\)  
The combination test group verifies the functionality of the DCM and IPD on the AWS IoT Greengrass core device by changing connectivity information on the AWS IoT Greengrass group in the cloud\. The test group rotates the AWS IoT Greengrass server certificate and verifies that AWS IoT Greengrass allows connections\.

Dependencies  
The dependencies test group checks if the device meets the required technical dependencies for AWS IoT Greengrass\.

Deployment  
The deployment test group is used to determine if AWS IoT Greengrass cores can be redeployed after a failing pinned Python Lambda deployment\.

Hardware Security Integration \(HSI\)  
The HSI test group is used to verify that the provided shared library can interface with the HSM and implements the required PKCS\#11 API correctly\. The HSM/Shared library should sign a CSR, perform TLS operations and provide the correct key lengths and public key algorithm\.

Device Certificate Manager \(DCM\)  
The DCM test group is used to verify that the AWS IoT Greengrass device certificate manager can generate a server certificate on startup and rotate them if they will expire soon\.

IP Detector \(IPD\)  
The IPD test group is used to verify that core connectivity information is updated when there are IP address changes in an AWS IoT Greengrass core device\.

Local Resource Access  
The LRA test group is used to verify the Local Resource Access \(LRA\) feature of AWS IoT Greengrass by providing access to local files and directories owned by various Linux users and groups to containerized Lambda functions through AWS IoT Greengrass LRA APIs\. Lambda functions should be allowed or denied access to local resources based on LRA configuration\.

Logging  
The logging test group is used to verify that the AWS IoT Greengrass logging service can write to a log file using a Python user Lambda function\.

MQTT  
The MQTT test group is used to verify the AWS IoT Greengrass message router functionality by sending messages on a topic that is routed to two Lambda functions\. 

Native  
The native test group is used to verify that AWS IoT Greengrass can run native \(compiled\) Lambda functions\.

Network  
The network test group is used to verify that socket connections can be established from a Lambda function\. These socket connections should be allowed or denied based on AWS IoT Greengrass `seccomp` configuration\.

Over\-the\-Air \(OTA\)  
The OTA test group is used to validate that OTA can process an OTA update from the cloud\.

Penetration  
The penetration test group checks if the AWS IoT Greengrass core software fails to start if hard link/soft link protection and `seccomp` are not enabled\. It is also used to verify other security\-related features\.

Shadow  
The shadow test group is used to verify local shadow and shadow cloud syncing functionality\.

Spooler  
The spooler test group is used to validate that the MQTT messages are queued with the default spooler configuration\.

Token Exchange Service \(TES\)  
The TES test group is used to verify that AWS IoT Greengrass can exchange its core certificate for valid AWS credentials\.

Version  
The version test group is used to verify that the specified version of AWS IoT Greengrass software is correct\.