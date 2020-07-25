# Configuring your device<a name="device-config-setup"></a>

To configure your device you must install AWS IoT Greengrass dependencies, configure the AWS IoT Greengrass Core software, configure your host computer to access your device, and configure user permissions on your device\.

## Verify AWS IoT Greengrass dependencies on the device under test<a name="install-gg-dependencies"></a>

Before IDT for AWS IoT Greengrass can test your devices, make sure that you have set up your device as described in [Getting started with AWS IoT Greengrass](https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-gs.html)\. For information about supported platforms, see [Supported platforms](https://docs.aws.amazon.com/greengrass/latest/developerguide/what-is-gg.html#gg-platforms)\.

## Configure the AWS IoT Greengrass software<a name="config-gg"></a>

IDT for AWS IoT Greengrass tests your device for compatibility with a specific version of AWS IoT Greengrass\. IDT provides two options for testing AWS IoT Greengrass on your devices:
+ Download and use a version of the [AWS IoT Greengrass Core software](what-is-gg.md#gg-core-download-tab)\. IDT installs the software for you\.
+ Use a version of the AWS IoT Greengrass Core software already installed on your device\.

**Note**  
Each version of AWS IoT Greengrass has a corresponding IDT version\. You must download the version of IDT that corresponds to the version of AWS IoT Greengrass you are using\.

There are two options for installing AWS IoT Greengrass on your device:
+ Download the AWS IoT Greengrass Core software and configure IDT for AWS IoT Greengrass to use it\.
+ Use an existing installation of the AWS IoT Greengrass Core software\.

The following sections describe these options\. You only need to do one\.

### Option 1: Download the AWS IoT Greengrass Core software and configure AWS IoT Device Tester to use it<a name="download-gg"></a>

You can download the AWS IoT Greengrass Core software from the [AWS IoT Greengrass Core Software](what-is-gg.md#gg-core-download-tab) downloads page\. 

1. Find the correct architecture and Linux distribution, and then choose **Download**\.

1. Copy the tar\.gz file to the `<device-tester-extract-location>/products/greengrass/ggc`\.

**Note**  
Do not change the name of the AWS IoT Greengrass tar\.gz file\. Do not place multiple files in this directory for the same operating system and architecture\. For example having both `greengrass-linux-armv7l-1.7.1.tar.gz` and `greengrass-linux-armv7l-1.8.1.tar.gz` files in that directory will cause the tests to fail\.

### Option 2: Use an existing installation of AWS IoT Greengrass with AWS IoT Device Tester<a name="existing-gg"></a>

Configure IDT to test the AWS IoT Greengrass Core software installed on your device by adding the `greengrassLocation` attribute to the `device.json` file in the `<device_tester_extract_location>/configs` folder\. For example:

```
"greengrassLocation" : "<path-to-greengrass-on-device>"
```

For more information about the `device.json` file, see [Configure device\.json](set-config.md#device-config)\.

On Linux devices, the default location of the AWS IoT Greengrass Core software is `/greengrass`\.

**Note**  
Your device should have an installation of the AWS IoT Greengrass Core software that has not been started\.  
Make sure you have added the `ggc_user` user and `ggc_group` on your device\. For more information, see [Environment setup for AWS IoT Greengrass](https://docs.aws.amazon.com/greengrass/latest/developerguide/module1.html)\.

## Configure your host computer to access your device under test<a name="configure-host"></a>

IDT runs on your host computer and must be able to use SSH to connect to your device\. There are two options to allow IDT to gain SSH access to your devices under test:

1. Follow the instructions here to create an SSH key pair and authorize your key to sign in to your device under test without specifying a password\.

1. Provide a user name and password for each device in the `device.json` file\. For more information, see [Configure device\.json](set-config.md#device-config)\.

You can use any SSL implementation to create an SSH key\. The following instructions show you how to use [SSH\-KEYGEN](https://www.ssh.com/ssh/keygen/) or [ PuTTYgen](https://www.ssh.com/ssh/putty/windows/puttygen) \(for Windows\)\. If you are using another SSL implementation, refer to the documentation for that implementation\.

IDT uses SSH keys to authenticate with your device under test\. 

**To create an SSH key with SSH\-KEYGEN**

1. Create an SSH key\.

   You can use the Open SSH ssh\-keygen command to create an SSH key pair\. If you already have an SSH key pair on your host computer, it is a best practice to create a SSH key pair specifically for IDT\. This way, after you have completed testing, your host computer can no longer connect to your device without entering a password\. It also allows you to restrict access to the remote device to only those who need it\.
**Note**  
Windows does not have an installed SSH client\. For information about installing an SSH client on Windows, see [Download SSH Client Software](https://www.ssh.com/ssh/#sec-Download-client-software)\.

   The ssh\-keygen command prompts you for a name and path to store the key pair\. By default, the key pair files are named `id_rsa` \(private key\) and `id_rsa.pub` \(public key\)\. On macOS and Linux, the default location of these files is `~/.ssh/`\. On Windows, the default location is `C:\Users\<user-name>\.ssh`\.

   When prompted, enter a key phrase to protect your SSH key\. For more information, see [Generate a New SSH key](https://www.ssh.com/ssh/keygen/)\.

1. Add authorized SSH keys to your device under test\.

   IDT must use your SSH private key to sign in to your device under test\. To authorize your SSH private key to sign in to your device under test, use the ssh\-copy\-id command from your host computer\. This command adds your public key into the `~/.ssh/authorized_keys` file on your device under test\. For example:

   $ ssh\-copy\-id *<remote\-ssh\-user>*@*<remote\-device\-ip>*

   Where *remote\-ssh\-user* is the user name used to sign in to your device under test and *remote\-device\-ip* is the IP address of the device under test to run tests against\. For example:

   ssh\-copy\-id pi@192\.168\.1\.5

   When prompted, enter the password for the user name you specified in the ssh\-copy\-id command\.

   ssh\-copy\-id assumes the public key is named `id_rsa.pub` and is stored the default location \(on macOS and Linux, `~/.ssh/` and on Windows, `C:\Users\<user-name>\.ssh`\)\. If you gave the public key a different name or stored it in a different location, you must specify the fully qualified path to your SSH public key using the \-i option to ssh\-copy\-id \(for example, ssh\-copy\-id \-i \~/my/path/myKey\.pub\)\. For more information about creating SSH keys and copying public keys, see [SSH\-COPY\-ID](https://www.ssh.com/ssh/copy-id)\.

**To create an SSH key using PuTTYgen \(Windows only\)**

1. Make sure you have the OpenSSH server and client installed on your device under test\. For more information, see [OpenSSH](https://www.openssh.com/)\.

1. Install [PuTTYgen](https://www.puttygen.com/) on your device under test\.

1. Open PuTTYgen\.

1. Choose **Generate** and move your mouse cursor inside the box to generate a private key\.

1. From the **Conversions** menu, choose **Export OpenSSH key**, and save the private key with a `.pem` file extension\.

1. Add the public key to the `/home/<user>/.ssh/authorized_keys` file on device under test\.

   1. Copy the public key text from the PuTTYgen window\.

   1. Use PuTTY to create a session on your device under test\.

      1. From a command prompt or Windows Powershell window, run the following command:

         C:/*<path\-to\-putty>*/putty\.exe \-ssh *<user>*@*<dut\-ip\-address>*

      1. When prompted, enter your device's password\.

      1. Use vi or another text editor to append the public key to the `/home/<user>/.ssh/authorized_keys` file on your device under test\.

1. Update your `device.json` file with your user name, the IP address, and path to the private key file that you just saved on your host computer for each device under test\. For more information, see [Configure device\.json](set-config.md#device-config)\. Make sure you provide the full path and file name to the private key and use forward slashes \('/'\)\. For example, for the Windows path `C:\DT\privatekey.pem`, use `C:/DT/privatekey.pem` in the `device.json` file\. 

## Configure user permissions on your device<a name="root-access"></a>

IDT performs operations on various directories and files in a device under test\. Some of these operations require elevated permissions \(using sudo\)\. To automate these operations, IDT for AWS IoT Greengrass must be able to run commands with sudo without being prompted for a password\.

Follow these steps on the device under test to allow sudo access without being prompted for a password\. 

**Note**  
`username` refers to the SSH user used by IDT to access the device under test\.

**To add the user to the sudo group**

1. On the device under test, run `sudo usermod -aG sudo <username>`\.

1. Sign out and then sign back in for changes to take effect\.

1. To verify your user name was added successfully, run sudo echo test\. If you are not prompted for a password, your user is configured correctly\.

1. Open the `/etc/sudoers` file and add the following line to the end of the file:

   `<ssh-username> ALL=(ALL) NOPASSWD: ALL`

## Configure your device to test optional features<a name="optional-feature-config"></a>

The following topics describe how to configure your devices to run IDT tests for optional features\. Follow these configuration steps only if you want to test these features\. Otherwise, continue to [Setting configuration to run the AWS IoT Greengrass qualification suite](set-config.md)\.

**Topics**
+ [Verify AWS IoT Greengrass dependencies on the device under test](#install-gg-dependencies)
+ [Configure the AWS IoT Greengrass software](#config-gg)
+ [Configure your host computer to access your device under test](#configure-host)
+ [Configure user permissions on your device](#root-access)
+ [Configure your device to test optional features](#optional-feature-config)
+ [Optional: Configuring your Docker container for IDT for AWS IoT Greengrass](docker-config-setup.md)
+ [Optional: Configuring your device for ML qualification](idt-ml-qualification.md)