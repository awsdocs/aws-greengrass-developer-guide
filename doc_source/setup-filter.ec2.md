# Setting up an Amazon EC2 instance<a name="setup-filter.ec2"></a>

Follow the steps in this topic to set up an Amazon EC2 instance to use as your AWS IoT Greengrass core\.

**Tip**  
Or, to use a script that sets up your environment and installs the AWS IoT Greengrass Core software for you, see [Quick start: Greengrass device setup](quick-start.md)\.

 Although you can complete this tutorial using an Amazon EC2 instance, AWS IoT Greengrass should ideally be used with physical hardware\. We recommend that you [set up a Raspberry Pi](setup-filter.rpi.md) instead of using an Amazon EC2 instance when possible\. If you're using a Raspberry Pi, you do not need to follow the steps in this topic\. 

 

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/) and launch an Amazon EC2 instance using an Amazon Linux AMI\. For information about Amazon EC2 instances, see the [Amazon EC2 Getting Started Guide](https://docs.aws.amazon.com/AWSEC2/latest/GettingStartedGuide/)\.

1. After your Amazon EC2 instance is running, enable port 8883 to allow incoming MQTT communications so that other devices can connect with the AWS IoT Greengrass core\.

   1. In the navigation pane of the Amazon EC2 console, choose **Security Groups**\.  
![\[Navigation pane with Security Groups highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.1.png)

   1. Select the security group for the instance that you just launched, and then choose the **Inbound** tab\.  
![\[Inbound tab highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-002.6.2.png)

   1. Choose **Edit**\.

      To enable port 8883, you add a custom TCP rule to the security group\. For more information, see [ Adding rules to a security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#adding-security-group-rule) in the *Amazon EC2 User Guide for Linux Instances*\.

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

   You can use [PuTTY](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html) for Windows or Terminal for macOS\. For more information, see [ Connect to your Linux instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html) in the *Amazon EC2 User Guide for Linux Instances*\.

   You are now ready to set up your Amazon EC2 instance for AWS IoT Greengrass\.

1. After you are connected to your Amazon EC2 instance, create the `ggc_user` and `ggc_group` accounts:

   ```
   sudo adduser --system ggc_user
   sudo groupadd --system ggc_group
   ```
**Note**  
If the `adduser` command isn't available on your system, use the following command\.  

   ```
   sudo useradd --system ggc_user
   ```

1. To improve security, make sure that hardlink and softlink \(symlink\) protections are enabled on the operating system of the Amazon EC2 instance at startup\.
**Note**  
 The steps for enabling hardlink and softlink protection vary by operating system\. Consult the documentation for your distribution\. 

   1.  Run the following command to check if hardlink and softlink protections are enabled: 

      ```
      sudo sysctl -a | grep fs.protected
      ```

       If hardlinks and softlinks are set to `1`, your protections are enabled correctly\. Proceed to step 6\. 
**Note**  
Softlinks are represented by `fs.protected_symlinks`\.

   1. If hardlinks and softlinks are not set to `1`, enable these protections\. Navigate to your system configuration file\. 

      ```
      cd /etc/sysctl.d
      ls
      ```

   1. Using your favorite text editor \(Leafpad, GNU nano, or vi\), add the following two lines to the end of the system configuration file\. On Amazon Linux 1, this is the `00-defaults.conf` file\. On Amazon Linux 2, this is the `99-amazon.conf` file\. You might need to change permissions \(using the `chmod` command\) to write to the file, or use the `sudo` command to edit as root \(for example, `sudo nano 00-defaults.conf`\)\.

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

1. Extract and run the following script to mount [Linux control groups](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01) \(cgroups\)\. This allows AWS IoT Greengrass to set the memory limit for Lambda functions\. Cgroups are also required to run AWS IoT Greengrass in the default [containerization](lambda-group-config.md#lambda-containerization-considerations) mode\.

   ```
   curl https://raw.githubusercontent.com/tianon/cgroupfs-mount/951c38ee8d802330454bdede20d85ec1c0f8d312/cgroupfs-mount > cgroupfs-mount.sh
   chmod +x cgroupfs-mount.sh 
   sudo bash ./cgroupfs-mount.sh
   ```

   Your Amazon EC2 instance should now be ready for AWS IoT Greengrass\.

1. <a name="install-java-8-runtime"></a>Optional\. Install the Java 8 runtime, which is required by [stream manager](stream-manager.md)\. This tutorial doesn't use stream manager, but it does use the **Default Group creation** workflow that enables stream manager by default\. Use the following commands to install the Java 8 runtime on the core device, or disable stream manager before you deploy your group\. Instructions for disabling stream manager are provided in Module 3\.
   + For Debian\-based distributions:

     ```
     sudo apt install openjdk-8-jdk
     ```
   + For Red Hat\-based distributions:

     ```
     sudo yum install java-1.8.0-openjdk
     ```

1. To make sure that you have all required dependencies, download and run the Greengrass dependency checker from the [AWS IoT Greengrass Samples](https://github.com/aws-samples/aws-greengrass-samples) repository on GitHub\. These commands download, unzip, and run the dependency checker script in your Amazon EC2 instance\.

   ```
   mkdir greengrass-dependency-checker-GGCv1.11.x
   cd greengrass-dependency-checker-GGCv1.11.x
   wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.11.x.zip
   unzip greengrass-dependency-checker-GGCv1.11.x.zip
   cd greengrass-dependency-checker-GGCv1.11.x
   sudo ./check_ggc_dependencies | more
   ```
**Important**  
<a name="lambda-runtime-prereqs"></a>This tutorial requires the Python 3\.7 runtime to run local Lambda functions\. When stream manager is enabled, it also requires the Java 8 runtime\. If the `check_ggc_dependencies` script produces warnings about these missing runtime prerequisites, make sure to install them before you continue\. You can ignore warnings about other missing optional runtime prerequisites\.

Your Amazon EC2 instance configuration is complete\. Continue to [Module 2: Installing the AWS IoT Greengrass Core software](module2.md)\.