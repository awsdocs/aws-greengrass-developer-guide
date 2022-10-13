--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Provision an AWS IoT thing to use as a Greengrass core<a name="provision-core"></a>

Greengrass *cores* are devices that run the AWS IoT Greengrass Core software to manage local IoT processes\. To set up a Greengrass core, you create an AWS IoT *thing*, which represents a device or logical entity that connects to AWS IoT\. When you register a device as an AWS IoT thing, that device can use a digital certificate and keys that allow it to access AWS IoT\. You use an [AWS IoT policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) to allow the device to communicate with the AWS IoT and AWS IoT Greengrass services\.

In this section, you register your device as an AWS IoT thing to use it as a Greengrass core\.

**To create an AWS IoT thing**

1. Navigate to the [AWS IoT console](https://console.aws.amazon.com/iot)\.

1. Under **Manage**, expand **All devices**, and then choose **Things**\.

1. On the **Things** page, choose **Create things**\.

1. <a name="gg-group-create-single-thing"></a>On the **Create things** page, choose **Create single thing**, and then choose **Next**\.

1. On the **Specify thing properties** page, do the following:

   1. For **Thing name**, enter a name that represents your device, such as **MyGreengrassV1Core**\.

   1. Choose **Next**\.

1. <a name="gg-group-create-device-configure-certificate"></a>On the **Configure device certificate** page, choose **Next**\.

1. On the **Attach policies to certificate** page, do one of the following:
   + Select an existing policy that grants permissions that cores require, and then choose **Create thing**\.

     A modal opens where you can download the certificates and keys that the device uses to connect to the AWS Cloud\.
   + Create an attach a new policy that grants core device permissions\. Do the following:

     1. Choose **Create policy**\.

        The **Create policy** page opens in a new tab\.

     1. On the **Create policy** page, do the following:

        1. For **Policy name**, enter a name that describes the policy, such as **GreengrassV1CorePolicy**\.

        1. On the **Policy statements** tab, under **Policy document**, choose **JSON**\.

        1. Enter the following policy document\. This policy allows the core to communicate with the AWS IoT Core service, interact with device shadows, and communicate with the AWS IoT Greengrass service\. For information about how to restrict this policy's access based on your use case, see [Minimal AWS IoT policy for the AWS IoT Greengrass core device](device-auth.md#gg-config-sec-min-iot-policy)\.

           ```
           {
             "Version": "2012-10-17",
             "Statement": [
               {
                 "Effect": "Allow",
                 "Action": [
                   "iot:Publish",
                   "iot:Subscribe",
                   "iot:Connect",
                   "iot:Receive"
                 ],
                 "Resource": [
                   "*"
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
                   "*"
                 ]
               },
               {
                 "Effect": "Allow",
                 "Action": [
                   "greengrass:*"
                 ],
                 "Resource": [
                   "*"
                 ]
               }
             ]
           }
           ```

        1. Choose **Create** to create the policy\.

1. Return to the browser tab with the **Attach policies to certificate** page open\. Do the following:

   1. In the **Policies** list, select the policy that you created, such as **GreengrassV1CorePolicy**\.

      If you don't see the policy, choose the refresh button\.

   1. Choose **Create thing**\.

      A modal opens where you can download the certificates and keys that the core uses to connect to AWS IoT\.

1. <a name="gg-group-create-device-download-certs"></a>In the **Download certificates and keys** modal, download the device's certificates\.
**Important**  
Before you choose **Done**, download the security resources\.

   Do the following:

   1. For **Device certificate**, choose **Download** to download the device certificate\.

   1. For **Public key file**, choose **Download** to download the public key for the certificate\.

   1. For **Private key file**, choose **Download** to download the private key file for the certificate\.

   1. Review [Server Authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) in the *AWS IoT Developer Guide* and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\. Under **Root CA certificates**, choose **Download** for a root CA certificate\.

   1. Choose **Done**\.

   Make a note of the certificate ID that's common in the file names for the device certificate and keys\. You need it later\.
