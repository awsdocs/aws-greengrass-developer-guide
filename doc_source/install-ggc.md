# Install the AWS IoT Greengrass Core software<a name="install-ggc"></a>

<a name="ggc-software-descripton"></a> The AWS IoT Greengrass Core software extends AWS functionality onto an AWS IoT Greengrass core device, making it possible for local devices to act locally on the data they generate\.

AWS IoT Greengrass provides several options for installing the AWS IoT Greengrass Core software:
+ [Download and extract a tar\.gz file](#download-and-extract-tarball)\.
+ [Run the Greengrass Device Setup script](#run-device-setup-script)\.
+ [Install from an APT repository](#ggc-package-manager)\.

AWS IoT Greengrass also provides containerized environments that run the AWS IoT Greengrass Core software\.
+ [Run AWS IoT Greengrass in a Docker container](#gg-docker-support)\.
+ [Run AWS IoT Greengrass in a snap](#gg-snap-support)\.

 

## Download and extract the AWS IoT Greengrass Core software package<a name="download-and-extract-tarball"></a>

Choose the AWS IoT Greengrass Core software for your platform to download as a tar\.gz file and extract on your device\. You can download recent versions of the software\. For more information, see [AWS IoT Greengrass Core software](what-is-gg.md#gg-core-download-tab)\.

 

## Run the Greengrass device setup script<a name="run-device-setup-script"></a>

Run Greengrass device setup to configure your device, install the latest AWS IoT Greengrass Core software version, and deploy a Hello World Lambda function in minutes\. For more information, see [Quick start: Greengrass device setup](quick-start.md)\.

 

## Install the AWS IoT Greengrass Core software from an APT repository<a name="ggc-package-manager"></a>

You can use the Advanced Package Tool \(APT\) package management system to install the latest version of the AWS IoT Greengrass Core software on your core device\. The APT repository provided by AWS IoT Greengrass includes the following packages:
+ `aws-iot-greengrass-core`\. Installs the AWS IoT Greengrass Core software\.
+ `aws-iot-greengrass-keyring`\. Installs the GnuPG \(GPG\) keys used to sign the AWS IoT Greengrass package repository\.

  By downloading this software, you agree to the [ Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

You should be aware of the following considerations when you use the `apt` command to install the AWS IoT Greengrass Core software:

The AWS IoT Greengrass Core software is installed in the root directory\.  
The `apt` command installs the AWS IoT Greengrass Core software in a `greengrass` directory in the root file system\. If `/greengrass` is already present, the command installs the new software version, but does not overwrite any group information or core configuration\.

Over\-the\-air \(OTA\) updates are not supported\.  <a name="ota-updates-not-supported"></a>
You can use the `apt` installation option to upgrade the AWS IoT Greengrass Core software on your core device, but it doesn't support the safe update path provided by the AWS IoT Greengrass OTA update agent\. The OTA update agent is a software component included with the AWS IoT Greengrass Core software package that's installed when you use the [Download and extract a tar\.gz file](#download-and-extract-tarball) or [Run the Greengrass device setup script](#run-device-setup-script) installation options\. The OTA update agent helps to guarantee that the core continues to function after an update by rolling back if the updates fails\. For more information, see [OTA updates of AWS IoT Greengrass Core software](core-ota-update.md)\.

We recommend that you keep the keyring package updated\.  <a name="install-keyring-package"></a>
Keeping the `aws-iot-greengrass-keyring` package updated allows you to receive updates for the GPG keys used to authenticate AWS IoT Greengrass APT repositories\. It also allows you to upgrade the AWS IoT Greengrass Core software more easily\. These trusted keys are installed in `/etc/apt/trusted.gpg.d/`\. Public keys are valid for two years\. If they expire, you must reconfigure the keyring package:  

```
wget -O aws-iot-greengrass-keyring.deb https://d1onfpft10uf5o.cloudfront.net/greengrass-apt/downloads/aws-iot-greengrass-keyring.deb
sudo dpkg -i aws-iot-greengrass-keyring.deb
```
In the unlikely event that the keys managed by AWS IoT Greengrass become compromised, you must update the `aws-iot-greengrass-keyring` package to replace the compromised keys with new keys\. For more information, contact [AWS Customer Support](https://aws.amazon.com/contact-us/)\.

### Requirements<a name="ggc-package-manager-reqs"></a>

The following requirements apply for using `apt` to install the AWS IoT Greengrass Core software:
+ Your device must be running one of the following platforms:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/install-ggc.html)
+ You must have root access on the device\.
+ To complete the steps in the following procedures, the following shell commands must be installed on the device: `sudo`, `wget` or `curl`, `dpkg`, `echo`, `unzip`, and `tar`\.

### Using apt to install the AWS IoT Greengrass Core software<a name="ggc-package-manager-install"></a>

You can use the APT package management system to install the AWS IoT Greengrass Core software on your device\. Some core configuration steps might be required before you install the software\.

In the following procedures, run the commands in a terminal window on your device\.

**To configure your core**

1. If you're setting up AWS IoT Greengrass for the first time, you must configure your core\. If the `adduser` or `addgroup` command is not available, use `useradd` or `groupadd` instead\.

   1. Create the `ggc_user` and `ggc_group` system accounts\.

      ```
      sudo adduser --system ggc_user
      sudo addgroup --system ggc_group
      ```

   1. Set up your core device certificates and keys and your core configuration file\.

      1. Follow the steps in [Configure AWS IoT Greengrass on AWS IoT](gg-config.md) to create a Greengrass group and register your core\. This process also generates a security resources package that you download\. The package is a tar\.gz file that contains a core device certificate, public\-private keys, and the core configuration file\. The name of the file starts with a 10\-digit hash \(for example, `c6973960cc-setup.tar.gz`\) that's also used for the certificate and key file names\.

         Skip step 11 where you download the AWS IoT Greengrass Core software\.

      1. Transfer the package to your core device and run the following command to install the security resources\. Replace *hash* with the 10\-digit hash from your tar\.gz file\.

         ```
         sudo mkdir -p /greengrass
         sudo tar -xzvf hash-setup.tar.gz -C /greengrass
         ```

      1. Download a root CA certificate\. For information about choosing the appropriate root CA certificate, see [Server authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) in the *AWS IoT Core Developer Guide*\.

         The following example downloads `AmazonRootCA1.pem`\. To use `curl`, replace `wget -O` in the command with `curl`\.

         ```
         cd /greengrass/certs/
         sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
         ```

      These steps install the certificates and keys to `/greengrass/certs` and the configuration file to `/greengrass/config`\. For more information, see [AWS IoT Greengrass core configuration file](gg-core.md#config-json)\.

1. Download and run the dependency checker for AWS IoT Greengrass\. The dependency checker makes sure you have all of the required dependencies for the AWS IoT Greengrass Core software\. You can skip this step if you're upgrading from one patch version to another patch version \(for example, v1\.9\.3 to v1\.9\.4\)\.

   1. In the directory where you want to download the script, run the following command\. To use `curl`, replace `wget` in the command with `curl`\.

      ```
      mkdir greengrass-dependency-checker-GGCv1.11.x
      cd greengrass-dependency-checker-GGCv1.11.x
      wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.11.x.zip
      unzip greengrass-dependency-checker-GGCv1.11.x.zip
      cd greengrass-dependency-checker-GGCv1.11.x
      sudo ./check_ggc_dependencies | more
      ```

   1. Where `more` appears, press the Spacebar key to page throught the output\.
      + If [stream manager](stream-manager.md) is enabled in your Greengrass group, you must also install the Java 8 runtime before you deploy the group\. This feature is enabled by default when you use the **Default Group creation** workflow in the AWS IoT Greengrass console to create a group\.
      + You can install your target Lambda runtimes \(for example, Python 3\.8\) and ignore the warnings about other missing optional runtime prerequisites\.
      + You can ignore warnings about missing shell commands\. These are required by the OTA update agent, which isn't included in this installation\.

       

**To install the AWS IoT Greengrass Core software**

1. Install the AWS IoT Greengrass keyring package and add the repository\. To use `curl`, replace `wget -O` in the command with `curl`\.

   ```
   wget -O aws-iot-greengrass-keyring.deb https://d1onfpft10uf5o.cloudfront.net/greengrass-apt/downloads/aws-iot-greengrass-keyring.deb
   sudo dpkg -i aws-iot-greengrass-keyring.deb
   echo "deb https://dnw9lb6lzp2d8.cloudfront.net stable main" | sudo tee /etc/apt/sources.list.d/greengrass.list
   ```
**Note**  
If you keep the keyring package updated on your device, this step is required only the first time you install the AWS IoT Greengrass Core software from the APT repository\.

1. Update your list of packages\.

   ```
   sudo apt update
   ```

1. Install the AWS IoT Greengrass Core software\.

   ```
   sudo apt install aws-iot-greengrass-core
   ```

1. Start the Greengrass daemon\. The following commands use [systemd scripts](#ggc-package-manager-systemd) installed with the `aws-iot-greengrass-core` package\.

   ```
   systemctl start greengrass.service
   systemctl status greengrass.service
   ```

   If the output displays an `Active` state of `active (running)`, the daemon started successfully\.

 

**To stop using the APT repository**

If you want to stop using the APT repository for AWS IoT Greengrass, remove the packages and update your sources list\.

**Note**  
The `remove` command removes the packages, but not your configuration information\. If you also want to permanently remove all configuration information \(including device certificates, group information, and log files\), replace `remove` in the following command with `purge`\.

```
sudo apt remove aws-iot-greengrass-core aws-iot-greengrass-keyring
sudo rm /etc/apt/sources.list.d/greengrass.list
sudo apt update
```

### Use systemd scripts to manage the Greengrass daemon lifecycle<a name="ggc-package-manager-systemd"></a>

The `aws-iot-greengrass-core` package also installs `systemd` scripts that you can use to manage the AWS IoT Greengrass Core software \(daemon\) lifecycle\.
+ To start the Greengrass daemon during boot:

  ```
  systemctl enable greengrass.service
  ```
+ To start the Greengrass daemon:

  ```
  systemctl start greengrass.service
  ```
+ To stop the Greengrass daemon:

  ```
  systemctl stop greengrass.service
  ```
+ To check the status of the Greengrass daemon:

  ```
  systemctl status greengrass.service
  ```

 

## Run AWS IoT Greengrass in a Docker container<a name="gg-docker-support"></a>

AWS IoT Greengrass provides a Dockerfile and Docker images that make it easier for you to run the AWS IoT Greengrass Core software in a Docker container\. For more information, see [AWS IoT Greengrass Docker software](what-is-gg.md#gg-docker-download)\.

**Note**  
You can also run a Docker application on a Greengrass core device\. To do so, use the [Greengrass Docker application deployment connector](docker-app-connector.md)\.

 

## Run AWS IoT Greengrass in a snap<a name="gg-snap-support"></a>

<a name="snap-ggc-v1.8-only"></a>Currently, AWS IoT Greengrass snap software is available for AWS IoT Greengrass core version 1\.8 only\.

<a name="snap-ggc-desc"></a> The AWS IoT Greengrass snap software download makes it possible for you to run a version of AWS IoT Greengrass with limited functionality on Linux cloud, desktop, and IoT environments through convenient containerized software packages\. These packages, or snaps, contain the AWS IoT Greengrass Core software and its dependencies\. You can download and use these packages on your Linux environments as\-is\.

<a name="snap-ggc-limitations"></a> The AWS IoT Greengrass snap allows you to run a version of AWS IoT Greengrass with limited functionality on your Linux environments\. Currently, Java, Node\.js, and native Lambda functions are not supported\. Machine learning inference, connectors, and noncontainerized Lambda functions are also not supported\.

### Getting started with AWS IoT Greengrass snap<a name="gg-snap-setup"></a>

 Because the prepackaged AWS IoT Greengrass snap is designed to use system defaults, you might need to perform these other steps: 
+  The AWS IoT Greengrass snap is configured to use default Greengrass user and group configurations\. This allows it to work easily with Greengrass groups or Lambda functions that run as root\. If you need to use Greengrass groups or Lambda functions that do not run as root, update these configurations and add them to your system\. 
+  The AWS IoT Greengrass snap uses many interfaces that must be connected before the snap can operate normally\. These interfaces are connected automatically during setup\. If you use other options while you set up your snap, you might need to connect these interfaces manually\. 

 For more information about the AWS IoT Greengrass snap and these modifications, see [Greengrass Snap Release Notes](https://assets.ubuntu.com/v1/15d52dd3-greengrass-snap-release-notes.pdf)\. 

1.  Install and upgrade snapd by running the following command in your device's terminal: 

   ```
   sudo apt-get update && sudo apt-get upgrade snapd
   ```

1.  If you need to use Greengrass groups or Lambda functions that do not run as root, update your default Greengrass user and group configurations, and add them to your system\. For more information about updating user and group configurations with AWS IoT Greengrass, see [Setting the default access identity for Lambda functions in a group](lambda-group-config.md#lambda-access-identity-groupsettings)\. 
   + For the Ubuntu Core system:
     +  To add the `ggc_user` user, use: 

       ```
       sudo adduser --extrausers --system ggc_user
       ```
     +  To add the `ggc_group` group, use: 

       ```
       sudo addgroup --extrausers --system ggc_group 
       ```
   + For the Ubuntu classic system:
     +  To add the `ggc_user` user to an Ubuntu classic system, omit the `--extrausers` flag and use: 

       ```
       sudo adduser --system ggc_user
       ```
     +  To add the `ggc_group` to an Ubuntu classic system, omit the `--extrausers` flag and use: 

       ```
       sudo addgroup --system ggc_group
       ```

1.  In your terminal, run the following command to install the Greengrass snap: 

   ```
   sudo snap install aws-iot-greengrass
   ```
**Note**  
 You can also use the AWS IoT Greengrass snap download link to install the Greengrass snap locally\. If you are installing locally from this file and do not have the associated assertions, use the `--dangerous` flag:   

   ```
   sudo snap install --dangerous aws-iot-greengrass*.snap
   ```
 The `--dangerous` flag interferes with the AWS IoT Greengrass snap's ability to connect its required interfaces\. If you use this flag, you must manually connect the required interfaces using the snap connect command\.   For more information, see [Greengrass Snap Release Notes](https://assets.ubuntu.com/v1/15d52dd3-greengrass-snap-release-notes.pdf)\. 

1.  After the snap is installed, run the following command to add your Greengrass certificate and configuration files: 

   ```
   sudo snap set aws-iot-greengrass gg-certs=/path-to-the-certs/22e592db.tgz
   ```
**Note**  
 If necessary, you can troubleshoot issues by viewing the AWS IoT Greengrass core logs, particularly `runtime.log`\. You can print the contents of `runtime.log` to your terminal by running the following command:   

   ```
   sudo cat /var/snap/aws-iot-greengrass/current/ggc-writable/var/log/system/runtime.log
   ```

1.  Run the following command to validate that your setup is functioning correctly: 

   ```
   $ snap services aws-iot-greengrass
   ```

   You should see the following response:

   ```
   Service                         Startup  Current  Notes
   aws-iot-greengrass.greengrassd  enabled  active   -
   ```

 Your Greengrass setup is now complete\. You can now use the AWS IoT Greengrass console, AWS REST API, or AWS CLI to deploy the Greengrass groups associated with this snap\. For information about using the console to deploy a Greengrass group, see the [Deploy cloud configurations to an AWS IoT Greengrass core device](https://docs.aws.amazon.com/greengrass/latest/developerguide/configs-core.html)\. For information about using the CLI or REST API to deploy a Greengrass group, see [CreateDeployment](https://docs.aws.amazon.com/greengrass/latest/apireference/createdeployment-post.html) in the *AWS IoT Greengrass API Reference*\. 

 For more information about configuring local resource access with snap AppArmor confinement, using the snapd REST API, and configuring snap interfaces, see [Greengrass Snap Release Notes](https://assets.ubuntu.com/v1/15d52dd3-greengrass-snap-release-notes.pdf)\. 

## Archive an AWS IoT Greengrass Core software installation<a name="archive-ggc-version"></a>

When you upgrade to a new version of the AWS IoT Greengrass Core software, you can archive the currently installed version\. This preserves your current installation environment so you can test a new software version on the same hardware\. This also makes it easy to roll back to your archived version for any reason\.

**To archive the current installation and install a new version**

1. Download the [AWS IoT Greengrass Core software](what-is-gg.md#gg-core-download-tab) installation package that you want to upgrade to\.

1. Copy the package to the destination core device\. For instructions that show how to transfer files, see this [step](gg-device-start.md#transfer-files-to-device)\.
**Note**  
You copy your current certificates, keys, and configuration file to the new installation later\.

   Run the commands in the following steps in your core device terminal\.

1. Make sure that the Greengrass daemon is stopped on the core device\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/ggc-version/bin/daemon`, then the daemon is running\.
**Note**  
This procedure is written with the assumption that the AWS IoT Greengrass Core software is installed in the `/greengrass` directory\.

   1. To stop the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd stop
      ```

1. Move the current Greengrass root directory to a different directory\.

   ```
   sudo mv /greengrass /greengrass_backup
   ```

1. Untar the new software on the core device\. Replace the *os\-architecture* and *version* placeholders in the command\.

   ```
   sudo tar –zxvf greengrass-os-architecture-version.tar.gz –C /
   ```

1. Copy the archived certificates, keys, and configuration file to the new installation\.

   ```
   sudo cp /greengrass_backup/certs/* /greengrass/certs
   sudo cp /greengrass_backup/config/* /greengrass/config
   ```

1. Start the daemon:

   ```
   cd /greengrass/ggc/core/
   sudo ./greengrassd start
   ```

Now, you can make a group deployment to test the new installation\. If something fails, you can restore the archived installation\.

**To restore the archived installation**

1. Stop the daemon\.

1. Delete the new `/greengrass` directory\.

1. Move the `/greengrass_backup` directory back to `/greengrass`\.

1. Start the daemon\.