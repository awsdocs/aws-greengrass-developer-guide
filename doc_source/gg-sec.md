# AWS IoT Greengrass Security<a name="gg-sec"></a>

AWS IoT Greengrass uses X\.509 certificates, managed subscriptions, AWS IoT policies, and IAM policies and roles to ensure your Greengrass applications are secure\. AWS IoT Greengrass core devices require an AWS IoT thing, a device certificate, and an AWS IoT policy to communicate with the Greengrass cloud service\.

This allows AWS IoT Greengrass core devices to securely connect to the AWS IoT cloud services\. It also allows the Greengrass cloud service to deploy configuration information, Lambda functions, connectors, and managed subscriptions to AWS IoT Greengrass core devices\.

AWS IoT devices require an AWS IoT thing, a device certificate, and an AWS IoT policy to connect to the Greengrass service\. This allows AWS IoT devices to use the Greengrass Discovery Service to find and connect to an AWS IoT Greengrass core device\. AWS IoT devices use the same device certificate used to connect to AWS IoT device gateway and AWS IoT Greengrass core devices\. The following diagram shows the components of the AWS IoT Greengrass security model:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-security.png)

A \- Greengrass service role  
A customer\-created IAM role that allows AWS IoT Greengrass access to your AWS IoT, Lambda, and other AWS resources\.

B \- Core device certificate  
An X\.509 certificate used to authenticate an AWS IoT Greengrass core\.

C \- Device certificate  
An X\.509 certificate used to authenticate an AWS IoT device\.

D \- Group role  
A role assumed by AWS IoT Greengrass when calling AWS services from a Lambda function or connector on an AWS IoT Greengrass core\.  
This the Lambda execution role for all the Greengrass Lambda functions and connectors that run on the core\. Use this role to specify access permissions that your user\-defined Lambda functions and connectors need to access AWS services, such as DynamoDB\.  
AWS IoT Greengrass doesn't use the Lambda execution role that's specified in AWS Lambda for the cloud version of the function\.

E \- Group CA  
A root CA certificate used by AWS IoT Greengrass devices to validate the certificate presented by an AWS IoT Greengrass core device during TLS mutual authentication\.

## Configuring Greengrass Security<a name="gg-config-sec"></a>

To configure your Greengrass application's security:

1. Create an AWS IoT thing for your AWS IoT Greengrass core device\.

1. Generate a key pair and device certificate for your AWS IoT Greengrass core device\.

