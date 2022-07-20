--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Create client devices in an AWS IoT Greengrass group<a name="device-group"></a>

In this step, you add two client devices to your Greengrass group\. This process includes registering the devices as AWS IoT things and configuring certificates and keys to allow them to connect to AWS IoT Greengrass\.

1. <a name="console-gg-groups"></a>In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="gg-group-add-device"></a>On the group configuration page, choose **Client devices**, and then choose **Associate**\.

1. <a name="gg-group-create-device"></a>In the **Associate a client device with this group** modal, choose **Create new AWS IoT thing**\.

   The **Create things** page opens in a new tab\.

1. <a name="gg-group-create-single-thing"></a>On the **Create things** page, choose **Create single thing**, and then choose **Next**\.

1. On the **Specify thing properties** page, register this client device as **HelloWorld\_Publisher**, and then choose **Next**\.

1. <a name="gg-group-create-device-configure-certificate"></a>On the **Configure device certificate** page, choose **Next**\.

1. <a name="gg-group-create-device-attach-policy"></a>On the **Attach policies to certificate** page, do one of the following:
   + Select an existing policy that grants permissions that client devices require, and then choose **Create thing**\.

     A modal opens where you can download the certificates and keys that the device uses to connect to the AWS Cloud and the core\.
   + Create and attach a new policy that grants client device permissions\. Do the following:

     1. Choose **Create policy**\.

        The **Create policy** page opens in a new tab\.

     1. On the **Create policy** page, do the following:

        1. For **Policy name**, enter a name that describes the policy, such as **GreengrassV1ClientDevicePolicy**\.

        1. On the **Policy statements** tab, under **Policy document**, choose **JSON**\.

        1. Enter the following policy document\. This policy allows the client device to discover Greengrass cores and communicate on all MQTT topics\. For information about how to restrict this policy's access, see [Device authentication and authorization for AWS IoT Greengrass](device-auth.md)\.

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

        1. In the **Policies** list, select the policy that you created, such as **GreengrassV1ClientDevicePolicy**\.

           If you don't see the policy, choose the refresh button\.

        1. Choose **Create thing**\.

           A modal opens where you can download the certificates and keys that the device uses to connect to the AWS Cloud and the core\.

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

1. Return to the browser tab with the **Associate a client device with this group** modal open\. Do the following:

   1. For **AWS IoT thing name**, choose the **HelloWorld\_Publisher** thing that you created\.

      If you don't see the thing, choose the refresh button\.

   1. Choose **Associate**\.

1. Repeat steps 3 \- 10 to add a second client device to the group\.

   Name this client device **HelloWorld\_Subscriber**\. Download the certificates and keys for this client device to your computer\. Again, make a note of the certificate's ID that's common in the file names for the HelloWorld\_Subscriber device\.

   You should now have two client devices in your Greengrass group:
   + HelloWorld\_Publisher
   + HelloWorld\_Subscriber

1. Create a folder on your computer for these client devices' security credentials\. Copy the certificates and keys into this folder\.