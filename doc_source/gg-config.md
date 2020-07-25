# Configure AWS IoT Greengrass on AWS IoT<a name="gg-config"></a>

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/) on your computer and open the AWS IoT console\. If this is your first time opening this console, choose **Get started**\.

1. In the navigation pane, choose **Greengrass**\.  
![\[AWS IoT navigation pane with Greengrass highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-greengrass.png)
**Note**  
If you don't see the **Greengrass** node, change to an AWS Region that supports AWS IoT Greengrass\. For the list of supported Regions, see [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) in the *Amazon Web Services General Reference*\.

1. On the **Welcome to AWS IoT Greengrass** page, choose **Create a Group**\.

   An AWS IoT Greengrass [group](what-is-gg.md#gg-group) contains settings and other information about its components, such as devices, Lambda functions, and connectors\. A group defines how its components can interact with each other\.
**Tip**  
For an example that uses the AWS IoT Greengrass API to create and deploy a group, see the [ gg\_group\_setup](https://github.com/awslabs/aws-greengrass-group-setup) package from GitHub\.

1. If prompted, on the **Greengrass needs your permission to access other services** dialog box, choose **Grant permission** to allow the console to create or configure the Greengrass service role for you\. You must use a service role to authorize AWS IoT Greengrass to access other AWS services on your behalf\. Otherwise, deployments fail\.  
![\[The "Greengrass needs your permission to access other services" dialog box.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/service-role-grant-perms.png)

   The AWS account you used to sign in must have permissions to create or manage the IAM role\. For more information, see [Greengrass service role](service-role.md)\.

1. On the **Set up your Greengrass group** page, choose **Use default creation** to create a group and an AWS IoT Greengrass [core](gg-core.md)\.

   Each group requires a core, which is a device that manages local IoT processes\. A core needs a certificate and keys that allow it to access AWS IoT and an [AWS IoT policy](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html) that allows it to perform AWS IoT and AWS IoT Greengrass actions\. When you choose the **Use default creation** option, these security resources are created for you and the core is provisioned in the AWS IoT registry\.  
![\[Set up your Greengrass Group console page with the Use default creation button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-005.png)

1. Enter a name for your group \(for example, **MyFirstGroup**\), and then choose **Next**\.  
![\[The Name your Group page with MyFirstGroup in the Group Name field and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-006.png)

1. Use the default name for the AWS IoT Greengrass core, and then choose **Next**\.  
![\[The Every Group needs a Core to function page with MyFirstGroup_Core in the Name field and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-007.png)

1. On the **Review Group creation** page, choose **Create Group and Core**\.  
![\[The Review Group creation page with the Create Group and Core button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-008.png)

   AWS IoT creates an AWS IoT Greengrass group with default security policies and configuration files for you to load onto your device\.

1. <a name="download-ggc-security-resources"></a>Download your core's security resources and configuration file\.

   1. On the confirmation page, under **Download and store your Core's security resources**, choose **Download these resources as a tar\.gz**\. The name of your downloaded `tar.gz` file starts with a 10\-digit hash that's also used for the certificate and key file names\.
**Important**  
Before you choose **Finish**, download the security resources\.  
![\[The Connect your Core device page with Download these resources as a tar.gz highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.png)

   1. Skip **Choose a root CA** for now\. The next section includes a step where you download the root CA certificate\.

1. After you download the security resources, choose **Finish**\.

   The group configuration page displays in the console:  
![\[Empty group configuration page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.2.png)

1. <a name="download-ggc-software"></a>From the [AWS IoT Greengrass Core software](what-is-gg.md#gg-core-download-tab) section in this guide, download the AWS IoT Greengrass Core software installation package\.

   Choose the package that best fits the CPU architecture, distribution, and OS of your core device\. For example:
   + For Raspberry Pi Model B or B\+, download the package for the Armv7l architecture and Raspbian distribution\.
   + For an Amazon EC2 instance, download the package for the x86\_64 architecture and Linux distribution\.
   + For NVIDIA Jetson TX2, download the package for the Armv8 \(AArch64\) architecture and Arch Linux distribution\.
   + For Intel Atom, download the package for the x86\_64 architecture and Linux distribution\.