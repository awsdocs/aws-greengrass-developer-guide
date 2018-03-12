# Troubleshooting AWS Greengrass Applications<a name="gg-troubleshooting"></a>

Use the following information to help troubleshoot issues in AWS Greengrass\.

## <a name="w3ab1c25b5"></a>


**Common Issues**  

| Symptom | Solution | 
| --- | --- | 
|  You see 403 Forbidden error on deployment in the logs\.  |  Make sure the AWS Greengrass core's policy in the cloud includes "greengrass:\*" as an allowed action\.   | 
|  Device's shadow does not sync with the cloud\.  |  Check that the AWS Greengrass core has permissions for "iot:UpdateThingShadow" and "iot:GetThingShadow" actions\. Also see [Troubleshooting Shadow Synchronization Timeout Issues](#troubleshooting-shadow-sync)\.  | 
| The AWS Greengrass core software does not run on Raspberry Pi because user namespace is not enabled\. | Run `rpi-update` to update\. Raspbian has released a new kernel 4\.9 that has user namespace enabled\. | 
| A `ConcurrentDeployment` error occurs when you run `create-deployment` for the first time\. | A deployment might be in progress\. You can run `get-deployment-history` to see if a deployment was created\. If not, try creating the deployment again\.  | 
| The AWS Greengrass core software does not start successfully\. | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-troubleshooting.html) | 
| The greengrassd script displays: `unable to accept TCP connection. accept tcp [::]:8000: accept4: too many open files`\. | The file descriptor limit for the AWS Greengrass core software has reached the threshold and must be increased\. Use the following command: <pre>ulimit -n 2048</pre> and restart the AWS Greengrass core software\.  In this example, the limit is increased to 2048\. Choose a value appropriate for your use case\.  | 
| Deployment fails with the following error message: Greengrass is not authorized to assume the Service Role associated with this account\. | Using the AWS CLI, check that an appropriate service role has been tied to your account using [AssociateServiceRoleToAccount](http://docs.aws.amazon.com/greengrass/latest/apireference/associateserviceroletoaccount-put.html) and that the account has at least the AWSGreengrassResourceAccessRolePolicy permission\. | 
| Deployment remains in the **In progress** state\. | Make sure that the AWS Greengrass daemon is running on your core device\. Run the following commands in your core device shell to check whether the daemon is running and to start it, if needed\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-troubleshooting.html)  | 

## Troubleshooting with Logs<a name="troubleshooting-logs"></a>

------
#### [ GGC v1\.3\.0 ]

If logs are configured to be stored on the local file system, start looking in the following locations:

`GREENGRASS_ROOT/ggc/var/log/crash.log`  
Shows messages generated when an AWS Greengrass core crashes\.

`GREENGRASS_ROOT/ggc/var/log/system/runtime.log`  
Shows messages about which component failed\.

`GREENGRASS_ROOT/ggc/var/log/system/`  
This folder contains all the logs from the AWS Greengrass system Lambda functions\. Using the messages in `ggc/var/log/system/` and `ggc/var/log/system/runtime.log`, you should be able to find out which error occurred in AWS Greengrass system Lambda functions\.

**Note**  
By default, *GREENGRASS\_ROOT* is the `/greengrass` directory\.

If the logs are configured to be stored on the cloud, use CloudWatch Logs to view log messages\. The crash log is still found on the AWS Greengrass core device\.

If connection errors occur when system Lambda function attempt to connect to AWS IoT, look in the AWS IoT logs in CloudWatch for more information\.

**Note**  
AWS IoT must be configured to write logs to CloudWatch\.

------
#### [ GGC v1\.1\.0 ]

If logs are configured to be stored on the local file system, start looking in the following locations:

`GREENGRASS_ROOT/ggc/var/log/crash.log`  
Shows messages generated when an AWS Greengrass core crashes\.

`GREENGRASS_ROOT/ggc/var/log/system/runtime.log`  
Shows messages about which component failed\.

`GREENGRASS_ROOT/ggc/var/log/system/`  
This folder contains all the logs from the AWS Greengrass system Lambda functions\. Using the messages in `ggc/var/log/system/` and `ggc/var/log/system/runtime.log`, you should be able to find out which error occurred in AWS Greengrass system Lambda functions\.

**Note**  
By default, *GREENGRASS\_ROOT* is the `/greengrass` directory\.

If the logs are configured to be stored on the cloud, use CloudWatch Logs to view log messages\. The crash log is still found on the AWS Greengrass core device\.

If connection errors occur when system Lambda function attempt to connect to AWS IoT, look in the AWS IoT logs in CloudWatch for more information\.

**Note**  
AWS IoT must be configured to write logs to CloudWatch\.

------
#### [ GGC v1\.0\.0 ]

If logs are configured to be stored on the local file system, start looking in the following locations:

`GREENGRASS_ROOT/crash.log`  
Shows messages generated when an AWS Greengrass core crashes\.

