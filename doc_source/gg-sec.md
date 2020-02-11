# AWS IoT Greengrass Security<a name="gg-sec"></a>

AWS IoT Greengrass uses X\.509 certificates, managed subscriptions, AWS IoT policies, and IAM policies and roles to secure the applications that run on devices in your local Greengrass environment\.

Devices in Greengrass environments fall into one of the following two categories\. Both device types require an entry in the AWS IoT registry, a device certificate, and an AWS IoT policy\.
+ **AWS IoT Greengrass cores**\. Core devices use certificates and policies to securely connect to AWS IoT\. The certificates and policies also allow AWS IoT Greengrass to deploy configuration information, Lambda functions, connectors, and managed subscriptions to core devices\.
+ **AWS IoT devices that connect to a Greengrass core**\. These Greengrass devices use certificates and policies to securely connect to AWS IoT and AWS IoT Greengrass services\. This allows devices to use the Greengrass Discovery service to find and connect to a core device\. A Greengrass device uses the same certificate to connect to the AWS IoT device gateway and core device\.

The following diagram shows the components of the AWS IoT Greengrass security model:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-security.png)

A \- Greengrass service role  
A customer\-created IAM role that allows AWS IoT Greengrass access to your AWS IoT, Lambda, and other AWS resources\. For more information, see [Greengrass Service Role](service-role.md)\.

B \- Core device certificate  
An X\.509 certificate used to authenticate an AWS IoT Greengrass core\.

C \- Device certificate  
An X\.509 certificate used to authenticate an AWS IoT Greengrass device\.

D \- Group role  
A customer\-created IAM role assumed by AWS IoT Greengrass when calling AWS services from a Lambda function or connector on an AWS IoT Greengrass core\.  
This is the Lambda execution role for all the Greengrass Lambda functions and connectors that run on the core\. Use this role to specify access permissions that your user\-defined Lambda functions and connectors need to access AWS services, such as DynamoDB\.  
AWS IoT Greengrass doesn't use the Lambda execution role that's specified in AWS Lambda for the cloud version of the function\.

E \- Group CA  
A root CA certificate used by AWS IoT Greengrass devices to validate the certificate presented by an AWS IoT Greengrass core device during Transport Layer Security \(TLS\) mutual authentication\.

## Configuring Greengrass Security<a name="gg-config-sec"></a>

To configure your Greengrass application's security:

1. Create an AWS IoT thing for your AWS IoT Greengrass core device\.

1. Generate a key pair and device certificate for your AWS IoT Greengrass core device\.

