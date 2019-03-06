# Start AWS IoT Greengrass on the Core Device<a name="gg-device-start"></a>

In the [previous step](gg-config.md#gg-core-download), you downloaded two files to your computer:
+ `greengrass-OS-architecture-1.7.1.tar.gz`\. This compressed file contains the AWS IoT Greengrass core software that runs on the core device\.
+ `hash-setup.tar.gz`\. This compressed file contains security certificates that enable secure communications between AWS IoT and the `config.json` file that contains configuration information specific to your AWS IoT Greengrass core and the AWS IoT endpoint\.

1. If you don't recall the IP address of your AWS IoT Greengrass core device, open a terminal on the AWS IoT Greengrass core device and run the following command:

   ```
   hostname -I
   ```

1. Transfer the two compressed files from your computer to the AWS IoT Greengrass core device\. For steps that show how to transfer files, choose your operating system:
**Note**  
For a Raspberry Pi, the default user name is **pi** and the default password is **raspberry**\.  
For an NVIDIA Jetson TX2, the default user name is **nvidia** and the default password is **nvidia**\.

------
#### [ Windows ]

   To transfer the compressed files from your computer to a Raspberry Pi AWS IoT Greengrass core device, use a tool such as [WinSCP](https://winscp.net/eng/download.php) or the [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) pscp command\. To use the pscp command, open a Command Prompt window on your computer and run the following:

   ```
   cd path-to-downloaded-files
   pscp -pw Pi-password greengrass-OS-architecture-1.7.1.tar.gz pi@IP-address:/home/pi
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
   sudo scp greengrass-OS-architecture-1.7.1.tar.gz pi@IP-address:/home/pi
   sudo scp hash-setup.tar.gz pi@IP-address:/home/pi
   ```

------
#### [ UNIX\-like system ]

   To transfer the compressed files from your computer to a Raspberry Pi AWS IoT Greengrass core device, open a terminal window on your computer and run the following commands:

   ```
   cd path-to-downloaded-files
   sudo scp greengrass-OS-architecture-1.7.1.tar.gz pi@IP-address:/home/pi
   sudo scp hash-setup.tar.gz pi@IP-address:/home/pi
   ```

------
#### [ Raspberry Pi web browser ]

   If you used the Raspberry Pi's web browser to download the compressed files, the files should be in the Pi's `~/Downloads` folder \(for example, `/home/pi/Downloads`\)\. Otherwise, the compressed files should be in the Pi's `~` folder \(for example, `/home/pi`\)\.

------

1. Open a terminal on the AWS IoT Greengrass core device and navigate to the folder that contains the compressed files \(for example, `cd /home/pi`\)\.

   ```
   cd path-to-compressed-files
   ```

1. Decompress the AWS IoT Greengrass core software and the security resources\.
   + The first command creates the `/greengrass` directory in the root folder of the AWS IoT Greengrass core device \(through the `-C /` argument\)\.
   + The second command copies the certificates into the `/greengrass/certs` folder and the [`config.json`](gg-core.md#config-json) file into the `/greengrass/config` folder \(through the `-C /greengrass` argument\)\.

   ```
   sudo tar -xzvf greengrass-OS-architecture-1.7.1.tar.gz -C /
   sudo tar -xzvf hash-setup.tar.gz -C /greengrass
   ```

1. Review the documentation on [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html) and choose the appropriate CA certificate\. Certificates enable your device to communicate with AWS IoT using the MQTT messaging protocol over TLS\.

   Make sure that the AWS IoT Greengrass core device is connected to the internet, and then run the following commands to copy the root CA certificate to your device\. This example uses `AmazonRootCA1.pem`\.

   ```
   cd /greengrass/certs/
   sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
   ```
**Note**  
The `wget -O` parameter is the capital letter O\.

   Run the following command to confirm that the `root.ca.pem` file is not empty:

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

   You can run the following command to confirm that the AWS IoT Greengrass core software \(daemon\) is functioning\. Replace *PID\-number* with your PID:

   ```
   ps aux | grep PID-number
   ```

   You should see an entry for the PID with a path to the running Greengrass daemon \(for example, `/greengrass/ggc/packages/1.7.1/bin/daemon`\)\. If you run into issues starting AWS IoT Greengrass, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.