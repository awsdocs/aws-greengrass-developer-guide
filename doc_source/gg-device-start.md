# Start AWS Greengrass on the Core Device<a name="gg-device-start"></a>

1. In the [prior step](gg-config.md#gg-core-download), you downloaded two files from the AWS Greengrass console:

   + `greengrass-platform-1.3.0.tar.gz` \- this compressed file contains the AWS Greengrass core software that runs on the AWS Greengrass core device\.

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
   pscp -pw Pi-password greengrass-platform-1.3.0.tar.gz pi@IP-address:/home/pi
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
   sudo scp greengrass-platform-1.3.0.tar.gz pi@IP-address:/home/pi
   sudo scp GUID-setup.tar.gz pi@IP-address:/home/pi
   ```

------
#### [ UNIX\-like system ]

   To transfer the compressed files from your computer to a Raspberry Pi AWS Greengrass core device, open a terminal window on your computer and run the following commands:

   ```
   cd path-to-downloaded-files
   sudo scp greengrass-platform-1.3.0.tar.gz pi@IP-address:/home/pi
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
   sudo tar -xzvf greengrass-platform-1.3.0.tar.gz -C /
   sudo tar -xzvf GUID-setup.tar.gz -C /greengrass
   ```

   Among other things, the first command creates the `/greengrass` directory in the root folder of the AWS Greengrass core device \(via the `-C /` argument\)\. The second command copies the certificates into the `/greengrass/certs` folder and the `config.json` file into the `/greengrass/config` folder \(via the `-C /greengrass` argument\)\. For more information, see [`config.json` Parameter Summary](#config.json-params)\.

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
   cd /greengrass/ggc/packages/1.3.0/
   sudo ./greengrassd start
   ```

   You should see output similar to the following \(note the PID number\):  
![\[Screenshot of Raspberry Pi output showing AWS Greengrass running as indicated by "Greengrass successfully started with PID: 2244".\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-010.png)

   Next, run the following command to confirm that the AWS Greengrass core software \(daemon\) is functioning\. Replace *PID\-number* with your own PID number:

   ```
   ps aux | grep PID-number
   ```

   You should see a path to the running AWS Greengrass daemon, as in `/greengrass/ggc/packages/1.3.0/bin/daemon`\. If you run into issues starting AWS Greengrass, see [Troubleshooting AWS Greengrass Applications](gg-troubleshooting.md)\.

## `config.json` Parameter Summary<a name="config.json-params"></a>

The AWS Greengrass `config.json` file is contained in the `/greengrass/config` directory \(or `/greengrass/configuration` for AWS Greengrass version 1\.0\.0\) and should be ready to go as\-is\. You can optionally review the contents of this file by running the following command \(replace `config` with `configuration` for v1\.0\.0\):

```
cat /greengrass/config/config.json
```

------
#### [ GGC v1\.3\.0 ]

```
{
   "coreThing": {
       "caPath": "ROOT_CA_PEM_HERE",
       "certPath": "CLOUD_PEM_CRT_HERE",
       "keyPath": "CLOUD_PEM_KEY_HERE",
       "thingArn": "THING_ARN_HERE",
       "iotHost": "HOST_PREFIX_HERE.iot.AWS_REGION_HERE.amazonaws.com",
       "ggHost": "greengrass.iot.AWS_REGION_HERE.amazonaws.com",
       "keepAlive": 600
   },
   "runtime": {
       "cgroup": {
           "useSystemd": "yes|no"
       }
   },
   "managedRespawn": true
}
```

The `config.json` file appears in `/greengrass/config/` and contains the following parameters:


****  

| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the [AWS IoT root CA](http://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem) relative to the `/greengrass/certs` folder\.  |  Save the file under the `/greengrass/certs` folder\.  | 
| certPath |  The path to the AWS Greengrass core certificate relative to the `/greengrass/certs` folder\.  | Save the file under the /greengrass/certs folder\. | 
| keyPath | The path to the AWS Greengrass core private key relative to /greengrass/certs folder\. | Save the file under the /greengrass/certs folder\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS Greengrass core\.  | You can find it in the AWS Greengrass console under the definition for your AWS IoT thing\. | 
| iotHost | Your AWS IoT endpoint\. | You can find it in the AWS IoT console under Settings\. | 
| ggHost | Your AWS endpoint\. | You can find it in the AWS IoT console under Settings with greengrass\. prepended\. | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default value is 600 seconds\. | 
| useSystemd | A binary flag, if your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Values are yes or no\. Use the dependency script in [Module 1](module1.md) to see if your device uses systemd\. | 
|  `managedRespawn`  |  An optional over\-the\-air \(OTA\) updates feature, this indicates that the OTA agent needs to run custom code before an update\.  |  For more information, see [OTA Updates of AWS Greengrass Core Software](core-ota-update.md)\.  | 

------
#### [ GGC v1\.1\.0 ]

```
{
   "coreThing": {
       "caPath": "ROOT_CA_PEM_HERE",
       "certPath": "CLOUD_PEM_CRT_HERE",
       "keyPath": "CLOUD_PEM_KEY_HERE",
       "thingArn": "THING_ARN_HERE",
       "iotHost": "HOST_PREFIX_HERE.iot.AWS_REGION_HERE.amazonaws.com",
       "ggHost": "greengrass.iot.AWS_REGION_HERE.amazonaws.com",
       "keepAlive": 600
   },
   "runtime": {
       "cgroup": {
           "useSystemd": "yes|no"
       }
   }
}
```

The `config.json` file exists in `/greengrass/config/` and contains the following parameters:


****  

| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the [AWS IoT root CA](http://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem) relative to the `/greengrass/certs` folder\.  |  Save the file under the `/greengrass/certs` folder\.  | 
| certPath |  The path to the AWS Greengrass core certificate relative to the `/greengrass/certs` folder\.  | Save the file under the /greengrass/certs folder\. | 
| keyPath | The path to the AWS Greengrass core private key relative to the /greengrass/certs folder\. | Save the file under the /greengrass/certs folder\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS Greengrass core\.  | You can find it in the AWS Greengrass console under the definition for your AWS IoT thing\. | 
| iotHost | Your AWS IoT endpoint\. | You can find it in the AWS IoT console under Settings\. | 
| ggHost | Your AWS endpoint\. | You can find it in the AWS IoT console under Settings with greengrass\. prepended\. | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default value is 600 seconds\. | 
| useSystemd | A binary flag, if your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Values are yes or no\. Use the dependency script in [Module 1](module1.md) to see if your device uses systemd\. | 

------
#### [ GGC v1\.0\.0 ]

```
{
   "coreThing": {
       "caPath": "ROOT_CA_PEM_HERE",
       "certPath": "CLOUD_PEM_CRT_HERE",
       "keyPath": "CLOUD_PEM_KEY_HERE",
       "thingArn": "THING_ARN_HERE",
       "iotHost": "HOST_PREFIX_HERE.iot.AWS_REGION_HERE.amazonaws.com",
       "ggHost": "greengrass.iot.AWS_REGION_HERE.amazonaws.com",
       "keepAlive": 600
   },
   "runtime": {
       "cgroup": {
           "useSystemd": "yes|no"
       }
   }
}
```

The `config.json` file exists in `/greengrass/configuration/` and contains the following parameters:


****  

| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the [AWS IoT root CA](http://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem) relative to the `/greengrass/configuration/certs` folder\.  |  Save the file under the `/greengrass/configuration/certs` folder\.  | 
| certPath |  The path to the AWS Greengrass core certificate relative to the `/greengrass/configuration/certs` folder\.  | Save the file under the /greengrass/configuration/certs folder\. | 
| keyPath | The path to the AWS Greengrass core private key relative to the /greengrass/configuration/certs folder\. | Save the file under the /greengrass/configuration/certs folder\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS Greengrass core\.  | You can find it in the AWS Greengrass console under the definition for your AWS IoT hing\. | 
| iotHost | Your AWS IoT endpoint\. | You can find it in the AWS IoT console under Settings\. | 
| ggHost | Your AWS endpoint\. |  You can find it in the AWS IoT console under **Settings** with `greengrass.` prepended\.  | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default value is 600 seconds\. | 
| useSystemd | A binary flag if your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Values are yes or no\. Use the dependency script in [Module 1](module1.md) to see if your device uses systemd\. | 

------