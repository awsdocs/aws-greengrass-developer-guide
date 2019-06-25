# Start AWS IoT Greengrass on the Core Device<a name="gg-device-start"></a>

**Note**  
This tutorial provides instructions for starting AWS IoT Greengrass on your Raspberry Pi, but you can use any supported device\.

In the [previous step](gg-config.md#gg-core-download), you downloaded two files to your computer:
+ `greengrass-OS-architecture-1.9.2.tar.gz`\. This compressed file contains the AWS IoT Greengrass Core software that runs on the core device\.
+ `hash-setup.tar.gz`\. This compressed file contains security certificates that enable secure communications between AWS IoT and the `config.json` file that contains configuration information specific to your AWS IoT Greengrass core and the AWS IoT endpoint\.

1. If you don't know the IP address of your AWS IoT Greengrass core device, open a terminal on the AWS IoT Greengrass core device and run the following command:
**Note**  
This command might not return the correct IP address for some devices\. Consult the documentation for your device to retrieve your device IP address\.

   ```
   hostname -I
   ```

1. Transfer the two compressed files from your computer to the AWS IoT Greengrass core device\. Choose your operating system for steps that show how to transfer files to your Raspberry Pi device\. The file transfer steps vary, depending on device  or EC2 instance\.
**Note**  
For a Raspberry Pi, the default user name is **pi** and the default password is **raspberry**\.  
For an NVIDIA Jetson TX2, the default user name is **nvidia** and the default password is **nvidia**\.

------
#### [ Windows ]

   To transfer the compressed files from your computer to a Raspberry Pi AWS IoT Greengrass core device, use a tool such as [WinSCP](https://winscp.net/eng/download.php) or the [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) pscp command\. To use the pscp command, open a Command Prompt window on your computer and run the following:

   ```
   cd path-to-downloaded-files
   pscp -pw Pi-password greengrass-OS-architecture-1.9.2.tar.gz pi@IP-address:/home/pi
   pscp -pw Pi-password hash-setup.tar.gz pi@IP-address:/home/pi
   ```

   For example:

![\[PuTTY's pscp command in use.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.5.png)

------
#### [ macOS ]

   To transfer the compressed files from your Mac to a Raspberry Pi AWS IoT Greengrass core device, open a Terminal window on your computer and run the following commands\. The *path\-to\-downloaded\-files* is typically `~/Downloads`\.

**Note**  
You might be prompted for two passwords\. If so, the first password is for the Mac's `sudo` command and the second is the password for the Raspberry Pi\.

   ```
   cd path-to-downloaded-files
   scp greengrass-OS-architecture-1.9.2.tar.gz pi@IP-address:/home/pi
   scp hash-setup.tar.gz pi@IP-address:/home/pi
   ```

------
#### [ UNIX\-like system ]

   To transfer the compressed files from your computer to a Raspberry Pi AWS IoT Greengrass core device, open a terminal window on your computer and run the following commands:

   ```
   cd path-to-downloaded-files
   scp greengrass-OS-architecture-1.9.2.tar.gz pi@IP-address:/home/pi
   scp hash-setup.tar.gz pi@IP-address:/home/pi
   ```

------
#### [ Raspberry Pi web browser ]

   If you used the Raspberry Pi's web browser to download the compressed files, the files should be in the Pi's `~/Downloads` folder \(for example, `/home/pi/Downloads`\)\. Otherwise, the compressed files should be in the Pi's `~` folder \(for example, `/home/pi`\)\.

------

1. Open a terminal on the AWS IoT Greengrass core device and navigate to the folder that contains the compressed files \(for example, `cd /home/pi`\)\.

   ```
   cd path-to-compressed-files
   ```

1. Decompress the AWS IoT Greengrass Core software and the security resources\.
   + The first command creates the `/greengrass` directory in the root folder of the AWS IoT Greengrass core device \(through the `-C /` argument\)\.
   + The second command copies the certificates into the `/greengrass/certs` folder and the [`config.json`](gg-core.md#config-json) file into the `/greengrass/config` folder \(through the `-C /greengrass` argument\)\.

   ```
   sudo tar -xzvf greengrass-OS-architecture-1.9.2.tar.gz -C /
   sudo tar -xzvf hash-setup.tar.gz -C /greengrass
   ```

1. Review the documentation about [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html#server-authentication) and choose the appropriate root CA certificate\. We recommend that you use Amazon Trust Services \(ATS\) endpoints and ATS root CA certificates\. Certificates enable your device to communicate with AWS IoT using the MQTT messaging protocol over TLS\.

   Make sure that the AWS IoT Greengrass core device is connected to the internet, and then download the root CA certificate to your core device\.
**Important**  
Your root CA certificate type must match your endpoint\. Use an ATS root CA certificate with an ATS endpoint \(preferred\) or a Verisign root CA certificate with a legacy endpoint\. For more information, see [Endpoints Must Match the Certificate Type](gg-core.md#certificate-endpoints)\.

   For ATS endpoints \(preferred\), download the appropriate ATS root CA certificate\. The following example downloads `AmazonRootCA1.pem`\. The `wget -O` parameter is the capital letter O\.

   ```
   cd /greengrass/certs/
   sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
   ```
**Note**  
For legacy endpoints, download a Verisign root CA certificate\. Although legacy endpoints are acceptable for the purposes of this tutorial, we recommend that you create an ATS endpoint and download an ATS root CA certificate\.  

   ```
   cd /greengrass/certs/
   sudo wget -O root.ca.pem https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem
   ```

   You can run the following command to confirm that the `root.ca.pem` file is not empty:

   ```
   cat root.ca.pem
   ```

   If the `root.ca.pem` file is empty, check the `wget` URL and try again\.

1. Start AWS IoT Greengrass on your core device\.

   ```
   cd /greengrass/ggc/core/
   sudo ./greengrassd start
   ```

   You should see a `Greengrass successfully started` message\. Make a note of the PID\.
**Note**  
To set up your core device to start AWS IoT Greengrass on system boot, see [Configure the Init System to Start the Greengrass Daemon](gg-core.md#start-on-boot)\.

   You can run the following command to confirm that the AWS IoT Greengrass Core software \(daemon\) is functioning\. Replace *PID\-number* with your PID:

   ```
   ps aux | grep PID-number
   ```

   You should see an entry for the PID with a path to the running Greengrass daemon \(for example, `/greengrass/ggc/packages/1.9.2/bin/daemon`\)\. If you run into issues starting AWS IoT Greengrass, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.