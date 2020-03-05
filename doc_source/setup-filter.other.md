# Setting Up Other Devices<a name="setup-filter.other"></a>

Follow the steps in this topic to set up a device \(other than a Raspberry Pi\) to use as your AWS IoT Greengrass core\.

**Tip**  
Or, to use a script that sets up your environment and installs the AWS IoT Greengrass Core software for you, see [Quick Start: Greengrass Device Setup](quick-start.md)\.

If you're new to AWS IoT Greengrass, we recommend that you use a Raspberry Pi or an Amazon EC2 instance as your core device, and follow the [setup steps](module1.md) appropriate for your device\. To use a different device or [supported platform](what-is-gg.md#gg-platforms), follow the steps in this topic\.

 

1. <a name="setup-jetson"></a>If your core device is an NVIDIA Jetson TX2, you must first flash the firmware with the JetPack 3\.3 installer\. If you're configuring a different device, skip to step 2\.
**Note**  
The JetPack installer version that you use is based on your target CUDA Toolkit version\. The following instructions use JetPack 3\.3 and CUDA Toolkit 9\.0 because the TensorFlow v1\.10\.1 and MXNet v1\.2\.1 binaries \(that AWS IoT Greengrass provides for machine learning inference on a Jetson TX2\) are compiled against this version of CUDA\. For more information, see [Perform Machine Learning Inference](ml-inference.md)\.

   1. On a physical desktop that is running Ubuntu 16\.04 or later, flash the firmware with the JetPack 3\.3 installer, as described in [Download and Install JetPack](https://docs.nvidia.com/jetson/archives/jetpack-archived/jetpack-33/index.html#jetpack/3.3/install.htm%3FTocPath%3D_____3) \(3\.3\) in the NVIDIA documentation\.

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

1. <a name="install-java-8-runtime"></a>Install the Java 8 runtime\. This tutorial uses the **Default Group creation** workflow, which enables [stream manager](stream-manager.md) in the group by default\. You must install the Java 8 runtime on the core device \(or [disable stream manager](configure-stream-manager.md#enable-stream-manager-console-existing-group)\) before you deploy your group\.
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
   mkdir greengrass-dependency-checker-GGCv1.10.x
   cd greengrass-dependency-checker-GGCv1.10.x
   wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.10.x.zip
   unzip greengrass-dependency-checker-GGCv1.10.x.zip
   cd greengrass-dependency-checker-GGCv1.10.x
   sudo ./check_ggc_dependencies | more
   ```
**Note**  
The `check_ggc_dependencies` script runs on AWS IoT Greengrass supported platforms and requires specific Linux system commands\. For more information, see the dependency checker's [Readme](https://github.com/aws-samples/aws-greengrass-samples/blob/master/greengrass-dependency-checker-GGCv1.10.x/README.md)\.

1. Install all required dependencies on your device, as indicated by the dependency checker output\. For missing kernel\-level dependencies, you might have to recompile your kernel\. For mounting Linux control groups \(`cgroups`\), you can run the [cgroupfs\-mount](https://raw.githubusercontent.com/tianon/cgroupfs-mount/master/cgroupfs-mount) script\. This allows AWS IoT Greengrass to set the memory limit for Lambda functions\. Cgroups are also required to run AWS IoT Greengrass in the default [containerization](lambda-group-config.md#lambda-containerization-considerations) mode\.

   If no errors appear in the output, AWS IoT Greengrass should be able to run successfully on your device\.
**Important**  
<a name="lambda-runtime-prereqs"></a>This tutorial requires the Python 3\.7 runtime to run local Lambda functions\. When stream manager is enabled, it also requires the Java 8 runtime\. If the `check_ggc_dependencies` script produces warnings about these missing runtime prerequisites, make sure to install them before you continue\. You can ignore warnings about other missing optional runtime prerequisites\.

   For the list of AWS IoT Greengrass requirements and dependencies, see [Supported Platforms and Requirements](what-is-gg.md#gg-platforms)\.