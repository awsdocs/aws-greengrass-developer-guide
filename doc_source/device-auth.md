# Device authentication and authorization for AWS IoT Greengrass<a name="device-auth"></a>

Devices in AWS IoT Greengrass environments use X\.509 certificates for authentication and AWS IoT policies for authorization\. Certificates and policies allow devices to securely connect with each other, AWS IoT Core, and AWS IoT Greengrass\.

X\.509 certificates are digital certificates that use the X\.509 public key infrastructure standard to associate a public key with the identity contained in a certificate\. X\.509 certificates are issued by a trusted entity called a certificate authority \(CA\)\. The CA maintains one or more special certificates called CA certificates that it uses to issue X\.509 certificates\. Only the certificate authority has access to CA certificates\.

AWS IoT policies define the set of operations allowed for AWS IoT devices\. Specifically, they allow and deny access to AWS IoT Core and AWS IoT Greengrass data plane operations, such as publishing MQTT messages and retrieving device shadows\.

All devices require an entry in the AWS IoT Core registry and an activated X\.509 certificate with an attached AWS IoT policy\. Devices fall into two categories:
+ **Greengrass cores**\. Greengrass core devices use certificates and AWS IoT policies to connect to AWS IoT Core\. The certificates and policies also allow AWS IoT Greengrass to deploy configuration information, Lambda functions, connectors, and managed subscriptions to core devices\.
+ **Devices connected to a Greengrass core**\. These connected devices \(also called *Greengrass devices*\) use certificates and policies to connect to AWS IoT Core and the AWS IoT Greengrass service\. This allows devices to use the AWS IoT Greengrass Discovery Service to find and connect to a core device\. A Greengrass device uses the same certificate to connect to the AWS IoT Core device gateway and core device\. Devices also use discovery information for mutual authentication with the core device\. For more information, see [Device connection workflow](gg-sec.md#gg-sec-connection) and [Manage device authentication with the Greengrass core](security-best-practices.md#manage-device-authentication-with-core)\.

## X\.509 certificates<a name="x509-certificates"></a>

Communication between core and connected devices and between devices and AWS IoT Core or AWS IoT Greengrass must be authenticated\. This mutual authentication is based on registered X\.509 device certificates and cryptographic keys\.

In an AWS IoT Greengrass environment, devices use certificates with public and private keys for the following Transport Layer Security \(TLS\) connections:
+ The AWS IoT client component on the Greengrass core connecting to AWS IoT Core and AWS IoT Greengrass over the internet\.
+ Greengrass connected devices connecting to AWS IoT Greengrass to get core discovery information over the internet\.
+ The MQTT server component on the Greengrass core connecting to Greengrass devices in the group over the local network\.

The AWS IoT Greengrass core device stores certificates in two locations:<a name="ggc-certificate-locations"></a>
+ Core device certificate in `/greengrass-root/certs`\. Typically, the core device certificate is named `hash.cert.pem` \(for example, `86c84488a5.cert.pem`\)\. This certificate is used by the AWS IoT client for mutual authentication when the core connects to the AWS IoT Core and AWS IoT Greengrass services\.
+ MQTT server certificate in `/greengrass-root/ggc/var/state/server`\. The MQTT server certificate is named `server.crt`\. This certificate is used for mutual authentication between the local MQTT server \(on the Greengrass core\) and Greengrass devices\.
**Note**  
*greengrass\-root* represents the path where the AWS IoT Greengrass Core software is installed on your device\. Typically, this is the `/greengrass` directory\.

For more information, see [AWS IoT Greengrass core security principals](gg-sec.md#gg-principals)\.

### Certificate authority \(CA\) certificates<a name="ca-certificates"></a>

Core devices and Greengrass connected devices download a root CA certificate used for authentication with AWS IoT Core and AWS IoT Greengrass services\. We recommend that you use an Amazon Trust Services \(ATS\) root CA certificate, such as [Amazon Root CA 1](https://www.amazontrust.com/repository/AmazonRootCA1.pem)\. For more information, see [CA certificates for server authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html#server-authentication-certs) in the *AWS IoT Core Developer Guide*\.

**Note**  
Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a VeriSign root CA certificate with a legacy endpoint\. Only some AWS Regions support legacy endpoints\. For more information, see [Service endpoints must match the root CA certificate type](gg-core.md#certificate-endpoints)\.

Greengrass connected devices also download the Greengrass group CA certificate\. This is used to validate the MQTT server certificate on the Greengrass core during mutual authentication\. For more information, see [Device connection workflow](gg-sec.md#gg-sec-connection)\. The default expiration of the MQTT server certificate is seven days\.

### Certificate rotation on the local MQTT server<a name="gg-cert-expire"></a>

 Greengrass connected devices use the local MQTT server certificate for mutual authentication with the Greengrass core device\. By default, this certificate expires in seven days\. This limited period is based on security best practices\. The MQTT server certificate is signed by the group CA certificate, which is stored in the cloud\. 

 For certificate rotation to occur, your Greengrass connected device must be online and able to access the AWS IoT Greengrass service directly on a regular basis\. When the certificate expires, the Greengrass core device attempts to connect to the AWS IoT Greengrass service to obtain a new certificate\. If the connection is successful, the core device downloads a new MQTT server certificate and restarts the local MQTT service\. At this point, all Greengrass devices connected to the core are disconnected\. If the device is offline at the time of expiry, it does not receive the replacement certificate\. Any new attempts to connect to the core device are rejected\. Existing connections are not affected\. Devices cannot connect to the core until the connection to the AWS IoT Greengrass service is restored and a new MQTT server certificate can be downloaded\. 

You can set the expiration to any value between 7 and 30 days, depending on your needs\. More frequent rotation requires more frequent cloud connection\. Less frequent rotation can pose security concerns\. If you want to set the certificate expiration to a value higher than 30 days, contact AWS Support\. 

In the AWS IoT console, you can manage the certificate on the group's **Settings** page\. In the AWS IoT Greengrass API, you can use the [UpdateGroupCertificateConfiguration](https://docs.aws.amazon.com/greengrass/latest/apireference/updategroupcertificateconfiguration-put.html) action\.

 When the MQTT server certificate expires, any attempt to validate the certificate fails\. The device must be able to detect the failure and terminate the connection\. 

## AWS IoT policies for data plane operations<a name="iot-policies"></a>

Use AWS IoT policies to authorize access to the AWS IoT Core and AWS IoT Greengrass data plane\. The AWS IoT Core data plane consists of operations for devices, users, and applications, such as connecting to AWS IoT Core and subscribing to topics\. The AWS IoT Greengrass data plane consists of operations for Greengrass devices, such as retrieving deployments and updating connectivity information\.

An AWS IoT policy is a JSON document that's similar to an IAM policy\. It contains one or more policy statements that specify the following properties:
+ `Effect`\. The access mode: `Allow` or `Deny`\.
+ `Action`\. The list of actions that are allowed or denied by the policy\.
+ `Resource`\. The list of resources on which the action is allowed or denied\.

For more information, see [AWS IoT policies](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) and [AWS IoT policy actions](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policy-actions.html) in the *AWS IoT Core Developer Guide*\.

 

### AWS IoT Greengrass policy actions<a name="gg-policy-actions"></a>Greengrass Core Actions

AWS IoT Greengrass defines the following policy actions that Greengrass core devices can use in AWS IoT policies:

`greengrass:AssumeRoleForGroup`  
Permission for a Greengrass core device to retrieve credentials using the Token Exchange Service \(TES\) system Lambda function\. The permissions that are tied to the retrieved credentials are based on the policy that's attached to the configured group role\.  
This permission is checked when a Greengrass core device attempts to retrieve credentials \(assuming the credentials are not cached locally\)\.

`greengrass:CreateCertificate`  
Permission for a Greengrass core device to create its own server certificate\.  
This permission is checked when a Greengrass core device creates a certificate\. Greengrass core devices attempt to create a server certificate upon first run, when the core's connectivity information changes, and on designated rotation periods\.

`greengrass:GetConnectivityInfo`  
Permission for a Greengrass core device to retrieve its own connectivity information\.  
This permission is checked when a Greengrass core device attempts to retrieve its connectivity information from AWS IoT Core\.

`greengrass:GetDeployment`  
Permission for a Greengrass core device to retrieve deployments\.  
This permission is checked when a Greengrass core device attempts to retrieve deployments and deployment statuses from the cloud\.

`greengrass:GetDeploymentArtifacts`  
Permission for a Greengrass core device to retrieve deployment artifacts such as group information or Lambda functions\.  
This permission is checked when a Greengrass core device receives a deployment and then attempts to retrieve deployment artifacts\.

`greengrass:UpdateConnectivityInfo`  
Permission for a Greengrass core device to update its own connectivity information with IP or hostname information\.  
This permission is checked when a Greengrass core device attempts to update its connectivity information in the cloud\.

`greengrass:UpdateCoreDeploymentStatus`  
Permission for a Greengrass core device to update the status of a deployment\.  
This permission is checked when a Greengrass core device receives a deployment and then attempts to update the deployment status\.

 Greengrass Device Actions

AWS IoT Greengrass defines the following policy action that Greengrass devices can use in AWS IoT policies:

`greengrass:Discover`  
Permission for a Greengrass device to use the [Discovery API](gg-discover-api.md) to retrieve its group's core connectivity information and group certificate authority\.  
This permission is checked when a Greengrass device calls the Discovery API with TLS mutual authentication\.

## Minimal AWS IoT policy for the AWS IoT Greengrass core device<a name="gg-config-sec-min-iot-policy"></a>

The following example policy includes the minimum set of actions required to support basic Greengrass functionality for your core device\.
+ The policy lists the MQTT topics and topic filters that the core device can publish messages to, subscribe to, and receive messages on, including topics used for shadow state\. To support message exchange between AWS IoT Core, Lambda functions, connectors, and devices in the Greengrass group, specify the topics and topic filters that you want to allow\. For more information, see [Publish/Subscribe policy examples](https://docs.aws.amazon.com/iot/latest/developerguide/pub-sub-policy.html) in the *AWS IoT Core Developer Guide*\.
+ The policy includes a section that allows AWS IoT Core to get, update, and delete the core device's shadow\. To allow shadow sync for connected devices in the Greengrass group, specify the target Amazon Resource Names \(ARNs\) in the `Resource` list \(for example, `arn:aws:iot:region:account-id:thing/device-name`\)\.
+ <a name="thing-policy-variable-not-supported"></a>The use of [thing policy variables](https://docs.aws.amazon.com/iot/latest/developerguide/thing-policy-variables.html) \(`iot:Connection.Thing.*`\) in the AWS IoT policy for a core device is not supported\. The core uses the same device certificate to make [multiple connections](gg-core.md#connection-client-id) to AWS IoT Core but the client ID in a connection might not be an exact match of the core thing name\.
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
                "iot:Receive"
            ],
            "Resource": [
                "arn:aws:iot:region:account-id:topic/$aws/things/core-name-*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iot:Subscribe"
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

**Note**  
AWS IoT policies for connected Greengrass devices typically require similar permissions for `iot:Connect`, `iot:Publish`, `iot:Receive`, and `iot:Subscribe` actions\.  
To allow a device to automatically detect connectivity information for the cores in the Greengrass groups that the device belongs to, the AWS IoT policy for a connected device must include the `greengrass:Discover` action\. In the `Resource` section, specify the ARN of the connected device, not the ARN of the Greengrass core device\. For example:  

```
{
    "Effect": "Allow",
    "Action": [
        "greengrass:Discover"
    ],
    "Resource": [
        "arn:aws:iot:region:account-id:thing/device-name"
    ]
}
```
The AWS IoT policy for connected devices doesn't typically require permissions for `iot:GetThingShadow`, `iot:UpdateThingShadow`, or `iot:DeleteThingShadow` actions, because the Greengrass core handles shadow sync operations for connected devices\. In this case, make sure that the `Resource` section for shadow actions in the core's AWS IoT policy includes the ARNs of the connected devices\.

 

In the AWS IoT console, you can view and edit the policy that's attached to your core's certificate\.

1. In the navigation pane, choose **Manage**, choose **Things**, and then choose your core\.

1. On your core's configuration page, choose **Security**\.

1. On the **Certificates** page, choose your certificate\.

1. On the certificate's configuration page, choose **Policies**, and then choose the policy\.

   If you want to edit the policy, choose **Edit policy document**\.