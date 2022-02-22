--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

--------

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

**Important**  
As of February 11, 2022, you can no longer install or update the AWS IoT Greengrass Core software from an APT repository\. On devices where you added the AWS IoT Greengrass repository, you must [remove the repository from the sources list](#ggc-package-manager-remove-sources)\. Devices that run the software from the APT repository will continue to operate normally\. We recommend that you update the AWS IoT Greengrass Core software using [tar files](#download-and-extract-tarball)\.

The APT repository provided by AWS IoT Greengrass includes the following packages:
+ `aws-iot-greengrass-core`\. Installs the AWS IoT Greengrass Core software\.
+ `aws-iot-greengrass-keyring`\. Installs the GnuPG \(GPG\) keys used to sign the AWS IoT Greengrass package repository\.

  By downloading this software, you agree to the [ Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

**Topics**
+ [Use systemd scripts to manage the Greengrass daemon lifecycle](#ggc-package-manager-systemd)
+ [Uninstall the AWS IoT Greengrass core software using the APT repository](#ggc-package-manager-uninstall)
+ [Remove the AWS IoT Greengrass core software repository sources](#ggc-package-manager-remove-sources)

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

### Uninstall the AWS IoT Greengrass core software using the APT repository<a name="ggc-package-manager-uninstall"></a>

When you uninstall the AWS IoT Greengrass core software, you can choose whether to preserve or remove the AWS IoT Greengrass core software's configuration information, such as device certificates, group information, and log files\.

**To uninstall the AWS IoT Greengrass core software and preserve configuration information**
+ Run the following command to remove the AWS IoT Greengrass core software packages and preserve configuration information in the `/greengrass` folder\.

  ```
  sudo apt remove aws-iot-greengrass-core aws-iot-greengrass-keyring
  ```

**To uninstall the AWS IoT Greengrass core software and remove configuration information**

1. Run the following command to remove the AWS IoT Greengrass core software packages and remove configuration information from the `/greengrass folder`\.

   ```
   sudo apt purge aws-iot-greengrass-core aws-iot-greengrass-keyring
   ```

1. Remove the AWS IoT Greengrass core software repository from your sources list\. For more information, see [Remove the AWS IoT Greengrass core software repository sources](#ggc-package-manager-remove-sources)\.

### Remove the AWS IoT Greengrass core software repository sources<a name="ggc-package-manager-remove-sources"></a>

You can remove the AWS IoT Greengrass core software repository sources when you no longer need to install or update the AWS IoT Greengrass core software from the APT repository\. After February 11, 2022, you must remove the repository from your sources list to avoid an error when you run `apt update`\.

**To remove the APT repository from the sources list**
+ Run the following commands to remove the AWS IoT Greengrass core software repository from the sources list\.

  ```
  sudo rm /etc/apt/sources.list.d/greengrass.list
  sudo apt update
  ```

## Run AWS IoT Greengrass in a Docker container<a name="gg-docker-support"></a>

AWS IoT Greengrass provides a Dockerfile and Docker images that make it easier for you to run the AWS IoT Greengrass Core software in a Docker container\. For more information, see [AWS IoT Greengrass Docker software](what-is-gg.md#gg-docker-download)\.

**Note**  
You can also run a Docker application on a Greengrass core device\. To do so, use the [Greengrass Docker application deployment connector](docker-app-connector.md)\.

 

## Run AWS IoT Greengrass in a snap<a name="gg-snap-support"></a>

<a name="gg-snap-description"></a>AWS IoT Greengrass snap 1\.11\.x enables you to run a limited version of AWS IoT Greengrass through convenient software packages, along with all necessary dependencies, in a containerized environment\.

**Note**  <a name="gg-snap-v1.11-note"></a>
The AWS IoT Greengrass snap is available for AWS IoT Greengrass Core software v1\.11\.x\. AWS IoT Greengrass doesn’t provide a snap for v1\.10\.x\. Unsupported versions don't receive bug fixes or updates\.   
The AWS IoT Greengrass snap doesn't support connectors and machine learning \(ML\) inference\.

### Snap concepts<a name="gg-snap-concepts"></a>

The following are essential snap concepts to help you understand how to use the AWS IoT Greengrass snap:

**[Channel](https://snapcraft.io/docs/channels)**  
A snap component that defines which version of a snap is installed and tracked for updates\. Snaps are automatically updated to the latest version of the current channel\.

**[Interface](https://snapcraft.io/docs/interface-management)**  
A snap component that grants access to resources, such as networks and user files\.  
To run the AWS IoT Greengrass snap, the following interfaces must be connected\. Note that `greengrass-support-no-container` must be connected first and never disconnected\.  

```
      - greengrass-support-no-container
      - hardware-observe
      - home-for-hooks
      - hugepages-control
      - log-observe
      - mount-observe
      - network
      - network-bind
      - network-control
      - process-control
      - system-observe
```
The other interfaces are optional\. If your Lambda functions require access to specific resources, you might need to connect to the appropriate interfaces\.

**[Refresh](https://snapcraft.io/docs/keeping-snaps-up-to-date)**  
Snaps are automatically updated\. The `snapd` daemon is the snap package manager that checks for updates four times a day by default\. Each update check is called a refresh\. When a refresh occurs, the daemon stops, the snap gets updated, and then the daemon restarts\.

For more information, see the [Snapcraft](https://snapcraft.io/) website\.

### What's new with AWS IoT Greengrass snap v1\.11\.x<a name="gg-snap-whats-new"></a>

The following describes what's new and changed with the version 1\.11\.x of the AWS IoT Greengrass snap\.
+ This version supports only the `snap_daemon` user, exposed as user ID \(UID\) and group \(GID\) `584788`\.
+ This version supports only noncontainerized Lambda functions\.
**Important**  
Because noncontainerized Lambda functions must share the same user \(`snap_daemon`\), the Lambda functions have no isolation from each other\. For more information, see [Controlling execution of Greengrass Lambda functions by using group\-specific configuration](https://docs.aws.amazon.com/greengrass/latest/developerguide/lambda-group-config.html)\.
+ This version supports C, C\+\+, Java 8, Node\.js 12\.x, Python 2\.7, Python 3\.7, and Python 3\.8 runtimes\.
**Note**  
To avoid redundant Python runtimes, Python 3\.7 Lambda functions actually run the Python 3\.8 runtime\.

### Getting started with AWS IoT Greengrass snap<a name="gg-snap-get-started"></a>

The following procedure helps you install and configure the AWS IoT Greengrass snap on your device\.

#### Requirements<a name="gg-snap-requirements"></a>

To run the AWS IoT Greengrass snap, you must do the following:
+ Run the AWS IoT Greengrass snap on a supported Linux distribution, such as Ubuntu, Linux Mint, Debian, and Fedora\.
+ Install the `snapd` daemon on your device\. The `snapd` daemon including the `snap` tool manages the snap environment on your device\. 

For the list of supported Linux distributions and installation instructions, see [Installing snapd](https://snapcraft.io/docs/installing-snapd) in the *Snap documentation*\.

#### Install and configure the AWS IoT Greengrass snap<a name="gg-snap-install-config"></a>

The following tutorial shows you how to install and configure the AWS IoT Greengrass snap on your device\.

**Note**  
Although this tutorial uses an Amazon EC2 instance \(x86 t2\.micro Ubuntu 20\.04\), you can run the AWS IoT Greengrass snap with physical hardware, such as a Raspberry Pi\.
The `snapd` daemon is preinstalled on Ubuntu\.

1. Install the `core18` snap by running the following command in your device's terminal:

   ```
   sudo snap install core18
   ```

   The `core18` snap is a [base snap](https://snapcraft.io/docs/base-snaps) that provides a runtime environment with commonly used libraries\. This snap is built from [Ubuntu 18\.04 LTS](http://releases.ubuntu.com/18.04/)\.

1. Upgrade `snapd` by running the following command:

   ```
   sudo snap install --channel=edge snapd; sudo snap refresh --channel=edge snapd
   ```
**Note**  
`snapd` support for the AWS IoT Greengrass snap hasn't reached the stable channel\. We recommend that you use `--channel=edge` instead of `--channel=stable`\.

1. Run the `snap list` command to check if you have the AWS IoT Greengrass snap installed\.

   The following example response shows that `snapd` is installed, but `aws-iot-greengrass` isn't\.

   ```
   Name              Version               Rev    Tracking         Publisher   Notes
   amazon-ssm-agent  3.0.161.0             2996   latest/stable/…  aws✓        classic
   core              16-2.48               10444  latest/stable    canonical✓  core
   core18            20200929              1932   latest/stable    canonical✓  base
   lxd               4.0.4                 18150  4.0/stable/…     canonical✓  -
   snapd             2.48+git548.g929ccfb  10526  latest/edge      canonical✓  snapd
   ```

1. Choose one of the following options to install AWS IoT Greengrass snap 1\.11\.x\.
   + To install the AWS IoT Greengrass snap, run the following command:

     ```
     sudo snap install aws-iot-greengrass
     ```

     Example response:

     ```
     aws-iot-greengrass 1.11.5 from Amazon Web Services (aws) installed
     ```
   + To migrate from an earlier version to v1\.11\.x or update to the latest available patch version, run the following command:

     ```
     sudo snap refresh --channel=1.11.x aws-iot-greengrass
     ```

   Like other snaps, the AWS IoT Greengrass snap uses channels to manage minor versions\. Snaps are automatically updated to the latest available version of the current channel\. For examples, if you specify `--channel=1.11.x`, your AWS IoT Greengrass snap is updated to v1\.11\.5\. 

   You can run the `snap info aws-iot-greengrass` command to get the list of available channels for AWS IoT Greengrass\.

   Example response:

   ```
   name:      aws-iot-greengrass
   summary:   AWS supported software that extends cloud capabilities to local devices.
   publisher: Amazon Web Services (aws✓)
   store-url: https://snapcraft.io/aws-iot-greengrass
   contact:   https://repost.aws/tags/TA4ckIed1sR4enZBey29rKTg/aws-io-t-greengrass
   license:   Proprietary
   description: |
     AWS IoT Greengrass seamlessly extends AWS onto edge devices so they can act locally on the data
     they generate, while still using the cloud for management, analytics, and durable storage.
     AWS IoT Greenrgrass snap v1.11.0 enables you to run a limited version of AWS IoT Greengrass with
     all necessary dependencies in a containerized environment.
     The AWS IoT Greengrass snap doesn't support connectors and machine learning (ML) inference.
     By downloading this software you agree to the Greengrass Core Software License Agreement
     (https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf).
     For more information, see Run AWS IoT Greengrass in a snap
     (https://docs.aws.amazon.com/greengrass/latest/developerguide/install-ggc.html#gg-snap-support) in
     the AWS IoT Greengrass Developer.
     If you need help, try the AWS IoT Greengrass tag on AWS re:Post
     (https://repost.aws/tags/TA4ckIed1sR4enZBey29rKTg/aws-io-t-greengrass) or connect with an AWS IQ expert
     (https://iq.aws.amazon.com/services/aws/greengrass).
   snap-id: SRDuhPJGj4XPxFNNZQKOTvURAp0wxKnd
   channels:
     latest/stable:    1.11.3 2021-06-15 (59) 111MB -
     latest/candidate: 1.11.3 2021-06-14 (59) 111MB -
     latest/beta:      1.11.3 2021-06-14 (59) 111MB -
     latest/edge:      1.11.3 2021-06-14 (59) 111MB -
     1.11.x/stable:    1.11.3 2021-06-15 (59) 111MB -
     1.11.x/candidate: 1.11.3 2021-06-15 (59) 111MB -
     1.11.x/beta:      1.11.3 2021-06-15 (59) 111MB -
     1.11.x/edge:      1.11.3 2021-06-15 (59) 111MB -
   ```

1. To run the AWS IoT Greengrass snap, connect the following interfaces:
**Important**  
You must first connect `greengrass-support-no-container` and never disconnect it\.

   ```
   sudo snap connect aws-iot-greengrass:greengrass-support-no-container
   sudo snap connect aws-iot-greengrass:hardware-observe
   sudo snap connect aws-iot-greengrass:home-for-hooks
   sudo snap connect aws-iot-greengrass:hugepages-control
   sudo snap connect aws-iot-greengrass:log-observe
   sudo snap connect aws-iot-greengrass:mount-observe
   sudo snap connect aws-iot-greengrass:network
   sudo snap connect aws-iot-greengrass:network-bind
   sudo snap connect aws-iot-greengrass:network-control
   sudo snap connect aws-iot-greengrass:process-control
   sudo snap connect aws-iot-greengrass:system-observe
   ```

1. To access specific resources that your Lambda functions need, you can connect to additional interfaces\.

   Run the following command to get the list of AWS IoT Greengrass snap supported interfaces:

   ```
   snap connections aws-iot-greengrass
   ```

   Example response:

   ```
   Interface                Plug                                                Slot                 Notes
   camera                   aws-iot-greengrass:camera                           -                    -
   dvb                      aws-iot-greengrass:dvb                              -                    -
   gpio                     aws-iot-greengrass:gpio                             -                    -
   gpio-memory-control      aws-iot-greengrass:gpio-memory-control              -                    -
   greengrass-support       aws-iot-greengrass:greengrass-support-no-container  :greengrass-support  -
   hardware-observe         aws-iot-greengrass:hardware-observe                 :hardware-observe    manual
   hardware-random-control  aws-iot-greengrass:hardware-random-control          -                    -
   home                     aws-iot-greengrass:home-for-greengrassd             -                    -
   home                     aws-iot-greengrass:home-for-hooks                   :home                manual
   hugepages-control        aws-iot-greengrass:hugepages-control                :hugepages-control   manual
   i2c                      aws-iot-greengrass:i2c                              -                    -
   iio                      aws-iot-greengrass:iio                              -                    -
   joystick                 aws-iot-greengrass:joystick                         -                    -
   log-observe              aws-iot-greengrass:log-observe                      :log-observe         manual
   mount-observe            aws-iot-greengrass:mount-observe                    :mount-observe       manual
   network                  aws-iot-greengrass:network                          :network             -
   network-bind             aws-iot-greengrass:network-bind                     :network-bind        -
   network-control          aws-iot-greengrass:network-control                  :network-control     -
   opengl                   aws-iot-greengrass:opengl                           :opengl              -
   optical-drive            aws-iot-greengrass:optical-drive                    :optical-drive       -
   process-control          aws-iot-greengrass:process-control                  :process-control     -
   raw-usb                  aws-iot-greengrass:raw-usb                          -                    -
   removable-media          aws-iot-greengrass:removable-media                  -                    -
   serial-port              aws-iot-greengrass:serial-port                      -                    -
   spi                      aws-iot-greengrass:spi                              -                    -
   system-observe           aws-iot-greengrass:system-observe                   :system-observe      -
   ```

   If you see a hyphen \(\-\) in the Slot column, the corresponding interface isn't connected\.

1. Follow [Configure AWS IoT Greengrass on AWS IoT](https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-config.html) to create a Greengrass group and download the security resources and configuration file; for example, `c6973960cc-setup.tar.gz`\.

   This compressed file contains the core device certificate and cryptographic keys that enable secure communications between AWS IoT Core and the `config.json` file that contains configuration information specific to your Greengrass core\. This information includes the location of certificate files and the AWS IoT Core endpoint\.
**Note**  
If you downloaded the file to a different device, follow the first two steps in [Start AWS IoT Greengrass on the core device](https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-device-start.html) to transfer the files to the AWS IoT Greengrass core device\.

1. For the AWS IoT Greengrass snap, make sure that you update the [config\.json](https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html#config-json) file, as shown in the following:
   + Replace *hash* with the 10\-digit hash number that is used for your security resources and configuration file names\.
   + Run the `tar -xzf hash-setup.tar.gz -C my-certs/` command to decompress the `hash-setup.tar.gz` file\.

   ```
   {
     ...
     "crypto" : {
       "principals" : {
         "SecretsManager" : {
           "privateKeyPath" : "file:///snap/aws-iot-greengrass/current/greengrass/certs/hash.private.key"
         },
         "IoTCertificate" : {
           "privateKeyPath" : "file:///snap/aws-iot-greengrass/current/greengrass/certs/hash.private.key",
           "certificatePath" : "file:///snap/aws-iot-greengrass/current/greengrass/certs/hash.cert.pem"
         }
       },
       "caPath" : "file:///snap/aws-iot-greengrass/current/greengrass/certs/root.ca.pem"
     },
     "writeDirectory": "/var/snap/aws-iot-greengrass/current/ggc-write-directory",
     "pidFileDirectory": "/var/snap/aws-iot-greengrass/current/pidFileDirectory"
   }
   ```

1. Run the following command to add your AWS IoT Greengrass certificate and configuration files:

   ```
   sudo snap set aws-iot-greengrass gg-certs=/home/ubuntu/my-certs
   ```

### Deploying a Lambda function<a name="gg-snap-lambda"></a>

This section shows you how to deploy a customer managed Lambda function on the AWS IoT Greengrass snap\.

**Important**  
AWS IoT Greengrass snap v1\.11 only supports noncontainerized Lambda functions\.

1. Run the following command to start the AWS IoT Greengrass daemon:

   ```
   sudo snap start aws-iot-greengrass
   ```

   Example response:

   ```
   Started.
   ```
**Note**  
If you get an error, you can use the `snap run` command for a detailed error message\. For more troubleshooting information, see [error: cannot perform the following tasks: \- Run service command "start" for services \["greengrassd"\] of snap "aws\-iot\-greengrass" \(\[start snap\.aws\-iot\-greengrass\.greengrassd\.service\] failed with exit status 1: Job for snap\.aws\-iot\-greengrass\.greengrassd\.service failed because the control process exited with error code\. See "systemctl status snap\.aws\-iot\-greengrass\.greengrassd\.service" and "journalctl \-xe" for details\.\)](#gg-snap-troubleshoot-snaprun)\.

1. Run the following command to confirm that the daemon is running: 

   ```
   snap services aws-iot-greengrass.greengrassd
   ```

   Example response:

   ```
   Service                         Startup   Current  Notes
   aws-iot-greengrass.greengrassd  disabled  active   -
   ```

1. Follow [Module 3 \(part 1\): Lambda functions on AWS IoT Greengrass](https://docs.aws.amazon.com/greengrass/latest/developerguide/module3-I.html) to create and deploy a Hello World Lambda function\. However, before you deploy the Lambda function, complete the next step\.

1. Make sure that your Lambda function run as the `snap_daemon` user and in the no container mode\. To update the settings of your Greengrass group, do the following in the AWS IoT Greengrass console:

   1. Sign in to the AWS IoT Greengrass console\.

   1. <a name="console-gg-groups"></a>In the AWS IoT console, in the navigation pane, choose **Greengrass**, **Classic \(V1\)**, **Groups**\.

   1. Under **Greengrass groups**, choose the target group\.

   1. On the group configuration page, in the navigation pane, choose **Settings**\.

   1. Under **Lambda runtime environment**, do the following:

      1. For **Default Lambda function user ID/ group ID**, choose **Another user ID/group ID**, and then enter **584788** for both **UID \(number\)** and **GID \(number\)**\.

      1. For **Default Lambda function containerization**, choose **No container**\.

### Stopping the AWS IoT Greengrass daemon<a name="gg-snap-stop"></a>

You can use the `snap stop` command to stop a service\.

To stop the AWS IoT Greengrass daemon, run the following command:

```
sudo snap stop aws-iot-greengrass
```

The command should return `Stopped.`\.

To check if you successfully stopped the snap, run the following command:

```
snap services aws-iot-greengrass.greengrassd
```

Example response:

```
Service                         Startup   Current   Notes
aws-iot-greengrass.greengrassd  disabled  inactive  -
```

### Uninstalling the AWS IoT Greengrass snap<a name="gg-snap-uninstall"></a>

To uninstall the AWS IoT Greengrass snap, run the following command:

```
sudo snap remove aws-iot-greengrass
```

Example response:

```
aws-iot-greengrass removed
```

### Troubleshooting the AWS IoT Greengrass snap<a name="gg-snap-troubleshoot"></a>

Use the following information to help troubleshoot issues with the AWS IoT Greengrass snap\.

#### Got permission denied errors\.<a name="gg-snap-troubleshoot-permission-denied"></a>

**Solution**: Permission denied errors are often because of missing interfaces\. For the list of missing interfaces and detailed troubleshooting information, you can use the `snappy-debug` tool\.

Run the following command to install the tool\.

```
sudo snap install snappy-debug
```

Example response:

```
snappy-debug 0.36-snapd2.45.1 from Canonical✓ installed
```

Run the `sudo snappy-debug` command in a separate terminal session\. The operation continues until a permission denied error occurs\.

For example, if your Lambda function tries to read a file in the `$HOME` directory, you may get the following response:

```
INFO: Following '/var/log/syslog'. If have dropped messages, use:
INFO: $ sudo journalctl --output=short --follow --all | sudo snappy-debug
kernel.printk_ratelimit = 0
= AppArmor =
Time: Dec  6 04:48:26
Log: apparmor="DENIED" operation="mknod" profile="snap.aws-iot-greengrass.greengrassd" name="/home/ubuntu/my-file.txt" pid=12345 comm="touch" requested_mask="c" denied_mask="c" fsuid=0 ouid=0
File: /home/ubuntu/my-file.txt (write)
Suggestion:
* add 'home' to 'plugs'
```

This example shows that creating the `/home/ubuntu/my-file.txt` file caused the permission error\. It also suggests that you add `home` to `plugs`\. However, this sugguestion is not applicable\. The `home-for-greengrassd` and `home-for-hooks` plugs are only given the read\-only access\.

For more information, see [The snappy\-debug snap](https://snapcraft.io/docs/debug-snaps#heading--snappy-debug) in the *Snap documentation*\.

#### error: cannot perform the following tasks: \- Run service command "start" for services \["greengrassd"\] of snap "aws\-iot\-greengrass" \(\[start snap\.aws\-iot\-greengrass\.greengrassd\.service\] failed with exit status 1: Job for snap\.aws\-iot\-greengrass\.greengrassd\.service failed because the control process exited with error code\. See "systemctl status snap\.aws\-iot\-greengrass\.greengrassd\.service" and "journalctl \-xe" for details\.\)<a name="gg-snap-troubleshoot-snaprun"></a>

**Solution**: You might see this error when the `snap start aws-iot-greengrass` command fails to start the AWS IoT Greengrass Core software\.

For more troubleshooting information, run the following command:

```
sudo snap run aws-iot-greengrass.greengrassd
```

Example response:

```
Couldn't find /snap/aws-iot-greengrass/44/greengrass/config/config.json.
```

This examples shows that AWS IoT Greengrass couldn't find the `config.json` file\. You might check the configuration and certificate files\.

#### /var/snap/aws\-iot\-greengrass/current/ggc\-write\-directory/packages/1\.11\.5/rootfs/merged is not an absolute path or is a symlink\.<a name="gg-snap-troubleshoot-lambda"></a>

**Solution**: The AWS IoT Greengrass snap supports only noncontainerized Lambda functions\. Make sure that you run your Lambda functions in the no container mode\. For more information, see [Considerations when choosing Lambda function containerization](https://docs.aws.amazon.com/greengrass/latest/developerguide/lambda-group-config.html#no-container-mode) in the *AWS IoT Greengrass Version 1 Developer Guide*\.

#### The snapd daemon failed to restart after you ran the sudo snap refresh snapd command\.<a name="gg-snap-troubleshoot-snapd"></a>

**Solution**: Follow steps 6 through 8 in [Install and configure the AWS IoT Greengrass snap](#gg-snap-install-config) to add the AWS IoT Greengrass certificate and configuration files to the AWS IoT Greengrass snap\.

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