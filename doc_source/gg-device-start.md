# Start AWS Greengrass on the Core Device<a name="gg-device-start"></a>

1. In the [prior step](gg-config.md#gg-core-download), you downloaded two files from the AWS Greengrass console:
   + `greengrass-OS-architecture-1.6.0.tar.gz` \- this compressed file contains the AWS Greengrass core software that runs on the AWS Greengrass core device\.
   + `GUID-setup.tar.gz` \- this compressed file contains security certificates enabling secure communications with the AWS IoT cloud and `config.json` which contains configuration information specific to your AWS Greengrass core and the AWS IoT endpoint\.

   If you don't recall the IP address of your AWS Greengrass core device, open a terminal on the AWS Greengrass core device and run the following command:

   ```
   hostname -I
   ```

   Based on your operating system, choose a tab to transfer the two compressed files from your computer to the AWS Greengrass core device:
**Note**  
Recall that the default login and password for the Raspberry Pi is **pi** and **raspberry**, respectively\.

------
#### [ Windows ]

   To transfer the compressed files from your computer to a Raspberry Pi AWS Greengrass core device, use a convenient tool such as [WinSCP](https://winscp.net/eng/download.php) or [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)'s `pscp` command\. To use the `pscp` command, open a Command Prompt window on your computer and run the following:

   ```
   cd path-to-downloaded-files
   pscp -pw Pi-password greengrass-OS-architecture-1.6.0.tar.gz pi@IP-address:/home/pi
   pscp -pw Pi-password GUID-setup.tar.gz pi@IP-address:/home/pi
   ```

   For example:

![\[PuTTY's pscp command in use.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.5.png)

------
#### [ macOS ]

   To transfer the compressed files from your Mac to a Raspberry Pi AWS Greengrass core device, open a Terminal window on your computer and run the following commands \(note that *path\-to\-downloaded\-files* is typically `~/Downloads`\)\.

**Note**  
You may be prompted for two passwords\. If so, the first password is for the Mac's `sudo` command and the second will be the password for the Raspberry Pi\.

   ```
   cd path-to-downloaded-files
   sudo scp greengrass-OS-architecture-1.6.0.tar.gz pi@IP-address:/home/pi
   sudo scp GUID-setup.tar.gz pi@IP-address:/home/pi
   ```

------
#### [ UNIX\-like system ]

   To transfer the compressed files from your computer to a Raspberry Pi AWS Greengrass core device, open a terminal window on your computer and run the following commands:

   ```
   cd path-to-downloaded-files
   sudo scp greengrass-OS-architecture-1.6.0.tar.gz pi@IP-address:/home/pi
   sudo scp GUID-setup.tar.gz pi@IP-address:/home/pi
   ```

------
#### [ Raspberry Pi web browser ]

   If you used the Raspberry Pi's web browser to download the compressed files, the files should be in the Pi's `~/Downloads` folder \(i\.e\., `/home/pi/Downloads`\)\. Otherwise, the compressed files should be in the Pi's `~` folder \(i\.e\., `/home/pi`\)\.

------

   Open a terminal on the AWS Greengrass core device and navigate to the folder containing the compressed files \(i\.e\., *path\-to\-compressed\-files*\)\.

   ```
   cd path-to-compressed-files
   ```

   Next, run the following commands to decompress the AWS Greengrass core binary file and the security resources \(certificates, etc\.\) file:

   ```
   sudo tar -xzvf greengrass-OS-architecture-1.6.0.tar.gz -C /
   sudo tar -xzvf GUID-setup.tar.gz -C /greengrass
   ```

   Among other things, the first command creates the `/greengrass` directory in the root folder of the AWS Greengrass core device \(via the `-C /` argument\)\. The second command copies the certificates into the `/greengrass/certs` folder and the `config.json` file into the `/greengrass/config` folder \(via the `-C /greengrass` argument\)\. For more information, see [AWS Greengrass Core Configuration File](gg-core.md#config-json)\.

1. Install the [Symantec VeriSign root CA](http://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem) onto your device\. This certificate enables your device to communicate with AWS IoT using the MQTT messaging protocol over TLS\. Make sure the AWS Greengrass core device is connected to the internet, then run the following commands \(note that `-O` is the capital letter `O`\):

   ```
   cd /greengrass/certs/
   sudo wget -O root.ca.pem http://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem
   ```

   Run the following command to confirm that the `root.ca.pem` file is not empty:

   ```
   cat root.ca.pem
   ```

   If the `root.ca.pem` file is empty, check the `wget` URL and try again\.

1. Use the following commands to start AWS Greengrass\.

   ```
   cd /greengrass/ggc/core/
   sudo ./greengrassd start
   ```

   You should see output similar to the following \(note the PID number\):  
![\[Screenshot of Raspberry Pi output showing AWS Greengrass running as indicated by "Greengrass successfully started with PID: 2244".\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-010.png)
**Note**  
To set up your core device to start AWS Greengrass on system boot, see [Configure the Init System to Start the Greengrass Daemon](gg-core.md#start-on-boot)\.

   Next, run the following command to confirm that the AWS Greengrass core software \(daemon\) is functioning\. Replace *PID\-number* with your own PID number:

   ```
   ps aux | grep PID-number
   ```

   You should see a path to the running AWS Greengrass daemon, as in `/greengrass/ggc/packages/1.6.0/bin/daemon`\. If you run into issues starting AWS Greengrass, see [Troubleshooting AWS Greengrass Applications](gg-troubleshooting.md)\.