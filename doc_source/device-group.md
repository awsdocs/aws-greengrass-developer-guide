# Create AWS IoT Devices in an AWS IoT Greengrass Group<a name="device-group"></a>

1. In the AWS IoT Core console, choose **Greengrass**, choose **Groups**, and then choose your group\.

1. On the group configuration page, choose **Devices**, and then choose **Add your first Device**\.  
![\[Screenshot of the Devices page with the Add your first Device button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-066.png)

1. Choose **Create New Device**\.  
![\[Screenshot of the Add a Device page with the Create New Device button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-067.png)

1. Register this device as **HelloWorld\_Publisher**, and then choose **Next**\.  
![\[Screenshot of Create a Registry entry for a device with the Name field set to HelloWorld_Publisher and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-068.png)

1. For **1\-Click**, choose **Use Defaults**\. This option generates a device certificate with attached AWS IoT policy and public and private key\.  
![\[Screenshot of Set up security with the Use Defaults button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-069.png)

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

1. <a name="root-ca-device"></a>Review the documentation on [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html), follow the links to download the appropriate CA certificate, and then save it as `root-ca-cert.pem` in the same folder\. For this module, this certificate and the certificates and keys for both devices should be in one folder on your computer \(not on the AWS IoT Greengrass core device\)\.
**Note**  
If you're using a web browser on the Mac and you see This certificate is already installed as a certificate authority, open a Terminal window and run the following commands to download the certificate into the folder that contains the HelloWorld\_Publisher and HelloWorld\_Subscriber device certificates and keys:  

   ```
   cd path-to-folder-containing-device-certificates
   curl -o ./root-ca-cert.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
   ```
Run `cat root-ca-cert.pem` to ensure that the file is not empty\. If the file is empty, check the URL and try the `curl` command again\.