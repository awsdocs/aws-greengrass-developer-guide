# Configure AWS IoT Greengrass on AWS IoT<a name="gg-config"></a>

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/) on your computer and open the AWS IoT console\. If this is your first time opening this console, choose **Get started**\.

1. Choose **Greengrass**\.
**Note**  
If you don't see the **Greengrass** node in the navigation pane, change to an AWS Region that supports AWS IoT Greengrass\. For the list of supported regions, see [AWS Regions and Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#greengrass_region) in the *AWS General Reference*\.  
![\[AWS IoT navigation pane with Greengrass highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-greengrass.png)

1. On the **Welcome to AWS IoT Greengrass** page, choose **Create a Group**\.

   An AWS IoT Greengrass [group](what-is-gg.md#gg-group) contains settings and other information about its components, such as devices, Lambda functions, and connectors\. A group defines how its components can interact with each other\.
**Tip**  
For an example that uses the AWS IoT Greengrass API to create and deploy a group, see the [ gg\_group\_setup](https://github.com/awslabs/aws-greengrass-group-setup) package from GitHub\.

1. If prompted, on the **Greengrass needs your permission to access other services** dialog box, choose **Grant permission** to allow the console to create or configure the Greengrass service role for you\. You must use a service role to authorize AWS IoT Greengrass to access other AWS services on your behalf\. Otherwise, deployments fail\.  
![\[The "Greengrass needs your permission to access other services" dialog box.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/service-role-grant-perms.png)

   The AWS account you used to sign in must have permissions to create or manage the IAM role\. For more information, see [Greengrass Service Role](service-role.md)\.

1. On the **Set up your Greengrass group** page, choose **Use easy creation** to create a group and an AWS IoT Greengrass [core](gg-core.md)\.

   Each group requires a core, which is a device that manages local IoT processes\. A core needs a certificate and keys that allow it to access AWS IoT and an AWS IoT policy that allows it to perform AWS IoT and AWS IoT Greengrass actions\. When you choose the **Use easy creation** option, these security resources are created for you and the core is provisioned in the AWS IoT registry\.  
![\[Set up your Greengrass Group console page with the Use easy creation button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-005.png)

1. Enter a name for your group \(for example, **MyFirstGroup**\), and then choose **Next**\.  
![\[The Name your Group page with MyFirstGroup in the Group Name field and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-006.png)

1. Use the default name for the AWS IoT Greengrass core, and then choose **Next**\.  
![\[The Every Group needs a Core to function page with MyFirstGroup_Core in the Name field and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-007.png)

1. On the **Run a scripted easy Group creation** page, choose **Create Group and Core**\.  
![\[The Run a scripted easy Group creation page with the Create Group and Core button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-008.png)

   AWS IoT creates an AWS IoT Greengrass group with default security policies and configuration files for you to load onto your device\.

1. <a name="gg-core-download"></a>Download your core's security resources and configuration file\. On the confirmation page, under **Download and store your Core's security resources**, choose **Download these resources as a tar\.gz**\. The name of your downloaded `tar.gz` file starts with a 10\-digit hash that's also used for the certificate and key file names\.  
![\[The Connect your Core device page with Download these resources as a tar.gz highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.png)
**Important**  
Download the security resources before you choose **Finish**\.

1. After you download the security resources, choose **Finish**\.

   The group configuration page is displayed in the console:  
![\[Empty group configuration page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.2.png)

1. Download the [AWS IoT Greengrass Core software](what-is-gg.md#gg-core-download-tab) installation package\. Choose the CPU architecture and distribution \(and operating system, if necessary\) that best describe your core device\. For example:
   + For Raspberry Pi Model B or B\+, download the Armv7l for Raspbian package\.
   + For an Amazon EC2 instance, download the x86\_64 for Linux package\.
   + For NVIDIA Jetson TX2, download the Armv8 \(AArch64\) for Ubuntu package\.
   + For Intel Atom, download the x86\_64 for Linux package\.
**Note**  
When stream manager is enabled, you must install the [Java 8 runtime](stream-manager.md#stream-manager-requirements) on the core device before you deploy your group\. The **Easy Group creation** workflow enables stream manager by default\.