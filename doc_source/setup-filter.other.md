# Setting up other devices<a name="setup-filter.other"></a>

Follow the steps in this topic to set up a device \(other than a Raspberry Pi\) to use as your AWS IoT Greengrass core\.

**Tip**  
Or, to use a script that sets up your environment and installs the AWS IoT Greengrass Core software for you, see [Quick start: Greengrass device setup](quick-start.md)\.

If you're new to AWS IoT Greengrass, we recommend that you use a Raspberry Pi or an Amazon EC2 instance as your core device, and follow the [setup steps](module1.md) appropriate for your device\.

If you plan to build a custom Linux\-based system using the Yocto Project, you can use the AWS IoT Greengrass Bitbake Recipe from the meta\-aws project\. This recipe also helps you develop a software platform that supports AWS edge software for embedded applications\. The Bitbake build installs, configures, and automatically runs the AWS IoT Greengrass Core software on your device\.

Yocto Project  
An open source collaboration project that helps you build custom Linux\-based systems for embedded applications regardless hardware architecture\. For more information, see the [Yocto Project](https://www.yoctoproject.org/)\.

meta\-aws  
An AWS managed project that provides Yocto recipes\. You can use the recipes to develop AWS edge sofware in Linux\-based systems built with [OpenEmbedded](https://www.openembedded.org/wiki/Main_Page) and Yocto Project\. For more information about this community supported capability, see the [meta\-aws](https://github.com/aws/meta-aws)project on GitHub\.

meta\-aws\-demos  
An AWS managed project that contains demonstrations for the meta\-aws project\. For more examples about the integration process, see the [meta\-aws\-demos](https://github.com/aws-samples/meta-aws-demos) project on GitHub\.

To use a different device or [supported platform](what-is-gg.md#gg-platforms), follow the steps in this topic\.

1. <a name="setup-jetson"></a>If your core device is an NVIDIA Jetson device, you must first flash the firmware with the JetPack 4\.3 installer\. If you're configuring a different device, skip to step 2\.
**Note**  
The JetPack installer version that you use is based on your target CUDA Toolkit version\. The following instructions use JetPack 4\.3 and CUDA Toolkit 10\.0\. For information about using the versions appropriate for your device, see [How to Install Jetpack](https://docs.nvidia.com/jetson/jetpack/install-jetpack/index.html) in the NVIDIA documentation\.

   1. On a physical desktop that is running Ubuntu 16\.04 or later, flash the firmware with the JetPack 4\.3 installer, as described in [Download and Install JetPack](https://docs.nvidia.com/jetson/archives/jetpack-archived/jetpack-33/index.html#jetpack/3.3/install.htm%3FTocPath%3D_____3) \(4\.3\) in the NVIDIA documentation\.

      Follow the instructions in the installer to install all the packages and dependencies on the Jetson board, which must be connected to the desktop with a Micro\-B cable\.

   1. Reboot your board in normal mode, and connect a display to the board\.
**Note**  
When you use SSH to connect to the Jetson board, use the default user name \(**nvidia**\) and the default password \(**nvidia**\)\.

1. Run the following commands to create user `ggc_user` and group `ggc_group`\. The commands you run differ, depending on the distribution installed on your core device\.
   + If your core device is running OpenWrt, run the following commands:

     ```
     opkg install shadow-useradd
     opkg install shadow-groupadd
     useradd --system ggc_user
     groupadd --system ggc_group
     ```
   + Otherwise, run the following commands:

     ```
     sudo adduser --system ggc_user
     sudo addgroup --system ggc_group
     ```
**Note**  
If the `addgroup` command isn't available on your system, use the following command\.  

     ```
     sudo groupadd --system ggc_group
     ```

1. <a name="install-java-8-runtime"></a>Optional\. Install the Java 8 runtime, which is required by [stream manager](stream-manager.md)\. This tutorial doesn't use stream manager, but it does use the **Default Group creation** workflow that enables stream manager by default\. Use the following commands to install the Java 8 runtime on the core device, or disable stream manager before you deploy your group\. Instructions for disabling stream manager are provided in Module 3\.
   + For Debian\-based or Ubuntu\-based distributions:

     ```
     sudo apt install openjdk-8-jdk
     ```
   + For Red Hat\-based distributions:

     ```
     sudo yum install java-1.8.0-openjdk
     ```

1. To make sure that you have all required dependencies, download and run the Greengrass dependency checker from the [AWS IoT Greengrass Samples](https://github.com/aws-samples/aws-greengrass-samples) repository on GitHub\. These commands unzip and run the dependency checker script\.

   ```
   mkdir greengrass-dependency-checker-GGCv1.11.x
   cd greengrass-dependency-checker-GGCv1.11.x
   wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.11.x.zip
   unzip greengrass-dependency-checker-GGCv1.11.x.zip
   cd greengrass-dependency-checker-GGCv1.11.x
   sudo ./check_ggc_dependencies | more
   ```
**Note**  
The `check_ggc_dependencies` script runs on AWS IoT Greengrass supported platforms and requires specific Linux system commands\. For more information, see the dependency checker's [Readme](https://github.com/aws-samples/aws-greengrass-samples/blob/master/greengrass-dependency-checker-GGCv1.11.x/README.md)\.

1. Install all required dependencies on your device, as indicated by the dependency checker output\. For missing kernel\-level dependencies, you might have to recompile your kernel\. For mounting Linux control groups \(`cgroups`\), you can run the [cgroupfs\-mount](https://raw.githubusercontent.com/tianon/cgroupfs-mount/master/cgroupfs-mount) script\. This allows AWS IoT Greengrass to set the memory limit for Lambda functions\. Cgroups are also required to run AWS IoT Greengrass in the default [containerization](lambda-group-config.md#lambda-containerization-considerations) mode\.

   If no errors appear in the output, AWS IoT Greengrass should be able to run successfully on your device\.
**Important**  
<a name="lambda-runtime-prereqs"></a>This tutorial requires the Python 3\.7 runtime to run local Lambda functions\. When stream manager is enabled, it also requires the Java 8 runtime\. If the `check_ggc_dependencies` script produces warnings about these missing runtime prerequisites, make sure to install them before you continue\. You can ignore warnings about other missing optional runtime prerequisites\.

   For the list of AWS IoT Greengrass requirements and dependencies, see [Supported platforms and requirements](what-is-gg.md#gg-platforms)\.