1. Create and attach an AWS IoT policy to the device certificate\. The certificate and policy allow the AWS IoT Greengrass core device access to AWS IoT and Greengrass cloud services\. For more information, see [Minimal AWS IoT Policy for the AWS IoT Greengrass Core Device](#gg-config-sec-min-iot-policy)\.

1. Create a Greengrass service role\. This IAM role authorizes AWS IoT Greengrass to access resources from other AWS services on your behalf\. This allows AWS IoT Greengrass to perform essential tasks, such as retrieving AWS Lambda functions and managing AWS IoT shadows\. You only need to create a service role once per AWS account\.

1. \(Optional\) Create a Greengrass group role\. This role grants permission to Lambda functions and connectors running on an AWS IoT Greengrass core to call other AWS services \(in the cloud\)\. You need to do this for each Greengrass group you create\.

1. Create an AWS IoT thing for each device that will connect to your AWS IoT Greengrass core\.

1. Create device certificates, key pairs, and AWS IoT policies for each device that will connect to your AWS IoT Greengrass core\.

**Note**  
You can also use existing AWS IoT things and certificates\.

### Minimal AWS IoT Policy for the AWS IoT Greengrass Core Device<a name="gg-config-sec-min-iot-policy"></a>

An AWS IoT policy defines the set of actions allowed for an AWS IoT thing—in this case, the Greengrass core device\. The policy is attached to the device certificate and used to access AWS IoT and Greengrass cloud services\. AWS IoT policies are JSON documents that follow the conventions of IAM policies\. For more information, see [AWS IoT Policies](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) in the *AWS IoT Developer Guide*\.

The following example policy includes the minimum set of actions required to support basic Greengrass functionality for your core device\. In the example, note the following:
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
        "iot:Subscribe",
        "iot:Receive"
      ],
      "Resource": [
        "arn:aws:iot:region:account-id:topicfilter/$aws/things/core-name-*",
        "arn:aws:iot:region:account-id:topic/$aws/things/core-name-*"
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

## Device Connection Workflow<a name="gg-sec-connection"></a>

This section describes how devices connect to the AWS IoT Greengrass cloud service and AWS IoT Greengrass core devices\.
+ An AWS IoT Greengrass core device uses its device certificate, private key, and the AWS IoT root CA certificate to connect to the Greengrass cloud service \.
+ The AWS IoT Greengrass core device downloads group membership information from the Greengrass service\.
+ When a deployment is made to the AWS IoT Greengrass core device, the Device Certificate Manager \(DCM\) handles certificate management for the AWS IoT Greengrass core device\.
+ An AWS IoT device connects to the Greengrass cloud service using its device certificate, private key, and the AWS IoT root CA\. After making the connection, the AWS IoT device uses the Greengrass Discovery Service to find the IP address of its AWS IoT Greengrass core device\. The device can also download the group's root CA certificate, which can be used to authenticate the Greengrass core device\.
+ An AWS IoT device attempts to connect to the AWS IoT Greengrass core, passing its device certificate and client ID\. If the client ID matches the thing name of the device and the certificate is valid, the connection is made\. Otherwise, the connection is terminated\. 

## Greengrass Messaging Workflow<a name="gg-msg-workflow"></a>

 A subscription table is used to define how messages are exchanged within a Greengrass group \(between AWS IoT Greengrass core devices, AWS IoT devices, Lambda functions, and connectors\)\. Each entry in the subscription table specifies a source, a destination, and an MQTT topic over which messages are sent or received\. Messages can be exchanged only if an entry exists in the subscription table specifying the source \(message sender\), the target \(message recipient\), and the MQTT topic\. Subscription table entries specify passing messages in one direction, from the source to the target\. If you want two\-way message passing, create two subscription table entries, one for each direction\. 

## MQTT Core Server Certificate Rotation<a name="gg-cert-expire"></a>

The MQTT core server certificate expires, by default, in 7 days\. You can set the expiration to any value between 7 and 30 days\. When the MQTT core server certificate expires, any attempt to validate the certificate fails\. The device must be able to detect the failure and terminate the connection\. Existing connections are not affected\. When the certificate expires, the AWS IoT Greengrass core device attempts to connect to the Greengrass cloud service to obtain a new certificate\. If the connection is successful, the AWS IoT Greengrass core device downloads a new MQTT core server certificate and restarts the local MQTT service\. At this point, all AWS IoT devices connected to the core are disconnected\.

If there is no internet connection when the AWS IoT Greengrass core attempts to get a new MQTT core server certificate, AWS IoT devices are unable to connect to the AWS IoT Greengrass core until the connection to the Greengrass cloud service is restored and a new MQTT core server certificate can be downloaded\.

When AWS IoT devices are disconnected from a core, they have to wait a short period of time and then attempt to reconnect to the AWS IoT Greengrass core device\. 

## AWS IoT Greengrass Cipher Suites<a name="gg-cipher-suites"></a>

AWS IoT Greengrass uses the AWS IoT transport security model to encrypt communication with the cloud by using [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) [cipher suites](https://en.wikipedia.org/wiki/Cipher_suite)\. In addition, AWS IoT Greengrass data is encrypted when at rest \(in the cloud\)\. For more information about AWS IoT transport security and supported cipher suites, see [ Transport Security](https://docs.aws.amazon.com/iot/latest/developerguide/iot-security-identity.html#transport-security) in the *AWS IoT Developer Guide*\.

As opposed to the AWS IoT cloud, the AWS IoT Greengrass core supports the following *local network* TLS cipher suites:

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-sec.html)

**Note**  
Only a subset of cipher suites are supported when hardware security is configured\. For more information, see [Supported Cipher Suites for Hardware Security Integration](hardware-security.md#cipher-suites-for-hsm)\.