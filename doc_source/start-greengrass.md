--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Install and run AWS IoT Greengrass on the core device<a name="start-greengrass"></a>

**Note**  
This tutorial provides instructions for you to run the AWS IoT Greengrass Core software on a Raspberry Pi, but you can use any supported device\.

In this section, you configure, install, and run the AWS IoT Greengrass Core software on your core device\.

**To install and run AWS IoT Greengrass**

1. From the [AWS IoT Greengrass Core software](what-is-gg.md#gg-core-download-tab) section in this guide, download the AWS IoT Greengrass Core software installation package\. Choose the package that best fits the CPU architecture, distribution, and OS of your core device\.
   + For Raspberry Pi, download the package for the Armv7l architecture and Linux operating system\.
   + For an Amazon EC2 instance, download the package for the x86\_64 architecture and Linux operating system\.
   + For NVIDIA Jetson TX2, download the package for the Armv8 \(AArch64\) architecture and Linux operating system\.
   + For Intel Atom, download the package for the x86\_64 architecture and Linux operating system\.

1. In previous steps, you downloaded five files to your computer:
   + `greengrass-OS-architecture-1.11.6.tar.gz` – This compressed file contains the AWS IoT Greengrass Core software that runs on the core device\.
   + `certificateId-certificate.pem.crt` – The device certificate file\.
   + `certificateId-public.pem.key` – The device certificate's public key file\.
   + `certificateId-private.pem.key` – The device certificate's private key file\.
   + `AmazonRootCA1.pem` – The Amazon root certificate authority \(CA\) file\.

   In this step, you transfer these files from your computer to your core device\. Do the following:

   1. If you don't know the IP address of your Greengrass core device, open a terminal on the core device and run the following command\.
**Note**  
This command might not return the correct IP address for some devices\. Consult the documentation for your device to retrieve your device IP address\.

      ```
      hostname -I
      ```

   1. <a name="transfer-files-to-device"></a>Transfer these files from your computer to your core device\. The file transfer steps vary depending on the operating system of your computer\. Choose your operating system for steps that show how to transfer files to your Raspberry Pi device\.
**Note**  
For a Raspberry Pi, the default user name is **pi** and the default password is **raspberry**\.  
For an NVIDIA Jetson TX2, the default user name is **nvidia** and the default password is **nvidia**\.

------
#### [ Windows ]

      To transfer the compressed files from your computer to a Raspberry Pi core device, use a tool such as [WinSCP](https://winscp.net/eng/download.php) or the [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) pscp command\. To use the pscp command, open a Command Prompt window on your computer and run the following:

      ```
      cd path-to-downloaded-files
      pscp -pw Pi-password greengrass-OS-architecture-1.11.6.tar.gz pi@IP-address:/home/pi
      pscp -pw Pi-password certificateId-certificate.pem.crt pi@IP-address:/home/pi
      pscp -pw Pi-password certificateId-public.pem.key pi@IP-address:/home/pi
      pscp -pw Pi-password certificateId-private.pem.key pi@IP-address:/home/pi
      pscp -pw Pi-password AmazonRootCA1.pem pi@IP-address:/home/pi
      ```

**Note**  
<a name="use-correct-package-version"></a>The version number in this command must match the version of your AWS IoT Greengrass Core software package\.

------
#### [ macOS ]

      To transfer the compressed files from your Mac to a Raspberry Pi core device, open a Terminal window on your computer and run the following commands\. The *path\-to\-downloaded\-files* is typically `~/Downloads`\.

**Note**  
You might be prompted for two passwords\. If so, the first password is for the Mac's `sudo` command and the second is the password for the Raspberry Pi\.

      ```
      cd path-to-downloaded-files
      scp greengrass-OS-architecture-1.11.6.tar.gz pi@IP-address:/home/pi
      scp certificateId-certificate.pem.crt pi@IP-address:/home/pi
      scp certificateId-public.pem.key pi@IP-address:/home/pi
      scp certificateId-private.pem.key pi@IP-address:/home/pi
      scp AmazonRootCA1.pem pi@IP-address:/home/pi
      ```

**Note**  
<a name="use-correct-package-version"></a>The version number in this command must match the version of your AWS IoT Greengrass Core software package\.

------
#### [ UNIX\-like system ]

      To transfer the compressed files from your computer to a Raspberry Pi core device, open a terminal window on your computer and run the following commands:

      ```
      cd path-to-downloaded-files
      scp greengrass-OS-architecture-1.11.6.tar.gz pi@IP-address:/home/pi
      scp certificateId-certificate.pem.crt pi@IP-address:/home/pi
      scp certificateId-public.pem.key pi@IP-address:/home/pi
      scp certificateId-private.pem.key pi@IP-address:/home/pi
      scp AmazonRootCA1.pem pi@IP-address:/home/pi
      ```

**Note**  
<a name="use-correct-package-version"></a>The version number in this command must match the version of your AWS IoT Greengrass Core software package\.

------
#### [ Raspberry Pi web browser ]

      If you used the Raspberry Pi's web browser to download the compressed files, the files should be in the Pi's `~/Downloads` folder, such as `/home/pi/Downloads`\. Otherwise, the compressed files should be in the Pi's `~` folder, such as `/home/pi`\.

------

1. On the Greengrass core device, open a terminal, and navigate to the folder that contains the AWS IoT Greengrass Core software and certificates\. Replace *path\-to\-transferred\-files* with the path where you transferred the files on the core device\. For example, on a Raspberry Pi, run `cd /home/pi`\.

   ```
   cd path-to-transferred-files
   ```

1. Unpack the AWS IoT Greengrass Core software on the core device\. Run the following command to unpack the software archive that you transferred to the core device\. This command uses the `-C /` argument to create the `/greengrass` folder in the root folder of the core device\.

   ```
   sudo tar -xzvf greengrass-OS-architecture-1.11.6.tar.gz -C /
   ```
**Note**  
<a name="use-correct-package-version"></a>The version number in this command must match the version of your AWS IoT Greengrass Core software package\.

1. Move the certificates and keys to the AWS IoT Greengrass Core software folder\. Run the following commands to create a folder for certificates and move the certificates and keys to it\. Replace *path\-to\-transferred\-files* with the path where you transferred the files on the core device, and replace *certificateId* with the certificate ID in the file names\. For example, on a Raspberry Pi, replace *path\-to\-transferred\-files* with **/home/pi**

   ```
   sudo mv path-to-transferred-files/certificateId-certificate.pem.crt /greengrass/certs
   sudo mv path-to-transferred-files/certificateId-public.pem.key /greengrass/certs
   sudo mv path-to-transferred-files/certificateId-private.pem.key /greengrass/certs
   sudo mv path-to-transferred-files/AmazonRootCA1.pem /greengrass/certs
   ```

1. The AWS IoT Greengrass Core software uses a configuration file that specifies parameters for the software\. This configuration file specifies the file paths for certificate files and the AWS Cloud endpoints to use\. In this step, you create the AWS IoT Greengrass Core software configuration file for your core\. Do the following:

   1. Get the Amazon Resource Name \(ARN\) for your core's AWS IoT thing\. Do the following:

      1. In the [AWS IoT console](https://console.aws.amazon.com/iot), under **Manage**, under **Greengrass devices**, choose **Groups \(V1\)**\.

      1. On the **Greengrass groups** page, choose the group that you created earlier\.

      1. Under **Overview**, choose **Greengrass core**\.

      1. On the core details page, copy the **AWS IoT thing ARN**, and save it to use in the AWS IoT Greengrass Core configuration file\.

   1. Get the AWS IoT device data endpoint for your AWS account in the current Region\. Devices use this endpoint to connect to AWS as AWS IoT things\. Do the following:

      1. In the [AWS IoT console](https://console.aws.amazon.com/iot), choose **Settings**\.

      1. Under **Device data endpoint**, copy the **Endpoint**, and save it to use in the AWS IoT Greengrass Core configuration file\.

   1. Create the AWS IoT Greengrass Core software configuration file\. For example, you can run the following command to use GNU nano to create the file\.

      ```
      sudo nano /greengrass/config/config.json
      ```

      Replace the contents of the file with the following JSON document\.

      ```
      {
        "coreThing" : {
          "caPath": "AmazonRootCA1.pem",
          "certPath": "certificateId-certificate.pem.crt",
          "keyPath": "certificateId-private.pem.key",
          "thingArn": "arn:aws:iot:region:account-id:thing/MyGreengrassV1Core",
          "iotHost": "device-data-prefix-ats.iot.region.amazonaws.com",
          "ggHost": "greengrass-ats.iot.region.amazonaws.com",
          "keepAlive": 600
        },
        "runtime": {
          "cgroup": {
            "useSystemd": "yes"
          }
        },
        "managedRespawn": false,
        "crypto": {
          "caPath": "file:///greengrass/certs/AmazonRootCA1.pem",
          "principals": {
            "SecretsManager": {
              "privateKeyPath": "file:///greengrass/certs/certificateId-private.pem.key"
            },
            "IoTCertificate": {
              "privateKeyPath": "file:///greengrass/certs/certificateId-private.pem.key",
              "certificatePath": "file:///greengrass/certs/certificateId-certificate.pem.crt"
            }
          }
        }
      }
      ```

      Then, do the following:
      + If you downloaded a different Amazon root CA certificate than Amazon Root CA 1, replace each instance of *AmazonRootCA1\.pem* with the name of the Amazon root CA file\.
      + Replace each instance of *certificateId* with the certificate ID in the name of the certificate and key files\.
      + Replace *arn:aws:iot:*region*:*account\-id*:thing/MyGreengrassV1Core* with the ARN of your core's thing that you saved earlier\.
      + Replace *MyGreengrassV1core* with the name of your core's thing\.
      + Replace *device\-data\-prefix\-ats\.iot\.region\.amazonaws\.com* with the AWS IoT device data endpoint that you saved earlier\.
      + Replace *region* with your AWS Region\.

      For more information about the configuration options that you can specify in this configuration file, see [AWS IoT Greengrass core configuration file](gg-core.md#config-json)\.

1. Make sure that your core device is connected to the internet\. Then, start AWS IoT Greengrass on your core device\.

   ```
   cd /greengrass/ggc/core/
   sudo ./greengrassd start
   ```

   You should see a `Greengrass successfully started` message\. Make a note of the PID\.
**Note**  
To set up your core device to start AWS IoT Greengrass on system boot, see [Configure the init system to start the Greengrass daemon](gg-core.md#start-on-boot)\.

   You can run the following command to confirm that the AWS IoT Greengrass Core software \(Greengrass daemon\) is functioning\. Replace *PID\-number* with your PID:

   ```
   ps aux | grep PID-number
   ```

   You should see an entry for the PID with a path to the running Greengrass daemon \(for example, `/greengrass/ggc/packages/1.11.6/bin/daemon`\)\. If you run into issues starting AWS IoT Greengrass, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.