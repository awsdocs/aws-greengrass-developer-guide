# Configure AWS Greengrass on AWS IoT<a name="gg-config"></a>

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/) on your computer and open the AWS IoT console\. If this is the first time opening this console, choose **Get started**\. 

   Next, choose **Greengrass**:  
![\[AWS IoT left navigation screenshot with Greengrass highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-003.png)

1. On the **Welcome to AWS Greengrass** page, choose **Get Started**:  
![\[Welcome to AWS Greengrass console page with the Get Started button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-004.png)

1. Create an AWS Greengrass group\. An AWS Greengrass group contains information about the devices and how messages are processed in the group\. Each AWS Greengrass group requires an AWS Greengrass core device that processes messages sent within the group\. An AWS Greengrass core needs a certificate and an AWS IoT policy to access AWS Greengrass and AWS Cloud Services\. On the **Set up your Greengrass group** page, choose **Use easy creation**\.  
![\[Set up your Greengrass Group console page with the Use easy creation button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-005.png)

1. Type a name for your group \(for example, **MyFirstGroup**\), then choose **Next**:  
![\[The Name your Group page with MyFirstGroup in the Group Name field and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-006.png)

1. Use the default name for the AWS Greengrass core, and choose **Next**:  
![\[The Every Group needs a Core to function page with MyFirstGroup_Core in the Name field and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-007.png)

1. On the **Run a scripted easy Group creation** page, choose **Create Group and Core**\.  
![\[The Run a scripted easy Group creation page with the Create Group and Core button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-008.png)

   AWS IoT creates an AWS Greengrass group for you with default security policies and configuration files for you to load onto your device\.

1. <a name="gg-core-download"></a>On the confirmation page, download your core's security resources and the AWS Greengrass Core software, as follows:

   1. Under **Download and store your Core's security resources**, choose **Download these resources as a tar\.gz** to download the required security resources for your AWS Greengrass core\.

   1. Under **Download the current Greengrass Core software**, choose the CPU architecture of your core device:

      + If you're using a Raspberry Pi, choose **ARMv7l**\.

      + If you're using an Amazon EC2 instance, choose **x86\_64**\.

   1. Choose **Download Greengrass version 1\.3\.0** to download the AWS Greengrass core binary\.
**Important**  
You must download both the security resources *and* the AWS Greengrass Core software before you choose **Finish**\.  
![\[The Connect your Core device page with Download these resources as a tar.gz, ARMv71, Download Greengrass version 1.3.0, and the Finish button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.png)

1. After downloading the security resources and the AWS Greengrass Core software, choose **Finish**\. The group configuration page is displayed in the console:  
![\[Empty group configuration page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.2.png)