# Start AWS IoT Greengrass on the core device<a name="gg-device-start"></a>

**Note**  
This tutorial provides instructions for starting AWS IoT Greengrass on your Raspberry Pi, but you can use any supported device\.

In the previous steps, you downloaded two files to your computer:
+ `hash-setup.tar.gz` \(for example, `c6973960cc-setup.tar.gz`\)\. This compressed file contains the core device certificate and cryptographic keys that enable secure communications between AWS IoT Core and the `config.json` file that contains configuration information specific to your Greengrass core\. This information includes the location of certificate files and the AWS IoT Core endpoint\.
+ `greengrass-OS-architecture-1.11.0.tar.gz`\. This compressed file contains the AWS IoT Greengrass Core software that runs on the core device\.

 

1. If you don't know the IP address of your Greengrass core device, open a terminal on the core device and run the following command:
**Note**  
This command might not return the correct IP address for some devices\. Consult the documentation for your device to retrieve your device IP address\.

   ```
   hostname -I
   ```

1. Transfer the two compressed files from your computer to the Greengrass core device\. Choose your operating system for steps that show how to transfer files to your Raspberry Pi device\. The file transfer steps vary, depending on device  or EC2 instance\.
**Note**  
For a Raspberry Pi, the default user name is **pi** and the default password is **raspberry**\.  
For an NVIDIA Jetson TX2, the default user name is **nvidia** and the default password is **nvidia**\.

------
#### [ Windows ]

   To transfer the compressed files from your computer to a Raspberry Pi core device, use a tool such as [WinSCP](https://winscp.net/eng/download.php) or the [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) pscp command\. To use the pscp command, open a Command Prompt window on your computer and run the following:

   ```
   cd path-to-downloaded-files
   pscp -pw Pi-password greengrass-OS-architecture-1.11.0.tar.gz pi@IP-address:/home/pi
   pscp -pw Pi-password hash-setup.tar.gz pi@IP-address:/home/pi
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
   scp greengrass-OS-architecture-1.11.0.tar.gz pi@IP-address:/home/pi
   scp hash-setup.tar.gz pi@IP-address:/home/pi
   ```

**Note**  
<a name="use-correct-package-version"></a>The version number in this command must match the version of your AWS IoT Greengrass Core software package\.

------
#### [ UNIX\-like system ]

   To transfer the compressed files from your computer to a Raspberry Pi core device, open a terminal window on your computer and run the following commands:

   ```
   cd path-to-downloaded-files
   scp greengrass-OS-architecture-1.11.0.tar.gz pi@IP-address:/home/pi
   scp hash-setup.tar.gz pi@IP-address:/home/pi
   ```

**Note**  
<a name="use-correct-package-version"></a>The version number in this command must match the version of your AWS IoT Greengrass Core software package\.

------
#### [ Raspberry Pi web browser ]

   If you used the Raspberry Pi's web browser to download the compressed files, the files should be in the Pi's `~/Downloads` folder \(for example, `/home/pi/Downloads`\)\. Otherwise, the compressed files should be in the Pi's `~` folder \(for example, `/home/pi`\)\.

------

1. Open a terminal on the Greengrass core device and navigate to the folder that contains the compressed files \(for example, `cd /home/pi`\)\.

   ```
   cd path-to-compressed-files
   ```

1. Install the AWS IoT Greengrass Core software and the security resources\.
   + The first command creates the `/greengrass` directory in the root folder of the core device \(through the `-C /` argument\)\.
   + The second command copies the core device certificate and keys into the `/greengrass/certs` folder and the [`config.json`](gg-core.md#config-json) file into the `/greengrass/config` folder \(through the `-C /greengrass` argument\)\.

   ```
   sudo tar -xzvf greengrass-OS-architecture-1.11.0.tar.gz -C /
   sudo tar -xzvf hash-setup.tar.gz -C /greengrass
   ```
**Note**  
<a name="use-correct-package-version"></a>The version number in this command must match the version of your AWS IoT Greengrass Core software package\.

1. Make sure that your core device is connected to the internet\. Then, download the root CA certificate to the `/greengrass/certs` folder on the device\.

   Review [Server Authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) in the *AWS IoT Developer Guide* and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\. Certificates enable your device to communicate with AWS IoT Core using the MQTT messaging protocol over TLS\.

   For example, run the following commands to download the Amazon Root CA 1 certificate and rename it to `root.ca.pem`\. This is the file name registered in the `config.json` that you downloaded from the console\.

   ```
   cd /greengrass/certs/
   sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
   ```

   You can run the following command to confirm that `root.ca.pem` is not empty\. If the file is empty, check the `wget` URL and try again\.

   ```
   cat root.ca.pem
   ```
**Note**  
Your root CA certificate type must match your endpoint\. If you configure the core to use legacy authentication endpoints, download a [VeriSign root CA certificate](https://www.websecurity.digicert.com/content/dam/websitesecurity/digitalassets/desktop/pdfs/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem) instead\. Although legacy endpoints are acceptable for the purposes of this tutorial, we recommend that you use an ATS endpoint and download an ATS root CA certificate\.

1. Start AWS IoT Greengrass on your core device\.

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

   You should see an entry for the PID with a path to the running Greengrass daemon \(for example, `/greengrass/ggc/packages/1.11.0/bin/daemon`\)\. If you run into issues starting AWS IoT Greengrass, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.