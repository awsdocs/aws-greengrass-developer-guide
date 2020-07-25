# Create AWS IoT devices in an AWS IoT Greengrass group<a name="device-group"></a>

In this step, you add two AWS IoT devices to your Greengrass group\. This process includes registering the devices and configuring certificates and keys to allow them to connect to AWS IoT Greengrass\.

 

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="gg-group-add-device"></a>On the group configuration page, choose **Devices**, and then choose **Add Device**\.  
![\[Screenshot of the Devices page with the Add your first Device button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-066.png)

1. <a name="gg-group-create-device"></a>On the **Add a Device** page, choose **Create New Device**\.

1. On the **Create a Registry entry for a device** page, register this device as **HelloWorld\_Publisher**, and then choose **Next**\.  
![\[Screenshot of Create a Registry entry for a device with the Name field set to HelloWorld_Publisher and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-068.png)

1. <a name="gg-group-generate-default-device-certs"></a>On the **Set up security** page, for **1\-Click**, choose **Use Defaults**\. This option generates a device certificate with an attached [AWS IoT policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) and public and private key\.

1. Create a folder on your computer\. Download the certificate and keys for your device into the folder\.  
![\[Screenshot of the Download security credentials page with the Download these resources as a tar.gz button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-070.png)

   Make a note of the common *hash* component in the file names for the HelloWorld\_Publisher device certificate and keys \(in this example, `bcc5afd26d`\)\. You need it later\. Choose **Finish**\.

1. Decompress the `hash-setup.tar.gz` file\. For example, run the following command:

   ```
   tar -xzf hash-setup.tar.gz
   ```
**Note**  
On Windows, you can decompress `.tar.gz` files using a tool such as [7\-Zip](http://www.7-zip.org/) or [WinZip](http://www.winzip.com/)\.

1. Choose **Add Device** and repeat steps 3 \- 7 to add a new device to the group\.

   Name this device **HelloWorld\_Subscriber**\. Download the certificates and keys for the device to your computer\. Save and decompress them in the same folder that you created for HelloWorld\_Publisher\.

   Again, make a note of the common *hash* component in the file names for the HelloWorld\_Subscriber device\.

   You should now have two devices in your AWS IoT Greengrass group:  
![\[Screenshot showing the HelloWorld_Publisher and HelloWorld_Subscriber devices.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-071.png)

1. <a name="root-ca-device"></a>Review [Server Authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) in the *AWS IoT Developer Guide* and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\. Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a VeriSign root CA certificate with a legacy endpoint\. Only some AWS Regions support legacy endpoints\. For more information, see [Service endpoints must match the root CA certificate type](gg-core.md#certificate-endpoints)\.

   Save the root CA certificate as `root-ca-cert.pem` in the same folder as the device certificates and keys for both devices\. All these files should be in one folder on your computer \(not on the Greengrass core device\)\.
   + For ATS endpoints \(preferred\), download the appropriate ATS root CA certificate, such as [Amazon Root CA 1](https://www.amazontrust.com/repository/AmazonRootCA1.pem)\.
   + For legacy endpoints, download a [ VeriSign root CA certificate](https://www.websecurity.digicert.com/content/dam/websitesecurity/digitalassets/desktop/pdfs/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem)\. Although legacy endpoints are acceptable for the purposes of this tutorial, we recommend that you use an ATS endpoint and download an ATS root CA certificate\.
**Note**  
If you're using a web browser on the Mac and you see This certificate is already installed as a certificate authority, open a Terminal window and download the certificate into the folder that contains the HelloWorld\_Publisher and HelloWorld\_Subscriber device certificates and keys\. For example, if you're using an ATS endpoint, you can run the following command to download the Amazon Root CA 1 certificate\.  

   ```
   cd path-to-folder-containing-device-certificates
   curl -o ./root-ca-cert.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
   ```
Run `cat root-ca-cert.pem` to ensure that the file is not empty\. If the file is empty, check the URL and try the `curl` command again\.