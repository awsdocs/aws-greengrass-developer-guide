# AWS Greengrass Security<a name="gg-sec"></a>

AWS Greengrass uses X\.509 certificates, managed subscriptions, AWS IoT policies, and IAM policies and roles to ensure your Greengrass applications are secure\. AWS Greengrass core devices require an AWS IoT thing, a device certificate, and an AWS IoT policy to communicate with the Greengrass cloud service\.

This allows AWS Greengrass core devices to securely connect to the AWS IoT cloud services\. It also allows the Greengrass cloud service to deploy configuration information, Lambda functions, and managed subscriptions to AWS Greengrass core devices\.

AWS IoT devices require an AWS IoT thing, a device certificate, and an AWS IoT policy to connect to the Greengrass service\. This allows AWS IoT devices to use the Greengrass Discovery Service to find and connect to an AWS Greengrass core device\. AWS IoT devices use the same device certificate used to connect to AWS IoT device gateway and AWS Greengrass core devices\. The following diagram shows the components of the AWS Greengrass security model:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-security.png)

A \- Greengrass service role  
A customer\-created IAM role that allows AWS Greengrass access to your AWS IoT and Lambda resources\.

B \- Core device certificate  
An X\.509 certificate used to authenticate an AWS Greengrass core\.

C \- Device certificate  
An X\.509 certificate used to authenticate an AWS IoT device\.

D \- Group role  
A role assumed by AWS Greengrass when calling into the cloud from a Lambda function on an AWS Greengrass core\.

E \- Group CA  
A root CA certificate used by AWS Greengrass devices to validate the certificate presented by an AWS Greengrass core device during TLS mutual authentication\.

## Configuring Greengrass Security<a name="gg-config-sec"></a>

To configure your Greengrass application's security:

1. Create an AWS IoT thing for your AWS Greengrass core device\.

1. Generate a key pair and device certificate for your AWS Greengrass core device\.

1. Create and attach an AWS IoT policy to the device certificate\. The certificate and policy allow the AWS Greengrass core device access to AWS IoT and Greengrass cloud services\.

1. Create a Greengrass service role\. This IAM role grants AWS Greengrass access to your Greengrass and AWS IoT resources\. You only need to create a service role once per AWS account\.

1. \(Optional\) Create a Greengrass group role\. This role grants permission to Lambda functions running on an AWS Greengrass core to call other AWS services \(in the cloud\)\. You need to do this for each Greengrass group you create\.

1. Create an AWS IoT thing for each device that will connect to your AWS Greengrass core\.

1. Create device certificates, key pairs, and AWS IoT policies for each device that will connect to your AWS Greengrass core\.

**Note**  
You can also use existing AWS IoT things and certificates\.

## Device Connection Workflow<a name="gg-sec-connection"></a>

This section describes how devices connect to the AWS Greengrass cloud service and AWS Greengrass core devices\.
+ An AWS Greengrass core device uses its device certificate, private key, and the AWS IoT root CA certificate to connect to the Greengrass cloud service \.
+ The AWS Greengrass core device downloads group membership information from the Greengrass service\.
+ When a deployment is made to the AWS Greengrass core device, the Device Certificate Manager \(DCM\) handles certificate management for the AWS Greengrass core device\.
+ An AWS IoT device connects to the Greengrass cloud service using its device certificate, private key, and the AWS IoT root CA\. After making the connection, the AWS IoT device uses the Greengrass Discovery Service to find the IP address of its AWS Greengrass core device\. The device can also download the group's root CA certificate, which can be used to authenticate the Greengrass core device\.
+ An AWS IoT device attempts to connect to the AWS Greengrass core, passing its device certificate and client ID\. If the client ID matches the thing name of the device and the certificate is valid, the connection is made\. Otherwise, the connection is terminated\. 

## Greengrass Messaging Workflow<a name="gg-msg-workflow"></a>

 A subscription table is used to define how messages are exchanged within a Greengrass group \(between AWS Greengrass core devices, AWS IoT devices, and Lambda functions\)\. Each entry in the subscription table specifies a source, a destination, and an MQTT topic over which messages are sent/received\. Messages can be exchanged only if an entry exists in the subscription table specifying the source \(message sender\), the target \(message recipient\), and the MQTT topic\. Subscription table entries specify passing messages in one direction, from the source to the target\. If you want two\-way message passing, create two subscription table entries, one for each direction\. 

## MQTT Core Server Certificate Rotation<a name="gg-cert-expire"></a>

The MQTT core server certificate expires, by default, in 7 days\. You can set the expiration to any value between 7 and 30 days\. When the MQTT core server certificate expires, any attempt to validate the certificate fails\. The device must be able to detect the failure and terminate the connection\. Existing connections are not affected\. When the certificate expires, the AWS Greengrass core device attempts to connect to the Greengrass cloud service to obtain a new certificate\. If the connection is successful, the AWS Greengrass core device downloads a new MQTT core server certificate and restarts the local MQTT service\. At this point, all AWS IoT devices connected to the core are disconnected\.

If there is no internet connection when the AWS Greengrass core attempts to get a new MQTT core server certificate, AWS IoT devices are unable to connect to the AWS Greengrass core until the connection to the Greengrass cloud service is restored and a new MQTT core server certificate can be downloaded\.

When AWS IoT devices are disconnected from a core, they have to wait a short period of time and then attempt to reconnect to the AWS Greengrass core device\. 

## AWS Greengrass Cipher Suites<a name="gg-cipher-suites"></a>

AWS Greengrass uses the AWS IoT transport security model to encrypt communication with the cloud by using [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) [cipher suites](https://en.wikipedia.org/wiki/Cipher_suite)\. In addition, AWS Greengrass data is encrypted when at rest \(in the cloud\)\. For more information about AWS IoT transport security and supported cipher suites, see [ Transport Security](https://docs.aws.amazon.com/iot/latest/developerguide/iot-security-identity.html#transport-security) in the *AWS IoT Developer Guide*\.

As opposed to the AWS IoT cloud, the AWS Greengrass core supports the following *local network* TLS cipher suites:

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-sec.html)