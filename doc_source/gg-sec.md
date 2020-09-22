# Overview of AWS IoT Greengrass security<a name="gg-sec"></a>

AWS IoT Greengrass uses X\.509 certificates, AWS IoT policies, and IAM policies and roles to secure the applications that run on devices in your local Greengrass environment\.

The following diagram shows the components of the AWS IoT Greengrass security model:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-security.png)

A \- Greengrass service role  
A customer\-created IAM role assumed by AWS IoT Greengrass when accessing to your AWS resources from AWS IoT Core, AWS Lambda, and other AWS services\. For more information, see [Greengrass service role](service-role.md)\.

B \- Core device certificate  
An X\.509 certificate used to authenticate a Greengrass core with AWS IoT Core and AWS IoT Greengrass\. For more information, see [Device authentication and authorization for AWS IoT Greengrass](device-auth.md)\.

C \- Device certificate  
An X\.509 certificate used to authenticate a Greengrass \(connected\) device with AWS IoT Core and AWS IoT Greengrass\. For more information, see [Device authentication and authorization for AWS IoT Greengrass](device-auth.md)\.

D \- Group role  
A customer\-created IAM role assumed by AWS IoT Greengrass when calling AWS services from a Greengrass core\.  
You use this role to specify access permissions that your user\-defined Lambda functions and connectors need to access AWS services, such as DynamoDB\. You also use it to allow AWS IoT Greengrass to export stream manager streams to AWS services and write to CloudWatch Logs\. For more information, see [Greengrass group role](group-role.md)\.  
AWS IoT Greengrass doesn't use the Lambda execution role that's specified in AWS Lambda for the cloud version of a Lambda function\.

E \- MQTT server certificate  
The certificate used for Transport Layer Security \(TLS\) mutual authentication between a Greengrass core device and connected devices in the Greengrass group\. The certificate is signed by the group CA certificate, which is stored in the AWS Cloud\.

## Device connection workflow<a name="gg-sec-connection"></a>

