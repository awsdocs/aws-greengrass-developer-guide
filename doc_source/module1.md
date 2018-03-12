# Module 1: Environment Setup for Greengrass<a name="module1"></a>

This module shows you how to get an out\-of\-the\-box Raspberry Pi, Amazon EC2 instance, or other device ready to be used by AWS Greengrass\.

**Important**  
Use the **Filter View** drop\-down list in the upper\-right corner of this webpage to choose your platform\.

This module should take less than 30 minutes to complete\.

## Setting Up a Raspberry Pi<a name="setup-filter.rpi"></a>

If you are setting up a Raspberry Pi for the first time, you must follow all of these steps\. If you are using an existing Raspberry Pi, you can skip to step 9\. However, we recommend that you re\-image your Raspberry Pi with the operating system as recommended in step 2\.

1. Download and install an SD card formater such as [SD Memory Card Formatter](https://www.sdcard.org/downloads/formatter_4/index.html) or [PiBakery](http://www.pibakery.org/download.html)\. Insert the SD card into your computer\. Start the program and choose the drive where your have inserted your SD card\. You can quick format the SD card\.

1. Download the [Raspbian Jessie](https://downloads.raspberrypi.org/raspbian/images/raspbian-2017-03-03/) operating system as a `.zip` file \(only 2017\-03\-02\-raspbian\-jessie\.zip is currently supported by AWS Greengrass\)\. Using [Etcher](https://etcher.io/) or another SD card\-writing tool, flash the downloaded `.zip` file onto the SD card \(do not unzip this file\)\. Because the operating system image is large, this step might take some time\. Eject your SD card from your computer, and install the microSD card onto your Raspberry Pi\.

1. For the first boot, we recommend that you connect the Raspberry Pi to a monitor \(through HDMI\), a keyboard, and a mouse\. Next, connect your Pi to a micro USB power source and the Raspbian operating system should start up\. 

1. You may want to configure the Pi's keyboard layout before proceeding\. To do so, choose the Raspberry icon in the upper\-right, choose **Preferences** and then **Mouse and Keyboard Settings**\. Next, choose the **Keyboard** tab, **Keyboard Layout**, and then choose an appropriate keyboard variant\.

1. Next, [connect your Raspberry Pi to the internet through a Wi\-Fi network](https://www.raspberrypi.org/documentation/configuration/wireless/desktop.md) or an Ethernet cable\.
**Note**  
Connect your Raspberry Pi to the *same* network that your computer is connected to, and be sure that both your computer and Raspberry Pi have internet access before proceeding\. If you're in a work environment or behind a firewall, you may need to connect your Pi and your computer to the guest network in order to get both devices on the same network\. This approach, however, may disconnect your computer from local network resources such as your intranet\. One solution may be to connect the Pi to the guest Wi\-Fi network, your computer to the guest Wi\-Fi network *and* your local network through an Ethernet cable\. In this configuration, your computer should be able to connect to the Raspberry Pi via the guest Wi\-Fi network and your local network resources through the Ethernet cable\.

1. You must set up [SSH](https://en.wikipedia.org/wiki/Secure_Shell) on your Pi to remotely connect to it\. On your Raspberry Pi, open a [terminal window](https://www.raspberrypi.org/documentation/usage/terminal/) and run the following command:

   ```
   sudo raspi-config
   ```

   You should see the following:  
![\[Raspberry Pi Software Configuration Tool (raspi-config) screenshot.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-001.png)

   Scroll down and choose **Interfacing Options** and then choose **P2 SSH**\. When prompted, choose **Yes** using the Tab key \(followed by Enter\)\. SSH should now be enabled, choose **OK**\. Tab key to **Finish** and then press the Enter key\. Lastly, reboot your Pi by running the following command:

   ```
   sudo reboot
   ```

1. On your Raspberry Pi, run the following command in the terminal:

   ```
   hostname -I
   ```

   This returns the IP address of your Raspberry Pi\.
**Note**  
For the following, if you receive an ECDSA key fingerprint related message `Are you sure you want to continue connecting (yes/no)?`, enter `yes`\. Additionally, the default password for the Raspberry Pi is **raspberry**\.

   If you are using macOS, open a Terminal window and type the following:

   ```
   ssh pi@IP-address
   ```

   Here, *IP\-address* corresponds to the IP address of your Raspberry Pi that you obtained by using the prior `hostname -I` command\.

   If you are using Windows, you need to install and configure [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)\. Choose **Connection**, **Data**, and make sure that **Prompt** is selected:   
![\[PuTTY window with Prompt selected.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-001.4.png)

   Next, choose **Session**, type the IP address of the Raspberry Pi, and choose **Open** using default settings\. For example \(your IP address, in all likelihood, will be different\):  
![\[PuTTY window with IP address in the "Host Name (or IP address)" field.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-001.5.png)

   If a PuTTY Security Alert dialog is displayed, choose an appropriate response such as **Yes**\.

   This will result in a terminal window similar to the following\. The default Raspberry Pi login and password are **pi** and **raspberry**, respectively\.  
![\[Initial PuTTY terminal window.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-001.6.png)
**Note**  
If your computer is connected to a remote network using VPN \(such as a work related network\), this may cause difficulty connecting from the computer to the Raspberry Pi using SSH\.

1. You are now ready to set up the Raspberry Pi for AWS Greengrass\. First, run the following commands from a local Raspberry Pi terminal window or an SSH terminal window:

   ```
   sudo adduser --system ggc_user
   sudo addgroup --system ggc_group
   ```

   Next, run the following commands to install `sqlite3`:

   ```
   sudo apt-get update
   sudo apt-get install sqlite3
   ```

1. Run the following commands to update the Linux kernel version of your Raspberry Pi\. 

   ```
   sudo apt-get install rpi-update
   sudo rpi-update b81a11258fc911170b40a0b09bbd63c84bc5ad59
   ```

   Although several kernel versions might work with AWS Greengrass, for the best security and performance, we recommend that you use the kernel version indicated in step 2\. In order to activate the new firmware, reboot your Raspberry Pi:

   ```
   sudo reboot
   ```

   As applicable, reconnect to the Raspberry Pi using SSH after minute or so\. Next, run the following command to ensure you have the correct kernel version:

   ```
   uname -a
   ```

   You should receive output similar to the following, the key item being the Linux Raspberry Pi version information `4.9.30`:  
![\[Raspberry Pi "uname -a" command output showing kernel version.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.5.png)

1. To improve security on the Pi device, run the following commands to enable hardlink and softlink protection at operating system start\-up\. 

   ```
   cd /etc/sysctl.d
   ls
   ```

   If you see the `98-rpi.conf` file, use a text editor \(such as `leafpad`, `nano`, or `vi`\) to add the following two lines to the end of the file \(you can run the text editor using the `sudo` command to avoid write permission issues, as in `sudo nano 98-rpi.conf`\)\.

   ```
   fs.protected_hardlinks = 1
   fs.protected_symlinks = 1
   ```

   If you do not see the `98-rpi.conf` file, follow the instructions in the `README.sysctl` file\. 

   Now reboot the Pi:

   ```
   sudo reboot
   ```

   After about a minute, connect to the Pi using SSH as applicable and then run the following commands from a Raspberry Pi terminal to confirm the hardlink/symlink change:

   ```
   sudo sysctl -a 2> /dev/null | grep fs.protected
   ```

   You should see `fs.protected_hardlinks = 1` and `fs.protected_symlinks = 1`\.

1. Your Raspberry Pi should now be ready for AWS Greengrass\. To ensure that you have all of the dependencies required for AWS Greengrass, download the AWS Greengrass dependency checker `.zip` file from the [GitHub repository](https://github.com/aws-samples/aws-greengrass-samples) and run it on the Pi as follows:

   ```
   cd /home/pi/Downloads
   git clone https://github.com/aws-samples/aws-greengrass-samples.git
   cd aws-greengrass-samples
   cd greengrass-dependency-checker-GGCv1.3.0
   sudo modprobe configs
   sudo ./check_ggc_dependencies | more
   ```

   With respect to the `more` command, press the Spacebar key to display another screen of text\. 
**Important**  
Because this tutorial only uses the AWS IoT Device SDK for Python, you can ignore warnings about the missing optional NodeJS 6\.10 and Java 8 prerequisites that the `check_ggc_dependencies` script may produce\.

   For information about the `modprobe` command, you can run `man modprobe` in the terminal\. 

Your Raspberry Pi configuration is complete\. Continue to [Module 2: Installing the Greengrass Core Software](module2.md)\.

## Setting Up an Amazon EC2 Instance<a name="setup-filter.ec2"></a>

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/) and launch an Amazon EC2 instance using an Amazon Linux AMI \(Amazon Machine Image\)\. For information about Amazon EC2 instances, see the [Amazon EC2 Getting Started Guide](http://docs.aws.amazon.com/AWSEC2/latest/GettingStartedGuide/)\.

1. After your Amazon EC2 instance is running, enable port 8883 to allow incoming MQTT communications so that other devices can connect with the AWS Greengrass core\. In the left pane of the Amazon EC2 console, choose **Security Groups**\.  
![\[Left pane with Security Groups highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.1.png)

   Choose the instance that you just launched, and then choose the **Inbound** tab\.  
![\[Inbound tab highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.2.png)

   By default, only one port for SSH is enabled\. To enable port 8883, choose the **Edit** button\. Next, choose the **Add Rule** button and create a custom TCP rule as shown below, then choose **Save**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.3.png)

1. In the left pane, choose **Instances**, choose your instance, and then choose the **Connect** button\. Connect to your Amazon EC2 instance by using SSH\. You can use [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) for Windows or Terminal for macOS\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.4.png)

1. Once connected to your Amazon EC2 instance through SSH, run the following commands to create user `ggc_user` and group `ggc_group`:

   ```
   sudo adduser --system ggc_user
   sudo groupadd --system ggc_group
   ```

1. To improve security on the device, enable hardlink/softlink protection on the operating system at start\-up\. To do so, run the following commands:

   ```
   cd /etc/sysctl.d
   ls
   ```

   Using your favorite text editor \(`leadpad`, `nano`, `vi`, etc\.\), add the following two lines to the end of the `00-defaults.conf` file, You might need to change permissions \(using the `chmod` command\) to write to the file, or use the `sudo` command to edit as root \(for example, `sudo nano 00-defaults.conf`\)\.

   ```
   fs.protected_hardlinks = 1
   fs.protected_symlinks = 1
   ```

   Run the following command to reboot the Amazon EC2 instance\.

   ```
   sudo reboot
   ```

   After a few minutes, connect to your instance by using SSH as above\. Then, run the following command to confirm the change\.

   ```
   sudo sysctl -a | grep fs.protected
   ```

   You should see that hardlinks and softlinks are set to 1\.

1. Extract and run the following script to mount [Linux control groups](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01) \(**cgroups**\)\. This is an AWS Greengrass dependency:

   ```
   curl https://raw.githubusercontent.com/tianon/cgroupfs-mount/951c38ee8d802330454bdede20d85ec1c0f8d312/cgroupfs-mount > cgroupfs-mount.sh
   chmod +x cgroupfs-mount.sh 
   sudo bash ./cgroupfs-mount.sh
   ```

   Your Amazon EC2 instance should now be ready for AWS Greengrass\. To be sure that you have all of the dependencies, extract and run the following AWS Greengrass dependency script from the [GitHub repository](https://github.com/aws-samples/aws-greengrass-samples):

   ```
   sudo yum install git
   git clone https://github.com/aws-samples/aws-greengrass-samples.git
   cd aws-greengrass-samples
   cd greengrass-dependency-checker-GGCv1.3.0 
   sudo ./check_ggc_dependencies
   ```

Your Amazon EC2 instance configuration is complete\. Continue to [Module 2: Installing the Greengrass Core Software](module2.md)\.

## Setting Up Other Devices<a name="setup-filter.other"></a>

If you are new to AWS Greengrass, we recommend that you use a Raspberry Pi or an Amazon EC2 instance and follow the steps provided above to set up the device\. Follow the below steps to make your own AWS Greengrass\-supported device ready for AWS Greengrass\.

1. To make sure you have other devices ready to run AWS Greengrass, download and extract the Greengrass dependency checker from the [GitHub repository](https://github.com/aws-samples/aws-greengrass-samples), and then run the following commands:

   ```
   git clone https://github.com/aws-samples/aws-greengrass-samples.git
   cd greengrass-dependency-checker-GGCv1.3.0 
   sudo ./check_ggc_dependencies
   ```

   This script runs on AWS Greengrass supported platforms and requires the following Linux system commands:

   ```
   printf, uname, cat, ls, head, find, zcat, awk, sed, sysctl, wc, cut, sort, expr, grep, test, dirname, readlink, xargs, strings, uniq
   ```

1. Install all required dependencies on your device, as indicated by the dependency script\. For missing kernel\-level dependencies, you might have to recompile your kernel\. For mounting Linux control groups \(`cgroups`\), you can run the [cgroupfs\-mount](https://raw.githubusercontent.com/tianon/cgroupfs-mount/master/cgroupfs-mount) script\.
**Note**  
If no errors appear in the output, AWS Greengrass should be able to run successfully on your device\.

These are the AWS Greengrass dependencies:

**Required items:**

+ For best security and performance, Linux kernel version 4\.4 or later\.

+ `libc` C library, version 2\.14 or later\.

+ The `/var/run` directory must be present on the device\. 

+ Hardlink/softlink protection must be enabled on the device\. Otherwise, AWS Greengrass can only be run in insecure mode, using the `-i` flag\.

+ `ggc_user` and `ggc_group` must be present on the device\.

+ The following Linux kernel configurations must be enabled on the device:

  + Namespace: `CONFIG_IPC_NS`, `CONFIG_UTS_NS`, `CONFIG_USER_NS`, `CONFIG_PID_NS` 

  + Cgroups: `CONFIG_CGROUP_DEVICE`, `CONFIG_CGROUPS, CONFIG_MEMCG`

  + Others: `CONFIG_POSIX_MQUEUE`, `CONFIG_OVERLAY_FS`, `CONFIG_HAVE_ARCH_SECCOMP_FILTER`, `CONFIG_SECCOMP_FILTER`, `CONFIG_KEYS`, `CONFIG_SECCOMP` 

+ The `sqlite3` package is required for AWS IoT device shadows\. Make sure it is added to your `PATH` environment variable\.

+ `/dev/stdin`, `/dev/stdout`, and `/dev/stderr` must be enabled\.

+ The Linux kernel must support cgroups\.

+ The *memory* `cgroup` must be enabled and mounted to allow AWS Greengrass to set the memory limit for Lambda functions\.

+ The root certificate for Amazon S3 and AWS IoT must be present in the system trust store\.

**Optional or feature\-dependent items:**

+ The *devices* `cgroup` must be enabled and mounted if Lambda functions with local resource access are used to open device files\.

+ If you are using Python Lambda functions, Python 2\.7 is required\. Make sure it is added to your `PATH` environment variable\.

+ If you are using Node\.JS Lambda functions, Node\.JS 6\.10 or later is required\. Make sure it is added to your `PATH` environment variable\.

+ If you are using Java Lambda functions, Java 8 or later is required\. Make sure it is added to your `PATH` environment variable\.

+ OpenSSL 1\.0\.1 or later and the following commands are required for the over\-the\-air \(OTA\) agent: `wget`, `realpath`, `tar`, `readlink`, `basename`, `dirname`, `pidof`, `df`, `grep`, and `umount`\.