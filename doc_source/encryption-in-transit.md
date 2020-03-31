# Encryption in Transit<a name="encryption-in-transit"></a>

AWS IoT Greengrass has three modes of communication where data is in transit:
+ [Data in Transit Over the Internet](#data-in-transit-internet)\. Communication between a Greengrass core and AWS IoT Greengrass over the internet is encrypted\.
+ [Data in Transit Over the Local Network](#data-in-transit-local-network)\. Communication between a Greengrass core and connected devices over a local network is encrypted\.
+ [Data On the Core Device](#data-in-transit-locally)\. Communication between components on the Greengrass core device is not encrypted\.

## Data in Transit Over the Internet<a name="data-in-transit-internet"></a>

AWS IoT Greengrass uses Transport Layer Security \(TLS\) to encrypt all communication over the internet\. All data sent to the AWS Cloud is sent over an TLS connection using MQTT or HTTPS protocols, so it is secure by default\. AWS IoT Greengrass uses the AWS IoT transport security model\. For more information, see [Transport Security](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html) in the *AWS IoT Core Developer Guide*\.

## Data in Transit Over the Local Network<a name="data-in-transit-local-network"></a>

AWS IoT Greengrass uses TLS to encrypt all communication over the local network between the Greengrass core and connected Greengrass devices\. For more information, see [Supported Cipher Suites for Local Network Communication](gg-sec.md#gg-cipher-suites)\.

It is your responsibility to protect the local network and private keys\.<a name="customer-responsibility-device-security"></a>

For Greengrass core devices, it's your responsibility to:  
+ Keep the kernel updated with the latest security patches\.
+ Keep system libraries updated with the latest security patches\.
+ Protect private keys\. For more information, see [Key Management for the Greengrass Core Device](key-management.md)\.

For connected devices, it's your responsibility to:  
+ Keep the TLS stack up to date\.
+ Protect private keys\.

## Data On the Core Device<a name="data-in-transit-locally"></a>

AWS IoT Greengrass doesn't encrypt data exchanged locally on the Greengrass core device because the data doesn't leave the device\. This includes communication between user\-defined Lambda functions, connectors, the AWS IoT Greengrass Core SDK, and system components, such as stream manager\.