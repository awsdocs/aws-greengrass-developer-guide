# Security best practices for AWS IoT Greengrass<a name="security-best-practices"></a>

This topic contains security best practices for AWS IoT Greengrass\.

## Grant minimum possible permissions<a name="least-privilege"></a>

Follow the principle of least privilege by using the minimum set of permissions in IAM roles\. Limit the use of the `*` wildcard for the `Action` and `Resource` properties in your IAM policies\. Instead, declare a finite set of actions and resources when possible\. For more information about least privilege and other policy best practices, see [Policy best practices](security_iam_id-based-policy-examples.md#security_iam_service-with-iam-policy-best-practices)\.

The least privilege best practice also applies to AWS IoT policies you attach to your Greengrass core and connected devices\.

## Don't hardcode credentials in Lambda functions<a name="no-hardcoded-credentials"></a>

Don't hardcode credentials in your user\-defined Lambda functions\. To better protect your credentials:
+ To interact with AWS services, define permissions for specific actions and resources in the [Greengrass group role](group-role.md)\.
+ Use [local secrets](secrets.md) to store your credentials\. Or, if the function uses the AWS SDK, use credentials from the default credential provider chain\.

## Don't log sensitive information<a name="protect-pii"></a>

You should prevent the logging of credentials and other personally identifiable information \(PII\)\. We recommend that you implement the following safeguards even though access to local logs on a core device requires root privileges and access to CloudWatch Logs requires IAM permissions\.
+ Don't use sensitive information in MQTT topic paths\.
+ Don't use sensitive information in device \(thing\) names, types, and attributes in the AWS IoT Core registry\.
+ Don't log sensitive information in your user\-defined Lambda functions\.
+ Don't use sensitive information in the names and IDs of Greengrass resources:
  + Connectors
  + Cores
  + Devices
  + Functions
  + Groups
  + Loggers
  + Resources \(local, machine learning, or secret\)
  + Subscriptions

## Create targeted subscriptions<a name="targeted-subscriptions"></a>

Subscriptions control the information flow in a Greengrass group by defining how messages are exchanged between services, devices, and Lambda functions\. To ensure that an application can do only what it's intended to do, your subscriptions should allow publishers to send messages to specific topics only, and limit subscribers to receive messages only from topics that are required for their functionality\.

## Keep your device clock in sync<a name="device-clock"></a>

It's important to have an accurate time on your device\. X\.509 certificates have an expiry date and time\. The clock on your device is used to verify that a server certificate is still valid\. Device clocks can drift over time or batteries can get discharged\.

For more information, see the [ Keep your device's clock in sync](https://docs.aws.amazon.com/iot/latest/developerguide/security-best-practices.html#device-clock) best practice in the *AWS IoT Core Developer Guide*\.

## Manage device authentication with the Greengrass core<a name="manage-device-authentication-with-core"></a>

<a name="gg-device-discovery"></a>Greengrass devices can run [FreeRTOS](https://docs.aws.amazon.com/freertos/latest/userguide/freertos-lib-gg-connectivity.html) or use the [AWS IoT Device SDK](what-is-gg.md#iot-device-sdk) or [ AWS IoT Greengrass Discovery API](gg-discover-api.md) to get discovery information used to connect and authenticate with the core in the same Greengrass group\. Discovery information includes:
+ Connectivity information for the Greengrass core that's in the same Greengrass group as the device\. This information includes the host address and port number of each endpoint for the core device\.
+ The group CA certificate used to sign the local MQTT server certificate\. Devices use the group CA certificate to validate the MQTT server certificate presented by the core\.

The following are best practices for connected devices to manage mutual authentication with a Greengrass core\. These practices can help mitigate your risk if your core device is compromised\.

**Validate the local MQTT server certificate for each connection\.**  
Devices should validate the MQTT server certificate presented by the core every time they establish a connection with the core\. This validation is the *connected device* side of the mutual authentication between a core device and connected devices\. Devices must be able to detect a failure and terminate the connection\.

**Do not hardcode discovery information\.**  
Devices should rely on discovery operations to get core connectivity information and the group CA certificate, even if the core uses a static IP address\. Devices should not hardcode this discovery information\.

**Periodically update discovery information\.**  
Devices should periodically run discovery to update core connectivity information and the group CA certificate\. We recommend that devices update this information before they establish a connection with the core\. Because shorter durations between discovery operations can minimize your potential exposure time, we recommend that devices periodically disconnect and reconnect to trigger the update\.

If you lose control of a Greengrass core device and you want to prevent connected devices from transmitting data to the core, do the following:<a name="make-devices-distrust-core"></a>

1. Remove the Greengrass core from the Greengrass group\.

1. Rotate the group CA certificate\. In the AWS IoT console, you can rotate the CA certificate on the group's **Settings** page\. In the AWS IoT Greengrass API, you can use the [CreateGroupCertificateAuthority](https://docs.aws.amazon.com/greengrass/latest/apireference/creategroupcertificateauthority-post.html) action\.

   We also recommend using full disk encryption if the hard drive of your core device is vulnerable to theft\.

For more information, see [Device authentication and authorization for AWS IoT Greengrass](device-auth.md)\.

## See also<a name="security-best-practices-see-also"></a>
+ [Security best practices in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/security-best-practices.html) in the *AWS IoT Developer Guide*
+ [ Ten security golden rules for IoT solutions](https://aws.amazon.com/blogs/iot/ten-security-golden-rules-for-iot-solutions/) on the *Internet of Things on AWS Official Blog*