`GREENGRASS_ROOT/var/log/system/runtime.log`  
Shows messages about which component failed\.

`GREENGRASS_ROOT/var/log/system/`  
This folder contains all the logs from the AWS Greengrass system Lambda functions\. Using the messages in `var/log/system/` and `var/log/system/runtime.log`, you should be able to find out which error occurred in AWS Greengrass system Lambda functions\.

**Note**  
By default, *GREENGRASS\_ROOT* is the `/greengrass` directory\.

If the logs are configured to be stored on the cloud, use CloudWatch Logs to view log messages\. The crash log is still found on the AWS Greengrass core device\.

If connection errors occur when system Lambda function attempt to connect to AWS IoT, look in the AWS IoT logs in CloudWatch for more information\.

**Note**  
AWS IoT must be configured to write logs to CloudWatch\.

------

## Troubleshooting Storage Issues<a name="troubleshooting-storage"></a>

When the local file storage is full, some components might start failing: 

+ Local shadow updates do not happen\.

+ New AWS Greengrass core MQTT server certificates cannot be downloaded locally\.

+ Deployments fail\.

You should always be aware of the amount of free space available locally\. This can be calculated based on the sizes of deployed Lambda functions, the logging configuration \(see [Troubleshooting with Logs](#troubleshooting-logs)\), and the number of shadows stored locally\. 

## Troubleshooting Messages<a name="troubleshooting-messages"></a>

All messages sent within AWS Greengrass are sent with QoS 0\. If an AWS Greengrass core is restarted, messages that have not been processed yet are lost\. For this reason, restart the AWS Greengrass core when the service disruption is the lowest\. The AWS Greengrass core is restarted by a deployment, too\.

## Troubleshooting Shadow Synchronization Timeout Issues<a name="troubleshooting-shadow-sync"></a>

------
#### [ GGC v1\.3\.0 ]

If there is a significant delay in communication between a Greengrass core device and the cloud, then shadow synchronization may fail due to a timeout\. You may see something like this in your log files:

```
[2017-07-20T10:01:58.006Z][ERROR]-cloud_shadow_client.go:57,Cloud shadow client error: unable to get cloud shadow what_the_thing_is_named for synchronization. Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][WARN]-sync_manager.go:263,Failed to get cloud copy: Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][ERROR]-sync_manager.go:375,Failed to execute sync operation {what_the_thing_is_named VersionDiscontinued []}"
```

A possible fix is to configure the amount of time your Greengrass core device waits for a host response\. Open the `GREENGRASS_ROOT/config/config.json` file and add a system\.shadowSyncTimeout field with a timeout value in seconds\. For example:

```
{
  "coreThing": {
    "caPath": "root-ca.pem",
    "certPath": "cloud.pem.crt",
    "keyPath": "cloud.pem.key",
    "thingArn": "arn:aws:iot:us-west-2:049039099382:thing/GGTestGroup42_Core",
    "iotHost": "your-AWS-IoT-endpoint", 
    "ggHost": "greengrass.iot.us-west-2.amazonaws.com",
    "keepAlive": 600
  },
  "runtime": {
    "cgroup": {
      "useSystemd": "yes"
    }
  },
  "system": {
    "shadowSyncTimeout": 10
  }
}
```

If no shadowSyncTimeout value is specified in the config\.json file, the default is 1 second\. 

------
#### [ GGC v1\.1\.0 ]

If there is a significant delay in communication between a Greengrass core device and the cloud, then shadow synchronization may fail due to a timeout\. You may see something like this in your log files:

```
[2017-07-20T10:01:58.006Z][ERROR]-cloud_shadow_client.go:57,Cloud shadow client error: unable to get cloud shadow what_the_thing_is_named for synchronization. Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][WARN]-sync_manager.go:263,Failed to get cloud copy: Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][ERROR]-sync_manager.go:375,Failed to execute sync operation {what_the_thing_is_named VersionDiscontinued []}"
```

A possible fix is to configure the amount of time your Greengrass core device waits for a host response\. Open the `GREENGRASS_ROOT/config/config.json` file and add a system\.shadowSyncTimeout field with a timeout value in seconds\. For example:

```
{
  "coreThing": {
    "caPath": "root-ca.pem",
    "certPath": "cloud.pem.crt",
    "keyPath": "cloud.pem.key",
    "thingArn": "arn:aws:iot:us-west-2:049039099382:thing/GGTestGroup42_Core",
    "iotHost": "your-AWS-IoT-endpoint", 
    "ggHost": "greengrass.iot.us-west-2.amazonaws.com",
    "keepAlive": 600
  },
  "runtime": {
    "cgroup": {
      "useSystemd": "yes"
    }
  },
  "system": {
    "shadowSyncTimeout": 10
  }
}
```

If no shadowSyncTimeout value is specified in the config\.json file, the default is 1 second\. 

------
#### [ GGC v1\.0\.0 ]

Not supported\.

------