This section describes how Greengrass connected devices connect to the AWS IoT Greengrass service and Greengrass core devices\. Greengrass connected devices are registered AWS IoT Core devices that are in the same Greengrass group as the core device\. 
+ A Greengrass core device uses its device certificate, private key, and the AWS IoT Core root CA certificate to connect to the AWS IoT Greengrass service\. On the core device, the `crypto` object in the [configuration file](gg-core.md#config-json) specifies the file path for these items\.
+ The Greengrass core device downloads group membership information from the AWS IoT Greengrass service\.
+ When a deployment is made to the Greengrass core device, the Device Certificate Manager \(DCM\) handles local server certificate management for the Greengrass core device\.
+ A connected device connects to the AWS IoT Greengrass service using its device certificate, private key, and the AWS IoT Core root CA certificate\. After making the connection, the AWS IoT Core device uses the Greengrass Discovery Service to find the IP address of its Greengrass core device\. The device also downloads the group CA certificate, which is used for TLS mutual authentication with the Greengrass core device\.
+ A connected device attempts to connect to the Greengrass core device, passing its device certificate and client ID\. If the client ID matches the thing name of the device and the certificate is valid \(part of the Greengrass group\), the connection is made\. Otherwise, the connection is terminated\. 

The AWS IoT policy for connected devices must grant the `greengrass:Discover` permission to allow devices to discover connectivity information for the core\. For more information about the policy statement, see [Discovery authorization](gg-discover-api.md#gg-discover-auth)\.

## Configuring AWS IoT Greengrass security<a name="gg-config-sec"></a>

**To configure your Greengrass application's security**

1. Create an AWS IoT Core thing for your Greengrass core device\.

1. Generate a key pair and device certificate for your Greengrass core device\.

1. Create and attach an [AWS IoT policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) to the device certificate\. The certificate and policy allow the Greengrass core device access to AWS IoT Core and AWS IoT Greengrass services\. For more information, see [Minimal AWS IoT policy for the core device](device-auth.md#gg-config-sec-min-iot-policy)\.
**Note**  
<a name="thing-policy-variable-not-supported"></a>The use of [thing policy variables](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html) \(`iot:Connection.Thing.*`\) in the AWS IoT policy for a core device is not supported\. The core uses the same device certificate to make [multiple connections](gg-core.md#connection-client-id) to AWS IoT Core but the client ID in a connection might not be an exact match of the core thing name\.

1. Create a [Greengrass service role](service-role.md)\. This IAM role authorizes AWS IoT Greengrass to access resources from other AWS services on your behalf\. This allows AWS IoT Greengrass to perform essential tasks, such as retrieving AWS Lambda functions and managing device shadows\.

   You can use the same service role across AWS Regions, but it must be associated with your AWS account in every AWS Region where you use AWS IoT Greengrass\.

1. \(Optional\) Create a [Greengrass group role](group-role.md)\. This IAM role grants permission to Lambda functions and connectors running on a Greengrass core to call AWS services\. For example, the [Kinesis Firehose connector](kinesis-firehose-connector.md) requires permission to write records to an Amazon Kinesis Data Firehose delivery stream\.

   You can attach only one role to a Greengrass group\.

1. Create an AWS IoT Core thing for each device that connects to your Greengrass core\.
**Note**  
You can also use existing AWS IoT Core things and certificates\.

1. Create device certificates, key pairs, and AWS IoT policies for each device that connects to your Greengrass core\.

## AWS IoT Greengrass core security principals<a name="gg-principals"></a>

The Greengrass core uses the following security principals: AWS IoT client, local MQTT server, and local secrets manager\. The configuration for these principals is stored in the `crypto` object in the `config.json` configuration file\. For more information, see [AWS IoT Greengrass core configuration file](gg-core.md#config-json)\.

This configuration includes the path to the private key used by the principal component for authentication and encryption\. AWS IoT Greengrass supports two modes of private key storage: hardware\-based or file system\-based \(default\)\. For more information about storing keys on hardware security modules, see [Hardware security integration](hardware-security.md)\.

**AWS IoT Client**  
The AWS IoT client \(IoT client\) manages communication over the internet between the Greengrass core and AWS IoT Core\. AWS IoT Greengrass uses X\.509 certificates with public and private keys for mutual authentication when establishing TLS connections for this communication\. For more information, see [X\.509 certificates and AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html) in the *AWS IoT Core Developer Guide*\.  
The IoT client supports RSA and EC certificates and keys\. The certificate and private key path are specified for the `IoTCertificate` principal in `config.json`\.

**MQTT Server**  
The local MQTT server manages communication over the local network between the Greengrass core and other Greengrass devices in the group\. AWS IoT Greengrass uses X\.509 certificates with public and private keys for mutual authentication when establishing TLS connections for this communication\.  
By default, AWS IoT Greengrass generates an RSA private key for you\. To configure the core to use a different private key, you must provide the key path for the `MQTTServerCertificate` principal in `config.json`\. You are responsible for rotating a customer\-provided key\.    
**Private key support**    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-sec.html)
The configuration of the private key determines related processes\. For the list of cipher suites that the Greengrass core supports as a server, see [TLS cipher suites support](#gg-cipher-suites)\.    
**If no private key is specified** \(default\)  
+ AWS IoT Greengrass rotates the key based on your rotation settings\.
+ The core generates an RSA key, which is used to generate the certificate\.
+ The MQTT server certificate has an RSA public key and an SHA\-256 RSA signature\.  
**If an RSA private key is specified** \(requires GGC v1\.7 or later\)  
+ You are responsible for rotating the key\.
+ The core uses the specified key to generate the certificate\.
+ The RSA key must have a minimum length of 2048 bits\.
+ The MQTT server certificate has an RSA public key and an SHA\-256 RSA signature\.  
**If an EC private key is specified** \(requires GGC v1\.9 or later\)  
+ You are responsible for rotating the key\.
+ The core uses the specified key to generate the certificate\.
+ The EC private key must use an NIST P\-256 or NIST P\-384 curve\.
+ The MQTT server certificate has an EC public key and an SHA\-256 RSA signature\.

  The MQTT server certificate presented by the core has an SHA\-256 RSA signature, regardless of the key type\. For this reason, clients must support SHA\-256 RSA certificate validation to establish a secure connection with the core\.

**Secrets Manager**  
The local secrets manager securely manages local copies of secrets that you create in AWS Secrets Manager\. It uses a private key to secure the data key that's used to encrypt the secrets\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.  
By default, the IoT client private key is used, but you can specify a different private key for the `SecretsManager` principal in `config.json`\. Only the RSA key type is supported\. For more information, see [Specify the private key for secret encryption](secrets.md#secrets-config-private-key)\.  
Currently, AWS IoT Greengrass supports only the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism for encryption and decryption of local secrets when using hardware\-based private keys\. If you're following vendor\-provided instructions to manually generate hardware\-based private keys, make sure to choose PKCS\#1 v1\.5\. AWS IoT Greengrass doesn't support Optimal Asymmetric Encryption Padding \(OAEP\)\.  
**Private key support**    
<a name="secrets-manager-private-key-support"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-sec.html)

## Managed subscriptions in the MQTT messaging workflow<a name="gg-msg-workflow"></a>

AWS IoT Greengrass uses a subscription table to define how MQTT messages can be exchanged between devices, functions, and connectors in a Greengrass group, and with AWS IoT Core or the local shadow service\. Each subscription specifies a source, target, and MQTT topic \(or subject\) over which messages are sent or received\. AWS IoT Greengrass allows messages to be sent from a source to a target only if a corresponding subscription is defined\.

A subscription defines the message flow in one direction only, from the source to the target\. To support two\-way message exchange, you must create two subscriptions, one for each direction\.

## TLS cipher suites support<a name="gg-cipher-suites"></a>

AWS IoT Greengrass uses the AWS IoT Core transport security model to encrypt communication with the cloud by using [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) [cipher suites](https://en.wikipedia.org/wiki/Cipher_suite)\. In addition, AWS IoT Greengrass data is encrypted when at rest \(in the cloud\)\. For more information about AWS IoT Core transport security and supported cipher suites, see [ Transport security](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html) in the *AWS IoT Core Developer Guide*\.

**Supported Cipher Suites for Local Network Communication**

As opposed to AWS IoT Core, the AWS IoT Greengrass core supports the following *local network* TLS cipher suites for certificate\-signing algorithms\. All of these cipher suites are supported when private keys are stored on the file system\. A subset are supported when the core is configured to use hardware security modules \(HSM\)\. For more information, see [AWS IoT Greengrass core security principals](#gg-principals) and [Hardware security integration](hardware-security.md)\. The table also includes the minimum version of AWS IoT Greengrass Core software required for support\.


|  | Cipher | HSM support | Minimum GGC version | 
| --- |--- |--- |--- |
| TLSv1\.2 | TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA | Supported | 1\.0 | 
| --- |--- |--- |--- |
| TLS\_ECDHE\_RSA\_WITH\_AES\_256\_CBC\_SHA | Supported | 1\.0 | 
| TLS\_ECDHE\_RSA\_WITH\_AES\_256\_GCM\_SHA384 | Supported | 1\.0 | 
| TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA | Not supported | 1\.0 | 
| TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256 | Not supported | 1\.0 | 
| TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA | Not supported | 1\.0 | 
| TLS\_RSA\_WITH\_AES\_256\_GCM\_SHA384 | Not supported | 1\.0 | 
| TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256 | Supported | 1\.9 | 
| TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384 | Supported | 1\.9 | 
| TLSv1\.1 | TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA | Supported | 1\.0 | 
| --- |--- |--- |--- |
| TLS\_ECDHE\_RSA\_WITH\_AES\_256\_CBC\_SHA | Supported | 1\.0 | 
| TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA | Not supported | 1\.0 | 
| TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA | Not supported | 1\.0 | 
| TLSv1\.0 | TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA | Supported | 1\.0 | 
| --- |--- |--- |--- |
| TLS\_ECDHE\_RSA\_WITH\_AES\_256\_CBC\_SHA | Supported | 1\.0 | 
| TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA | Not supported | 1\.0 | 
| TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA | Not supported | 1\.0 | 