1. Create and attach an [AWS IoT policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) to the device certificate\. The certificate and policy allow the AWS IoT Greengrass core device access to AWS IoT and AWS IoT Greengrass services\. For more information, see [Minimal AWS IoT Policy for the Core Device](#gg-config-sec-min-iot-policy)\.
**Note**  
The use of [thing policy variables](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html) in the AWS IoT policy for a core device is not supported\. The core uses the same device certificate to make [multiple connections](gg-core.md#connection-client-id) to AWS IoT Core but the client ID in a connection might not be an exact match of the core thing name\.

1. Create a Greengrass service role\. This IAM role authorizes AWS IoT Greengrass to access resources from other AWS services on your behalf\. This allows AWS IoT Greengrass to perform essential tasks, such as retrieving AWS Lambda functions and managing AWS IoT shadows\. You can use the same service role across AWS Regions, but it must be associated with every AWS Region where you use AWS IoT Greengrass\. For more information, see [Greengrass Service Role](service-role.md)\.

1. \(Optional\) Create a Greengrass group role\. This IAM role grants permission to Lambda functions and connectors running on an AWS IoT Greengrass core to call AWS services\. For example, the [Kinesis Firehose connector](kinesis-firehose-connector.md) requires permission to write records to an Amazon Kinesis Data Firehose delivery stream\. You create a separate group role for each Greengrass group\.

1. Create an AWS IoT thing for each device that will connect to your AWS IoT Greengrass core\.

1. Create device certificates, key pairs, and AWS IoT policies for each device that connects to your AWS IoT Greengrass core\.

**Note**  
You can also use existing AWS IoT things and certificates\.

### Minimal AWS IoT Policy for the AWS IoT Greengrass Core Device<a name="gg-config-sec-min-iot-policy"></a>

An AWS IoT policy defines the set of actions allowed for an AWS IoT thing \(in this case, the Greengrass core device\)\. The policy is attached to the device certificate and used to access AWS IoT and AWS IoT Greengrass services\. AWS IoT policies are JSON documents that follow the conventions of IAM policies\. For more information, see [AWS IoT Policies](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) in the *AWS IoT Developer Guide*\.

The following example policy includes the minimum set of actions required to support basic Greengrass functionality for your core device\. Make a note of the following:
+ The policy lists the MQTT topics and topic filters that the core device can publish messages to, subscribe to, and receive messages on, including topics used for shadow state\. To support message exchange between AWS IoT, Lambda functions, connectors, and devices in the Greengrass group, specify the topics and topic filters that you want to allow\. For more information, see [Publish/Subscribe Policy Examples](https://docs.aws.amazon.com/iot/latest/developerguide/pub-sub-policy.html) in the *AWS IoT Developer Guide*\.
+ The policy includes a section that allows AWS IoT to get, update, and delete the core device's shadow\. To allow shadow sync for other devices in the Greengrass group, specify the target ARNs in the `Resource` list \(for example, `arn:aws:iot:region:account-id:thing/device-name`\)\.
+ For the `greengrass:UpdateCoreDeploymentStatus` permission, the final segment in the `Resource` ARN is the URL\-encoded ARN of the core device\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iot:Connect"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iot:Publish",
                "iot:Subscribe"
            ],
            "Resource": [
                "arn:aws:iot:region:account-id:topic/$aws/things/core-name-*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iot:Receive"
            ],
            "Resource": [
                "arn:aws:iot:region:account-id:topicfilter/$aws/things/core-name-*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iot:GetThingShadow",
                "iot:UpdateThingShadow",
                "iot:DeleteThingShadow"
            ],
            "Resource": [
                "arn:aws:iot:region:account-id:thing/core-name-*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "greengrass:AssumeRoleForGroup",
                "greengrass:CreateCertificate"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "greengrass:GetDeployment"
            ],
            "Resource": [
                "arn:aws:greengrass:region:account-id:/greengrass/groups/group-id/deployments/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "greengrass:GetDeploymentArtifacts"
            ],
            "Resource": [
                "arn:aws:greengrass:region:account-id:/greengrass/groups/group-id/deployments/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "greengrass:UpdateCoreDeploymentStatus"
            ],
            "Resource": [
                "arn:aws:greengrass:region:account-id:/greengrass/groups/group-id/deployments/*/cores/arn%3Aaws%3Aiot%3Aregion%3Aaccount-id%3Athing%2Fcore-name"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "greengrass:GetConnectivityInfo",
                "greengrass:UpdateConnectivityInfo"
            ],
            "Resource": [
                "arn:aws:iot:region:account-id:thing/core-name"
            ]
        }
    ]
}
```

On the AWS IoT console, you can easily view and edit the policy that's attached to your core's certificate\.

1. In the navigation pane, choose **Manage**, choose **Things**, and then choose your core\.

1. On your core's configuration page, choose **Security**\.

1. On the **Certificates** page, choose your certificate\.

1. On the certificate's configuration page, choose **Policies**, and then choose the policy\.

   If you want to edit the policy, choose **Edit policy document**\.

## AWS IoT Greengrass Core Security Principals<a name="gg-principals"></a>

The AWS IoT Greengrass core uses the following security principals: AWS IoT client, local MQTT server, and local secrets manager\. The configuration for these principals is stored in the `crypto` object in the `config.json` configuration file\. For more information, see [AWS IoT Greengrass Core Configuration File](gg-core.md#config-json)\.

This configuration includes the path to the private key used by the principal component for authentication and encryption\. AWS IoT Greengrass supports two modes of private key storage: hardware\-based or file system\-based \(default\)\. For more information about storing keys on hardware security modules, see [Hardware Security Integration](hardware-security.md)\.

**AWS IoT Client**  
The AWS IoT client manages communication over the internet between the Greengrass core and AWS IoT\. AWS IoT Greengrass uses X\.509 certificates with public and private keys for mutual authentication when establishing TLS connections for this communication\. For more information, see [X\.509 Certificates and AWS IoT](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html) in the *AWS IoT Developer Guide*\.  
The IoT client supports RSA and EC certificates and keys\. The certificate and private key path are specified for the `IoTCertificate` principal in `config.json`\.

**MQTT Server**  
The local MQTT server manages communication over the local network between the Greengrass core and other Greengrass devices in the group\. AWS IoT Greengrass uses X\.509 certificates with public and private keys for mutual authentication when establishing TLS connections for this communication\.  
By default, AWS IoT Greengrass generates an RSA private key for you\. To configure the core to use a different private key, you must provide the key path for the `MQTTServerCertificate` principal in `config.json`\.    
**Private Key Support**    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-sec.html)
The configuration of the private key determines related processes\. For the list of cipher suites that the Greengrass core supports as a server, see [TLS Cipher Suites Support](#gg-cipher-suites)\.    
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
The local secrets manager securely manages local copies of secrets that you create in AWS Secrets Manager\. It uses a private key to secure the data key that's used to encrypt the secrets\. For more information, see [Deploy Secrets to the AWS IoT Greengrass Core](secrets.md)\.  
By default, the IoT client private key is used, but you can specify a different private key for the `SecretsManager` principal in `config.json`\. Only the RSA key type is supported\. For more information, see [Specify the Private Key for Secret Encryption](secrets.md#secrets-config-private-key)\.  
Currently, AWS IoT Greengrass supports only the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism for encryption and decryption of local secrets when using hardware\-based private keys\. If you're following vendor\-provided instructions to manually generate hardware\-based private keys, make sure to choose PKCS\#1 v1\.5\. AWS IoT Greengrass doesn't support Optimal Asymmetric Encryption Padding \(OAEP\)\.  
**Private Key Support**    
<a name="secrets-manager-private-key-support"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-sec.html)

## Device Connection Workflow<a name="gg-sec-connection"></a>

This section describes how devices connect to the AWS IoT Greengrass cloud service and AWS IoT Greengrass core devices\.
+ An AWS IoT Greengrass core device uses its device certificate, private key, and the AWS IoT root CA certificate to connect to the Greengrass cloud service \.
+ The AWS IoT Greengrass core device downloads group membership information from the Greengrass service\.
+ When a deployment is made to the AWS IoT Greengrass core device, the Device Certificate Manager \(DCM\) handles certificate management for the AWS IoT Greengrass core device\.
+ An AWS IoT device connects to the Greengrass cloud service using its device certificate, private key, and the AWS IoT root CA\. After making the connection, the AWS IoT device uses the Greengrass Discovery Service to find the IP address of its AWS IoT Greengrass core device\. The device can also download the group's root CA certificate, which can be used to authenticate the Greengrass core device\.
+ An AWS IoT device attempts to connect to the AWS IoT Greengrass core, passing its device certificate and client ID\. If the client ID matches the thing name of the device and the certificate is valid, the connection is made\. Otherwise, the connection is terminated\. 

## Greengrass Messaging Workflow<a name="gg-msg-workflow"></a>

AWS IoT Greengrass uses a subscription table to define how MQTT messages can be exchanged between devices, functions, and connectors in a Greengrass group, and with AWS IoT or the local shadow service\. Each subscription specifies a source, target, and MQTT topic \(or subject\) over which messages are sent or received\. AWS IoT Greengrass allows messages to be sent from a source to a target only if a corresponding subscription is defined\.

A subscription defines the message flow in one direction only, from the source to the target\. To support two\-way message exchange, you must create two subscriptions, one for each direction\.

## Certificate Rotation on the MQTT Core Server<a name="gg-cert-expire"></a>

 Greengrass devices use the MQTT core server certificate to authenticate with the Greengrass core device\.   By default, this certificate expires in seven days\. Its lifetime is limited for security reasons, so that if unauthorized parties obtain the certificate, it cannot be used for long\.  

 For certificate rotation to occur, your Greengrass\-enabled device must be online and able to access the Greengrass cloud service directly on a regular basis\. When the certificate expires, the Greengrass core device attempts to connect to the Greengrass cloud service to obtain a new certificate\. If the connection is successful, the core device downloads a new MQTT core server certificate and restarts the local MQTT service\. At this point, all Greengrass devices connected to the core are disconnected\. If the device is offline at the time of expiry, it does not receive the replacement certificate\. Any new attempts to connect to the core device are rejected\. Existing connections are not affected\. Devices cannot connect to the core until the connection to the Greengrass cloud service is restored and a new MQTT core server certificate can be downloaded\. 

You can set the expiration to any value between 7 and 30 days, depending on your needs\. More frequent rotation requires more frequent cloud connection\. Less frequent rotation can pose security concerns\. If you want to set the certificate expiration to a value higher than 30 days, contact AWS Support\. 

 When the MQTT core server certificate expires, any attempt to validate the certificate fails\. The device must be able to detect the failure and terminate the connection\. 

## TLS Cipher Suites Support<a name="gg-cipher-suites"></a>

AWS IoT Greengrass uses the AWS IoT transport security model to encrypt communication with the cloud by using [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) [cipher suites](https://en.wikipedia.org/wiki/Cipher_suite)\. In addition, AWS IoT Greengrass data is encrypted when at rest \(in the cloud\)\. For more information about AWS IoT transport security and supported cipher suites, see [ Transport Security](https://docs.aws.amazon.com/iot/latest/developerguide/transport-security.html) in the *AWS IoT Developer Guide*\.

**Supported Cipher Suites for Local Network Communication**

As opposed to the AWS IoT cloud, the AWS IoT Greengrass core supports the following *local network* TLS cipher suites for certificate\-signing algorithms\. All of these cipher suites are supported when private keys are stored on the file system\. A subset are supported when the core is configured to use hardware security modules \(HSM\)\. For more information, see [AWS IoT Greengrass Core Security Principals](#gg-principals) and [Hardware Security Integration](hardware-security.md)\. The table also includes the minimum version of AWS IoT Greengrass Core software required for support\.

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-sec.html)