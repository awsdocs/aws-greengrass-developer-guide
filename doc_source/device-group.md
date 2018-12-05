# Create AWS IoT Devices in an AWS IoT Greengrass Group<a name="device-group"></a>

1. In the AWS IoT console, choose **Greengrass**, choose **Groups**, and then choose your group\. On the group configuration page, choose **Devices**, and then choose **Add your first Device**\.  
![\[Screenshot of the Devices page with the Add your first Device button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-066.png)

   Choose **Create New Device**\.  
![\[Screenshot of the Add a Device page with the Create New Device button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-067.png)

   Register this device as **HelloWorld\_Publisher**, and then choose **Next**\.  
![\[Screenshot of Create a Registry entry for a device with the Name field set to HelloWorld_Publisher and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-068.png)

   For **1\-Click**, choose **Use Defaults**\.  
![\[Screenshot of Set up security with the Use Defaults button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-069.png)

   Create a folder on your computer\.

   Download the certificates for your device into the folder, and then decompress them\. To decompress `tar.gz` files on Windows, see step 2 of [Create and Package a Lambda Function](create-lambda.md)\.  
![\[Screenshot of the Download security credentials page with the Download these resources as a tar.gz button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-070.png)

   Make a note of the common GUID\-like filename component for the HelloWorld\_Publisher device \(in this example, `51d2737e90`\)\. You need it later\. Choose **Finish**\.

1. Choose **Add Device** and repeat step 1 to add another device to the group\. Name it **HelloWorld\_Subscriber**\. Download the certificates for your second device onto your computer and save them in the same folder as the first set of certificates\. Choose **Finish**\. Again, make a note of the common GUID\-like filename component for the HelloWorld\_Subscriber device\.

   You should now have two devices in your AWS IoT Greengrass group:  
![\[Screenshot showing the HelloWorld_Publisher and HelloWorld_Subscriber devices.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-071.png)

1. Review the documentation on [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html), follow the links to download the appropriate CA certificate, and then save it as `root-ca-cert.pem` in the folder you just created\. For this module, this certificate and the certificates and keys for both devices should be in one folder on your computer \(not on the AWS IoT Greengrass core device\)\.
**Note**  
If you're using a web browser on the Mac and you see This certificate is already installed as a certificate authority, open a Terminal window and run the following commands to download the certificate into the folder that contains the HelloWorld\_Publisher and HelloWorld\_Subscriber device certificates and keys:  

   ```
   cd path-to-folder-containing-device-certificates
   curl -o ./root-ca-cert.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
   ```
Run `cat root-ca-cert.pem` to ensure that the file is not empty\. If the file is empty, check the URL and try the `curl` command again\.