# Setting Up an Amazon EC2 Instance<a name="setup-filter.ec2"></a>

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

Your Amazon EC2 instance configuration is complete\. Continue to [Module 2: Installing the AWS IoT Greengrass Core Software](module2.md)\.