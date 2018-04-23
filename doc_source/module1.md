# Module 1: Environment Setup for Greengrass<a name="module1"></a>

This module shows you how to get an out\-of\-the\-box Raspberry Pi, Amazon EC2 instance, or other device ready to be used by AWS Greengrass\.

**Important**  
Use the **Filter View** drop\-down list in the upper\-right corner of this webpage to choose your platform\.

This module should take less than 30 minutes to complete\.

## Setting Up a Raspberry Pi<a name="setup-filter.rpi"></a>

If you are setting up a Raspberry Pi for the first time, you must follow all of these steps\. If you are using an existing Raspberry Pi, you can skip to step 9\. However, we recommend that you re\-image your Raspberry Pi with the operating system as recommended in step 2\.

1. Download and install an SD card formatter such as [SD Memory Card Formatter](https://www.sdcard.org/downloads/formatter_4/index.html) or [PiBakery](http://www.pibakery.org/download.html)\. Insert the SD card into your computer\. Start the program and choose the drive where your have inserted your SD card\. You can quick format the SD card\.

1. Download the [Raspbian Jessie](https://downloads.raspberrypi.org/raspbian/images/raspbian-2017-03-03/) operating system as a `.zip` file\. Only `2017-03-02-raspbian-jessie.zip` is currently supported by AWS Greengrass\.

1. Using an SD card\-writing tool \(such as [Etcher](https://etcher.io/)\), follow the tool's instructions to flash the downloaded `2017-03-02-raspbian-jessie.zip` file onto the SD card\. Because the operating system image is large, this step might take some time\. Eject your SD card from your computer, and insert the microSD card into your Raspberry Pi\.

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
   cd greengrass-dependency-checker-GGCv1.5.0
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
   cd greengrass-dependency-checker-GGCv1.5.0
   sudo ./check_ggc_dependencies
   ```

Your Amazon EC2 instance configuration is complete\. Continue to [Module 2: Installing the Greengrass Core Software](module2.md)\.

## Setting Up Other Devices<a name="setup-filter.other"></a>

If you are new to AWS Greengrass, we recommend that you use a Raspberry Pi or an Amazon EC2 instance and follow the steps provided above to set up the device\. Follow the below steps to make your own AWS Greengrass\-supported device ready for AWS Greengrass\.

1. To make sure you have other devices ready to run AWS Greengrass, download and extract the Greengrass dependency checker from the [GitHub repository](https://github.com/aws-samples/aws-greengrass-samples), and then run the following commands:

   ```
   git clone https://github.com/aws-samples/aws-greengrass-samples.git
   cd greengrass-dependency-checker-GGCv1.5.0 
   sudo ./check_ggc_dependencies
   ```

   This script runs on AWS Greengrass supported platforms and requires the following Linux system commands:

   ```
   printf, uname, cat, ls, head, find, zcat, awk, sed, sysctl, wc, cut, sort, expr, grep, test, dirname, readlink, xargs, strings, uniq
   ```

1. Install all required dependencies on your device, as indicated by the dependency script\. For missing kernel\-level dependencies, you might have to recompile your kernel\. For mounting Linux control groups \(`cgroups`\), you can run the [cgroupfs\-mount](https://raw.githubusercontent.com/tianon/cgroupfs-mount/master/cgroupfs-mount) script\.
**Note**  
If no errors appear in the output, AWS Greengrass should be able to run successfully on your device\.

   For the list of AWS Greengrass requirements and dependencies, see [Supported Platforms and Requirements](what-is-gg.md#gg-platforms)\.

### Configuring NVIDIA Jetson TX2 for AWS Greengrass<a name="setup-jetson"></a>

If your core device is an NVIDIA Jetson TX2, it must be configured before you can install the AWS Greengrass\. The following steps describe how to flash the firmware to a JetPack installer and rebuild the kernel so that the device is ready to install the AWS Greengrass core software\.

**Note**  
The JetPack installer version that you use is based on your target CUDA Toolkit version\. The following instructions assume that you're using JetPack 3\.1 and CUDA Toolkit 8\.0, because the binaries for TensorFlow v1\.4\.0 that AWS Greengrass provides for machine learning \(ML\) inference are compiled against this version of CUDA\. For more information about AWS Greengrass ML inference, see [Perform Machine Learning Inference](ml-inference.md)\.

#### Flash the JetPack 3\.1 Firmware<a name="jetson-flash-firmware"></a>

1. On a physical desktop that is running Ubuntu 14 or 16, flash the firmware to JetPack 3\.1, as described in [Download and Install JetPack L4T](http://docs.nvidia.com/jetpack-l4t/index.html#developertools/mobile/jetpack/l4t/3.0/jetpack_l4t_install.htm)\.

   Follow the instructions in the installer to install all the packages and dependencies on the Jetson board, which must be connected to the desktop with a Micro\-B cable\. Start the device in forced recovery mode\.
**Note**  
After the JetPack installation, you must use *ubuntu* credentials to log onto the device\. The SSH agent hangs when it tries to log in using any other account, even if you SSH directly to the board using this account\.

1. Reboot your board in normal mode, and then connect a display to the board\.

#### Rebuild the NVIDIA Jetson TX2 Kernel<a name="jetson-kernel-config"></a>

Run the following commands on the Jetson board\.

1. Check the kernel configurations:

   ```
   nvidia@tegra-ubuntu:~$ zcat /proc/config.gz | grep -e CONFIG_KEYS -e CONFIG_POSIX_MQUEUE -e CONFIG_OF_OVERLAY -e CONFIG_OVERLAY_FS -e CONFIG_HAVE_ARCH_SECCOMP_FILTER -e CONFIG_SECCOMP_FILTER -e CONFIG_SECCOMP -e CONFIG_DEVPTS_MULTIPLE_INSTANCES -e CONFIG_IPC_NS -e CONFIG_NET_NS -e CONFIG_UTS_NS -e CONFIG_USER_NS -e CONFIG_PID_NS -e CONFIG_CGROUPS –e CONFIG_MEMCG -e CONFIG_CGROUP_FREEZER -e CONFIG_CGROUP_DEVICE
   # CONFIG_POSIX_MQUEUE is not set
   CONFIG_CGROUPS=y
   CONFIG_CGROUP_FREEZER=y
   # CONFIG_CGROUP_DEVICE is not set
   # CONFIG_MEMCG is not set
   CONFIG_UTS_NS=y
   CONFIG_IPC_NS=y
   # CONFIG_USER_NS is not set
   CONFIG_PID_NS=y
   CONFIG_NET_NS=y
   CONFIG_HAVE_ARCH_SECCOMP_FILTER=y
   CONFIG_SECCOMP_FILTER=y
   CONFIG_SECCOMP=y
   # CONFIG_OF_OVERLAY is not set
   CONFIG_DEVPTS_MULTIPLE_INSTANCES=y
   # CONFIG_OVERLAY_FS is not set
   # CONFIG_KEYS is not set
   ```

1. Check the performance and power settings:

   ```
   nvidia@tegra-ubuntu:~$ sudo nvpmodel –q
   NV Power Mode: MAXP_CORE_ARM
   3
   ```

1. Put the Jetson into high performance mode:

   ```
   nvidia@tegra-ubuntu:~$ sudo nvpmodel –m 0
   ```

1. Clone the git repository:

   ```
   nvidia@tegra-ubuntu:~$ cd /
   nvidia@tegra-ubuntu:~$ sudo git clone https://github.com/jetsonhacks/buildJetsonTX2Kernel.git
   ```

1. Modify the `getKernelSources.sh` script, based on the following diff of the changes:

   ```
   index f47f28d..3dd863a 100755
   --- a/scripts/getKernelSources.sh
   +++ b/scripts/getKernelSources.sh
   @@ -1,12 +1,15 @@
    #!/bin/bash
    apt-add-repository universe
    apt-get update
   -apt-get install qt5-default pkg-config -y
   +apt-get install qt5-default pkg-config libncurses5-dev libssl-dev -y
    cd /usr/src
    wget http://developer.download.nvidia.com/embedded/L4T/r28_Release_v1.0/BSP/source_release.tbz2
    tar -xvf source_release.tbz2 sources/kernel_src-tx2.tbz2
    tar -xvf sources/kernel_src-tx2.tbz2
    cd kernel/kernel-4.4
   +make clean
    zcat /proc/config.gz > .config
   -make xconfig
   +echo "type something to continue"
   +read
   +make menuconfig
   ```

1. Run the `getKernelSources` script:

   ```
   nvidia@tegra-ubuntu:~$ cd /buildJetsonTX2Kernel
   nvidia@tegra-ubuntu:~$ sudo ./getKernelSources.sh
   ```

1. When prompted for "type something to continue", press **CTRL \+ Z** to background the script\.

1. Go to /usr/src/kernel/kernel\-4\.4/security/keys and edit the `Kconfig` file by adding the following lines between `KEYS` and `PERSISTENT_KEYRINGS`:

   ```
   config KEYS_COMPAT
       def_bool y
       depends on COMPAT && KEYS
   ```

1. Unpause the script:

   ```
   nvidia@tegra-ubuntu:~$ cd /usr/src/kernel/kernel-4.4/
   nvidia@tegra-ubuntu:~$ fg
   ```

   Type some characters to unblock the script\.

1. In the setup window that opens, choose **Enable loadable module support**, and then open the submenu to enable **optionModule** signature verification\. Use the arrow keys to move and the spacebar to select any option\. Then, save the change and exit\.

1. Verify that `KEYS_COMPAT` is enabled:

   ```
   nvidia@tegra-ubuntu:~$ grep --color KEYS_COMPAT /usr/src/kernel/kernel-4.4/.config
   ```

1. Open the kernel configuration interface and enable kernel configurations:

   ```
   nvidia@tegra-ubuntu:~$ sudo make xconfig
   ```

   A window opens that shows all the kernel configurations\. Use **FIND** to search for the following keywords and tick\-mark them\.
**Note**  
Keywords vary by configuration\. The following list contains alternative versions in parentheses that can help you find the equivalent keywords for your configuration\.
   + CONFIG\_POSIX\_MQUEUE  \(POSIX Message Queue\)
   + CONFIG\_OF\_OVERLAY  \(Overlay Filesystem Support\)
   + CONFIG\_OVERLAY\_FS  \(Overlay Filesystem Support\)
   + CONFIG\_USER\_NS  \(User Namespace\)
   + CONFIG\_MEMCG  \(Memory Resource Controller for Control Group\)
   + CONFIG\_CGROUP\_DEVICE  \(Device Controller for cgroups\)

1. Build the kernel:

   ```
   nvidia@tegra-ubuntu:~$ cd /buildJetsonTX2Kernel
   nvidia@tegra-ubuntu:~$ sudo ./makeKernel.sh
   ```

1. Verify that the kernel configurations are enabled:

   ```
   nvidia@tegra-ubuntu:~$ grep --color CONFIG_POSIX_MQUEUE /usr/src/kernel/kernel-4.4/.config
   nvidia@tegra-ubuntu:~$ grep --color CONFIG_OF_OVERLAY /usr/src/kernel/kernel-4.4/.config
   nvidia@tegra-ubuntu:~$ grep --color CONFIG_OVERLAY_FS /usr/src/kernel/kernel-4.4/.config
   nvidia@tegra-ubuntu:~$ grep --color CONFIG_USER_NS /usr/src/kernel/kernel-4.4/.config
   nvidia@tegra-ubuntu:~$ grep --color CONFIG_MEMCG /usr/src/kernel/kernel-4.4/.config
   nvidia@tegra-ubuntu:~$ grep --color CONFIG_CGROUP_DEVICE /usr/src/kernel/kernel-4.4/.config
   nvidia@tegra-ubuntu:~$ grep --color CONFIG_KEYS_COMPAT /usr/src/kernel/kernel-4.4/.config
   nvidia@tegra-ubuntu:~$ grep --color CONFIG_COMPAT /usr/src/kernel/kernel-4.4/.config
   nvidia@tegra-ubuntu:~$ grep --color CONFIG_KEYS /usr/src/kernel/kernel-4.4/.config
   ```

1. Copy the image:

   ```
   nvidia@tegra-ubuntu:~$ sudo ./copyImage.sh
   ```