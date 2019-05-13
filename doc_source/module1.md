# Module 1: Environment Setup for Greengrass<a name="module1"></a>

This module shows you how to get an out\-of\-the\-box Raspberry Pi, Amazon EC2 instance, or other device ready to be used by AWS IoT Greengrass\.

**Important**  
Use the **Filter View** drop\-down list in the upper\-right corner of this webpage to choose your platform\.

This module should take less than 30 minutes to complete\.

## Setting Up a Raspberry Pi<a name="setup-filter.rpi"></a>

If you are setting up a Raspberry Pi for the first time, you must follow all of these steps\. Otherwise, you can skip to step 9\. However, we recommend that you re\-image your Raspberry Pi with the operating system as recommended in step 2\.

1. Download and install an SD card formatter such as [SD Memory Card Formatter](https://www.sdcard.org/downloads/formatter_4/index.html) or [PiBakery](http://www.pibakery.org/download.html)\. Insert the SD card into your computer\. Start the program and choose the drive where you have inserted your SD card\. You can perform a quick format of the SD card\.

1. Download the [Raspbian Stretch](https://downloads.raspberrypi.org/raspbian/images/raspbian-2018-06-29/) operating system as a `.zip` file\.

1. Using an SD card\-writing tool \(such as [Etcher](https://etcher.io/)\), follow the tool's instructions to flash the downloaded `zip` file onto the SD card\. Because the operating system image is large, this step might take some time\. Eject your SD card from your computer, and insert the microSD card into your Raspberry Pi\.

1. For the first boot, we recommend that you connect the Raspberry Pi to a monitor \(through HDMI\), a keyboard, and a mouse\. Next, connect your Pi to a micro USB power source and the Raspbian operating system should start up\. 

1. You might want to configure the Pi's keyboard layout before you continue\. To do so, choose the Raspberry icon in the upper\-right, choose **Preferences** and then choose **Mouse and Keyboard Settings**\. Next, on the **Keyboard** tab, choose **Keyboard Layout**, and then choose an appropriate keyboard variant\.

1. Next, [connect your Raspberry Pi to the internet through a Wi\-Fi network](https://www.raspberrypi.org/documentation/configuration/wireless/desktop.md) or an Ethernet cable\.
**Note**  
Connect your Raspberry Pi to the *same* network that your computer is connected to, and be sure that both your computer and Raspberry Pi have internet access before you continue\. If you're in a work environment or behind a firewall, you might need to connect your Pi and your computer to the guest network to get both devices on the same network\. However, this approach might disconnect your computer from local network resources, such as your intranet\. One solution is to connect the Pi to the guest Wi\-Fi network and to connect your computer to the guest Wi\-Fi network *and* your local network through an Ethernet cable\. In this configuration, your computer should be able to connect to the Raspberry Pi through the guest Wi\-Fi network and your local network resources through the Ethernet cable\.

1. You must set up [SSH](https://en.wikipedia.org/wiki/Secure_Shell) on your Pi to remotely connect to it\. On your Raspberry Pi, open a [terminal window](https://www.raspberrypi.org/documentation/usage/terminal/) and run the following command:

   ```
   sudo raspi-config
   ```

   You should see the following:  
![\[Raspberry Pi Software Configuration Tool (raspi-config) screenshot.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-001.png)

   Scroll down and choose **Interfacing Options** and then choose **P2 SSH**\. When prompted, choose **Yes**\. \(Use the Tab key followed by Enter\)\. SSH should now be enabled\. Choose **OK**\. Use the Tab key to choose **Finish** and then press Enter\. If the Raspberry Pi doesn't reboot automatically, run the following command:

   ```
   sudo reboot
   ```

1. On your Raspberry Pi, run the following command in the terminal:

   ```
   hostname -I
   ```

   This returns the IP address of your Raspberry Pi\.
**Note**  
For the following, if you receive an ECDSA key fingerprint message \(`Are you sure you want to continue connecting (yes/no)?`\), enter `yes`\. The default password for the Raspberry Pi is **raspberry**\.

   If you are using macOS, open a terminal window and enter the following:

   ```
   ssh pi@IP-address
   ```

   *IP\-address* is the IP address of your Raspberry Pi that you obtained by using the `hostname -I` command\.

   If you are using Windows, you need to install and configure [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)\. Expand **Connection**, choose **Data**, and make sure that **Prompt** is selected:   
![\[PuTTY window with Prompt selected.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-001.4.png)

   Next, choose **Session**, enter the IP address of the Raspberry Pi, and then choose **Open** using default settings\.   
![\[PuTTY window with IP address in the "Host Name (or IP address)" field.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-001.5.png)

   If a PuTTY security alert is displayed, choose **Yes**\.

   The default Raspberry Pi login and password are **pi** and **raspberry**, respectively\.  
![\[Initial PuTTY terminal window.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-001.6.png)
**Note**  
If your computer is connected to a remote network using VPN, you might have difficulty connecting from the computer to the Raspberry Pi using SSH\.

1. You are now ready to set up the Raspberry Pi for AWS IoT Greengrass\. First, run the following commands from a local Raspberry Pi terminal window or an SSH terminal window:

   ```
   sudo adduser --system ggc_user
   sudo addgroup --system ggc_group
   ```

1. To improve security on the Pi device, enable hardlink and softlink protection on the operating system at start up\.

   1. Navigate to the `98-rpi.conf` file\.

      ```
      cd /etc/sysctl.d
      ls
      ```
**Note**  
If you don't see the `98-rpi.conf` file, follow the instructions in the `README.sysctl` file\.

   1. Use a text editor \(such as Leafpad, GNU nano, or vi\) to add the following two lines to the end of the file\. You might need to use the `sudo` command to edit as root \(for example, `sudo nano 98-rpi.conf`\)\.

      ```
      fs.protected_hardlinks = 1
      fs.protected_symlinks = 1
      ```

   1. Reboot the Pi\.

      ```
      sudo reboot
      ```

      After about a minute, connect to the Pi using SSH and then run the following command to confirm the change:

      ```
      sudo sysctl -a 2> /dev/null | grep fs.protected
      ```

      You should see `fs.protected_hardlinks = 1` and `fs.protected_symlinks = 1`\.

1. <a name="stretch-step"></a> Edit your command line boot file to enable and mount memory cgroups\. This allows AWS IoT Greengrass to set the memory limit for Lambda functions\. Cgroups are also required to run AWS IoT Greengrass in the default [containerization](lambda-group-config.md#lambda-containerization-considerations) mode\.

   1.  Navigate to your `boot` directory\. 

      ```
      cd /boot/
      ```

   1.  Use a text editor to open `cmdline.txt`\. Append the following to the end of the existing line, not as a new line\.

      ```
      cgroup_enable=memory cgroup_memory=1
      ```

   1. Now reboot the Pi\.

      ```
      sudo reboot
      ```

   Your Raspberry Pi should now be ready for AWS IoT Greengrass\.

1. To make sure that you have all required dependencies, download and run the Greengrass dependency checker from the [AWS IoT Greengrass Samples](https://github.com/aws-samples/aws-greengrass-samples) repository on GitHub\. These commands unzip and run the dependency checker script in the `Downloads` directory\.

   ```
   cd /home/pi/Downloads
   mkdir greengrass-dependency-checker-GGCv1.9.0
   cd greengrass-dependency-checker-GGCv1.9.0
   wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.9.0.zip
   unzip greengrass-dependency-checker-GGCv1.9.0.zip
   sudo modprobe configs
   sudo ./check_ggc_dependencies | more
   ```

   Where `more` appears, press the Spacebar key to display another screen of text\. 
**Important**  
This tutorial requires Python 2\.7\. The `check_ggc_dependencies` script might produce warnings about the missing optional Node\.js and Java prerequisites\. You can ignore these warnings\.

   For information about the modprobe command, run man modprobe in the terminal\. 

Your Raspberry Pi configuration is complete\. Continue to [Module 2: Installing the Greengrass Core Software](module2.md)\.

## Setting Up an Amazon EC2 Instance<a name="setup-filter.ec2"></a>

This section provides instructions for setting up your Amazon EC2 instance\.

**Note**  
 Although you can complete these modules using an Amazon EC2 instance, AWS IoT Greengrass should ideally be used with physical hardware\. We recommend that you use a Raspberry Pi for these tutorials\. 

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/) and launch an Amazon EC2 instance using an Amazon Linux AMI\. For information about Amazon EC2 instances, see the [Amazon EC2 Getting Started Guide](https://docs.aws.amazon.com/AWSEC2/latest/GettingStartedGuide/)\.

1. After your Amazon EC2 instance is running, enable port 8883 to allow incoming MQTT communications so that other devices can connect with the AWS IoT Greengrass core\.

   1. In the navigation pane of the Amazon EC2 console, choose **Security Groups**\.  
![\[Navigation pane with Security Groups highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.1.png)

   1. Select the security group for the instance that you just launched, and then choose the **Inbound** tab\.  
![\[Inbound tab highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.2.png)

   1. Choose **Edit**\.

      To enable port 8883, you add a custom TCP rule to the security group\. For more information, see [ Adding Rules to a Security Group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#adding-security-group-rule) in the *Amazon EC2 User Guide for Linux Instances*\.

   1. On the **Edit inbound rules** page, choose **Add Rule**, enter the following settings, and then choose **Save**\.
      + For **Type**, choose **Custom TCP Rule**\.
      + For **Port Range**, enter **8883**\.
      + For **Source**, choose **Anywhere**\.
      + For **Description**, enter **MQTT Communications**\.

         
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.3.png)

1. Connect to your Amazon EC2 instance\.

   1. In the navigation pane, choose **Instances**, choose your instance, and then choose **Connect**\.

   1. Follow the instructions on the **Connect To Your Instance** page to connect to your instance [ by using SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html) and your private key file\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.4.png)

   You can use [PuTTY](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html) for Windows or Terminal for macOS\. For more information, see [ Connect to Your Linux Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html) in the *Amazon EC2 User Guide for Linux Instances*\.

1. After you are connected to your Amazon EC2 instance, create user `ggc_user` and group `ggc_group`:

   ```
   sudo adduser --system ggc_user
   sudo groupadd --system ggc_group
   ```

1. To improve security on the device, enable hardlink and softlink protection on the operating system at start up\.

   1. Navigate to the `00-defaults.conf` file\.

      ```
      cd /etc/sysctl.d
      ls
      ```

   1. Using your favorite text editor \(Leafpad, GNU nano, or vi\), add the following two lines to the end of the `00-defaults.conf` file\. You might need to change permissions \(using the `chmod` command\) to write to the file, or use the `sudo` command to edit as root \(for example, `sudo nano 00-defaults.conf`\)\.

      ```
      fs.protected_hardlinks = 1
      fs.protected_symlinks = 1
      ```

   1. Reboot the Amazon EC2 instance\.

      ```
      sudo reboot
      ```

      After a few minutes, connect to your instance using SSH and then run the following command to confirm the change\.

      ```
      sudo sysctl -a | grep fs.protected
      ```

      You should see that hardlinks and softlinks are set to 1\.

1. Extract and run the following script to mount [Linux control groups](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01) \(**cgroups**\)\. This allows AWS IoT Greengrass to set the memory limit for Lambda functions\. Cgroups are also required to run AWS IoT Greengrass in the default [containerization](lambda-group-config.md#lambda-containerization-considerations) mode\.

   ```
   curl https://raw.githubusercontent.com/tianon/cgroupfs-mount/951c38ee8d802330454bdede20d85ec1c0f8d312/cgroupfs-mount > cgroupfs-mount.sh
   chmod +x cgroupfs-mount.sh 
   sudo bash ./cgroupfs-mount.sh
   ```

   Your Amazon EC2 instance should now be ready for AWS IoT Greengrass\.

1. To make sure that you have all required dependencies, download and run the Greengrass dependency checker from the [AWS IoT Greengrass Samples](https://github.com/aws-samples/aws-greengrass-samples) repository on GitHub\. These commands unzip and run the dependency checker script in the current directory\.

   ```
   mkdir greengrass-dependency-checker-GGCv1.9.0
   cd greengrass-dependency-checker-GGCv1.9.0
   wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.9.0.zip
   unzip greengrass-dependency-checker-GGCv1.9.0.zip
   sudo ./check_ggc_dependencies | more
   ```
**Important**  
This tutorial requires Python 2\.7\. The `check_ggc_dependencies` script might produce warnings about the missing optional Node\.js and Java prerequisites\. You can ignore these warnings\.

Your Amazon EC2 instance configuration is complete\. Continue to [Module 2: Installing the Greengrass Core Software](module2.md)\.

## Setting Up Other Devices<a name="setup-filter.other"></a>

If you're new to AWS IoT Greengrass, we recommend that you use a Raspberry Pi or an Amazon EC2 instance as your core device, and follow the setup steps in the corresponding section\. To use a different platform, follow the steps in this section\. For information about supported device platforms, see [Greengrass Core Platform Compatibility](https://aws.amazon.com/greengrass/faqs/)\.

1. <a name="setup-jetson"></a>If your core device is an NVIDIA Jetson TX2, you must first flash the firmware with the JetPack 3\.3 installer\. If you're configuring a different device, skip to step 2\.
**Note**  
The JetPack installer version that you use is based on your target CUDA Toolkit version\. The following instructions use JetPack 3\.3 and CUDA Toolkit 9\.0 because the TensorFlow v1\.10\.1 and MXNet v1\.2\.1 binaries \(that AWS IoT Greengrass provides for machine learning inference on a Jetson TX2\) are compiled against this version of CUDA\. For more information, see [Perform Machine Learning Inference](ml-inference.md)\.

   1. On a physical desktop that is running Ubuntu 16\.04 or later, flash the firmware with the JetPack 3\.3 installer, as described in [Download and Install JetPack](https://docs.nvidia.com/jetson/archives/jetpack-archived/jetpack-33/index.html#jetpack/3.3/install.htm%3FTocPath%3D_____3) \(3\.3\) in the NVIDIA documentation\.

      Follow the instructions in the installer to install all the packages and dependencies on the Jetson board, which must be connected to the desktop with a Micro\-B cable\.

   1. Reboot your board in normal mode, and connect a display to the board\.
**Note**  
When you use SSH to connect to the Jetson board, use the default user name \(**nvidia**\) and the default password \(**nvidia**\)\.

1. Run the following commands to create user `ggc_user` and group `ggc_group`\.

   ```
   sudo adduser --system ggc_user
   sudo addgroup --system ggc_group
   ```
**Note**  
If the `addgroup` command isn't available on your system, use the following command\.  

   ```
   sudo groupadd --system ggc_group
   ```

1. To make sure that you have all required dependencies, download and run the Greengrass dependency checker from the [AWS IoT Greengrass Samples](https://github.com/aws-samples/aws-greengrass-samples) repository on GitHub\. These commands unzip and run the dependency checker script in the current directory\.

   ```
   mkdir greengrass-dependency-checker-GGCv1.9.0
   cd greengrass-dependency-checker-GGCv1.9.0
   wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.9.0.zip
   unzip greengrass-dependency-checker-GGCv1.9.0.zip
   sudo ./check_ggc_dependencies | more
   ```
**Note**  
The `check_ggc_dependencies` script runs on AWS IoT Greengrass supported platforms and requires the following Linux system commands: `printf`, `uname`, `cat`, `ls`, `head`, `find`, `zcat`, `awk`, `sed`, `sysctl`, `wc`, `cut`, `sort`, `expr`, `grep`, `test`, `dirname`, `readlink`, `xargs`, `strings`, `uniq`\.  
For more information, see the dependency checker's [Readme](https://github.com/aws-samples/aws-greengrass-samples/blob/master/greengrass-dependency-checker-GGCv1.9.0/README.md)\.

1. Install all required dependencies on your device, as indicated by the dependency checker output\. For missing kernel\-level dependencies, you might have to recompile your kernel\. For mounting Linux control groups \(`cgroups`\), you can run the [cgroupfs\-mount](https://raw.githubusercontent.com/tianon/cgroupfs-mount/master/cgroupfs-mount) script\. This allows AWS IoT Greengrass to set the memory limit for Lambda functions\. Cgroups are also required to run AWS IoT Greengrass in the default [containerization](lambda-group-config.md#lambda-containerization-considerations) mode\.

   If no errors appear in the output, AWS IoT Greengrass should be able to run successfully on your device\.
**Important**  
This tutorial requires Python 2\.7\. The `check_ggc_dependencies` script might produce warnings about the missing optional Node\.js and Java prerequisites\. You can ignore these warnings\.

   For the list of AWS IoT Greengrass requirements and dependencies, see [Supported Platforms and Requirements](what-is-gg.md#gg-platforms)\.