# Test Group Descriptions<a name="dt-test-groups"></a>

Dependencies  
The dependencies test group checks if the device meets all the required technical dependencies for AWS IoT Greengrass\.

Combination \(Device Security Interaction\)  
The combination test group verifies the functionality of the AWS IoT Greengrass Core's device certificate manager and IP detector by changing connectivity information on the AWS IoT Greengrass group in the cloud\. The AWS IoT Greengrass server certificate should be rotated and AWS IoT Greengrass should allow connections after rotating the certificate\.

Device Certificate Manager \(DCM\)  
The DCM test group verifies that the AWS IoT Greengrass Device Certificate Manager is able to generate a server certificate upon startup and rotate them if they will expire soon\.

Deployment  
The Deployment test group verifies if AWS IoT Greengrass Cores can be re\-deployed after a failing pinned Python Lambda deployment\.

Local Resource Access \(LRA\)  
The LRA test group verifies the Local Resource Access \(LRA\) feature of AWS IoT Greengrass by providing access to local files and directories owned by various Linux users and groups to containerized Lambdas through AWS IoT Greengrass LRA APIs\. Lambdas functions should be allowed or denied access to local resources based on LRA configuration\.

MQTT  
The MQTT test group verifies the AWS IoT Greengrass message router functionality by sending messages on a topic which is routed to two Lambda functions\. Lambda functions should acknowlege they received the message\.

Native  
The native test group verifies that AWS IoT Greengrass is able to run native \(compiled\) Lambda functions\.

Network  
The network test group verifies socket connections are able to be established from within a Lambda function\. These socket connections should be allowed or denied based on AWS IoT Greengrass `seccomp` configuration\.

Penetration  
The penetration test group checks if the AWS IoT Greengrass core software fails to start if hard/soft\-link protection and `seccomp` are not enabled\. It also verifies other security related features\.

Over\-the\-Air \(OTA\)  
The OTA test group validates that OTA is able to process an OTA update from the cloud\.

Shadow  
The shadow test group verifies local shadow and shadow cloud syncing functionality\.

Spooler  
The spooler test group validates that the MQTT messages are queued with the default spooler configuration\.

Token Exchange Service \(TES\)  
The TES test group verifies that AWS IoT Greengrass is able to exchange it's core certificate for valid AWS credentials\.

IP Detector \(IPD\)  
The IPD test group verifies that core connectivity information is updated when there are IP changes in a AWS IoT Greengrass Core device\.

Version  
Verifies the specified version of AWS IoT Greengrass software is correct\.

Logging  
Verifies that the AWS IoT Greengrass logging service is able to write to a log file using a Python user Lambda function\.