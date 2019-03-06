# Configure AWS IoT Greengrass on AWS IoT<a name="gg-config"></a>

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/) on your computer and open the AWS IoT Core console\. If this is the first time opening this console, choose **Get started**\.

1. Choose **Greengrass**\.  
![\[AWS IoT navigation pane with Greengrass highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-greengrass.png)

1. On the **Welcome to AWS IoT Greengrass** page, choose **Create a Group**\.

   An AWS IoT Greengrass [group](what-is-gg.md#gg-group) contains settings and other information about its components, such as devices, Lambda functions, and connectors\. A group defines how its components can interact with each other\.
**Tip**  
For an example that uses the AWS IoT Greengrass API to create and deploy a group, see the [ gg\_group\_setup](https://github.com/awslabs/aws-greengrass-group-setup) package from GitHub\.

1. On the **Set up your Greengrass group** page, choose **Use easy creation**\.

   Each group requires an AWS IoT Greengrass [core](gg-core.md), which manages local processes\. A core needs a certificate that allows it to access AWS IoT and an AWS IoT policy that allows it to perform AWS IoT and AWS IoT Greengrass actions\. This step creates and provisions a core for the new group\.  
![\[Set up your Greengrass Group console page with the Use easy creation button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-005.png)

1. Enter a name for your group \(for example, **MyFirstGroup**\), and then choose **Next**\.  
![\[The Name your Group page with MyFirstGroup in the Group Name field and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-006.png)

1. Use the default name for the AWS IoT Greengrass core, and then choose **Next**\.  
![\[The Every Group needs a Core to function page with MyFirstGroup_Core in the Name field and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-007.png)

1. On the **Run a scripted easy Group creation** page, choose **Create Group and Core**\.  
![\[The Run a scripted easy Group creation page with the Create Group and Core button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-008.png)

   AWS IoT creates an AWS IoT Greengrass group with default security policies and configuration files for you to load onto your device\.

1. <a name="gg-core-download"></a>On the confirmation page, download your core's security resources and the AWS IoT Greengrass core software:
**Note**  
You can alternatively download the core software from the [AWS IoT Greengrass Core Software](what-is-gg.md#gg-core-download-tab) downloads\.

   1. Under **Download and store your Core's security resources**, choose **Download these resources as a tar\.gz** to download the required security resources for your AWS IoT Greengrass core\.  
![\[The Connect your Core device page with Download these resources as a tar.gz highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.png)

   1. Under **Software configurations**, choose the CPU architecture and distribution \(and operating system, if necessary\) that best describes your core device\. For example:
      + For Raspberry Pi, download the ARMv7l for Raspbian package\.
      + For an Amazon EC2 instance, download the x86\_64 for Linux package\.
      + For NVIDIA Jetson TX2, download the ARMv8 \(AArch64\) for Ubuntu package\.
      + For Intel Atom, download the x86\_64 for Linux package\.  
![\[The Connect your Core device page with Download highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.1.png)
**Important**  
Download both the security resources and core software before you choose **Finish**\.

1. After you have downloaded the security resources and the AWS IoT Greengrass Core software, choose **Finish**\.

   The group configuration page is displayed in the console:  
![\[Empty group configuration page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.2.png)