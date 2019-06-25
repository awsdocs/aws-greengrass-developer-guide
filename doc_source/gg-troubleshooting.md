# Troubleshooting AWS IoT Greengrass<a name="gg-troubleshooting"></a>

Use the following information to help troubleshoot issues in AWS IoT Greengrass\.

## Troubleshooting Issues<a name="gg-troubleshooting-issues"></a>


**AWS IoT Greengrass Core Issues**  

| Symptom | Solution | 
| --- | --- | 
|  Device's shadow does not sync with the cloud\.  |  Make sure that the AWS IoT Greengrass core has permissions for `iot:UpdateThingShadow` and `iot:GetThingShadow` actions in the [ Greengrass service role](service-role.md)\. If the service role uses the `AWSGreengrassResourceAccessRolePolicy` managed policy, these permissions are included by default\. See [Troubleshooting Shadow Synchronization Timeout Issues](#troubleshooting-shadow-sync)\.  | 
| The AWS IoT Greengrass Core software does not run on Raspberry Pi because user namespace is not enabled\.  User namespace support is required to run AWS IoT Greengrass in the default `Greengrass container` mode\. It isn't required to run in `No container` mode\. For more information, see [Considerations When Choosing Lambda Function Containerization](lambda-group-config.md#lambda-containerization-considerations)\.  | Run `BRANCH=stable rpi-update` to update to the latest stable versions of the firmware and kernel\. For Raspbian distributions, this instruction applies for Jessie, but not Stretch\. User namespaces are enabled in Stretch by default\.  | 
| The AWS IoT Greengrass Core software does not start successfully\. | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-troubleshooting.html) | 
| The AWS IoT Greengrass Core software does not start successfully\. You're running v1\.6 or earlier and you receive the following error in `crash.log`: `The configuration file is missing the CaPath, CertPath or KeyPath. The Greengrass daemon process with [pid = pid] died`\. |  Do one of the following: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-troubleshooting.html) | 
| The AWS IoT Greengrass Core software does not start, and you receive the following error: `Failed to parse /greengrass-root/config/config.json`\. | Make sure the [Greengrass configuration file](gg-core.md#config-json) is using valid JSON format\. Open `config.json` \(located in `/greengrass-root/config`\) and validate the JSON format\. For example, make sure that commas are used correctly\.  | 
| The AWS IoT Greengrass Core software does not start, and you receive the following error: `[ERROR]-Runtime failed to start: unable to start workers: container test timed out`\. | Set the `postStartHealthCheckTimeout` property in the [Greengrass configuration file](gg-core.md#config-json)\. This optional property configures the amount of time \(in milliseconds\) that the Greengrass daemon waits for the post\-start health check to finish\. The default value is 30 seconds \(30000 ms\)\. Open `config.json` \(located in `/greengrass-root/config`\)\. In the `runtime` object, add the `postStartHealthCheckTimeout` property and set the value to a number greater than 30000\. Add a comma where needed to create a valid JSON document\. For example: <pre>  ...<br />  "runtime" : {<br />    "cgroup" : {<br />      "useSystemd" : "yes"<br />    },<br />    "postStartHealthCheckTimeout" : 40000<br />  },<br />  ...</pre>  | 
| TheAWS IoT Greengrass Core software does not start on a Raspberry Pi, and you receive the following error: `Failed to invoke PutLogEvents on local Cloudwatch, logGroup: /GreengrassSystem/connection_manager, error: RequestError: send request failed caused by: Post http://path/cloudwatch/logs/: dial tcp address: getsockopt: connection refused, response: { }`\. | This can occur if the OS version is Raspbian Stretch and the required memory setup has not been completed\. If you are using Stretch, you need to perform more memory setup\. For more information, see [this step](setup-filter.rpi.md#stretch-step)\.  | 
| The AWS IoT Greengrass Core software does not start, and you receive the following error: `Unable to create server due to: failed to load group: chmod /greengrass-root/ggc/deployment/lambda/arn:aws:lambda:region:account-id:function:function-name:version/file-name: no such file or directory` | If you deployed a [Lambda executable](lambda-functions.md#lambda-executables) to the core, check the function's `Handler` property in the `group.json` file \(located in /*greengrass\-root*/ggc/deployment/group\)\. If the handler is not the exact name of your compiled executable, replace the contents of the `group.json` file with an empty JSON object \(`{}`\), and run the following commands to start AWS IoT Greengrass: <pre>cd /greengrass/ggc/core/<br />sudo ./greengrassd start</pre> Then, use the [AWS Lambda API](https://docs.aws.amazon.com/cli/latest/reference/lambda/) to update the function configuration's `handler` parameter, publish a new function version, and update the alias\. For more information, see [AWS Lambda Function Versioning and Aliases](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\. Assuming that you added the function to your Greengrass group by alias \(recommended\), you can now redeploy your group\. \(If not, you must point to the new function version or alias in your group definition and subscriptions before you deploy the group\.\) | 
| The AWS IoT Greengrass Core software does not start, and you receive the following error: `Spool size should be at least 262144 bytes`\. | Open the `group.json` file \(located in `/greengrass-root/ggc/deployment/group`\), replace the contents of the file with an empty JSON object \(`{}`\), and run the following commands to start AWS IoT Greengrass: <pre>cd /greengrass/ggc/core/<br />sudo ./greengrassd start</pre> Then follow the steps in the [To Cache Messages in Local Storage](gg-core.md#configure-local-storage-cache) procedure\. For the `GGCloudSpooler` function, make sure to specify a `GG_CONFIG_MAX_SIZE_BYTES` value that's greater than or equal to 262144\. | 
|  The AWS IoT Greengrass Core software does not start, and you receive the following error in `runtime.log`: `container_linux.go:344: starting container process caused "process_linux.go:424: container init caused \"rootfs_linux.go:64: mounting \\\"/greengrass/ggc/socket/greengrass_ipc.sock\\\" to rootfs \\\"/greengrass/ggc/packages/version/rootfs/merged\\\" at \\\"/greengrass_ipc.sock\\\" caused \\\"stat /greengrass/ggc/socket/greengrass_ipc.sock: permission denied\\\"\""` | This occurs if your `umask` is higher than `0022`\. To resolve this issue, you must set the `umask` to `0022` or lower\. A value of `0022` grants everyone read permission to new files by default\. | 
| The AWS IoT Greengrass Core software does not start, and you receive the following error: `Greengrass daemon running with PID: [processid]. Some system components failed to start. Check 'runtime.log' for errors` | Check runtime\.log and crash\.log for error messages\. For more information, see [Troubleshooting with Logs](#troubleshooting-logs)\.  | 
| The `greengrassd` script displays: `unable to accept TCP connection. accept tcp [::]:8000: accept4: too many open files`\. | The file descriptor limit for the AWS IoT Greengrass Core software has reached the threshold and must be increased\. Use the following command and then restart the AWS IoT Greengrass Core software\. <pre>ulimit -n 2048</pre>  In this example, the limit is increased to 2048\. Choose a value appropriate for your use case\.  | 
| You receive the following error: `Runtime execution error: unable to start lambda container. container_linux.go:259: starting container process caused "process_linux.go:345: container init caused \"rootfs_linux.go:50: preparing rootfs caused \\\"permission denied\\\"\""` | Either install AWS IoT Greengrass directly under the root directory, or make sure that the `/greengrass` directory and its parent directories have `execute` permissions for everyone\. | 
| You receive the following warning in `runtime.log`: `[WARN]-[5]GK Remote: Error retrieving public key data: ErrPrincipalNotConfigured: private key for MqttCertificate is not set` | AWS IoT Greengrass uses a common handler to validate the properties of all security principals\. This warning is expected unless you specified a custom private key for the local MQTT server\. For more information, see [AWS IoT Greengrass Core Security Principals](gg-sec.md#gg-principals)\. | 
| An over\-the\-air \(OTA\) update fails with the following error: `Permission denied when attempting to use role arn:aws:iam::account-id:role/role-name to access s3 url https://region-greengrass-updates.s3.region.amazonaws.com/core/architecture/greengrass-core-distribution-version.tar.gz` | In the signer role policy, add the target AWS Region as a `Resource`\. Thes signer role is used to presign the S3 URL for the AWS IoT Greengrass software update\. For more information, see [S3 URL signer role](core-ota-update.md#s3-url-signer-role)\. | 
| The core is in an infinite connect\-disconnect loop\. The `runtime.log` file contains a continuous series of connect and disconnect entries\. | This can happen when another device is hard\-coded to use the core thing name as the client ID for MQTT connections to AWS IoT\. Simultaneous connections in the same AWS Region and AWS account must use unique client IDs\. By default, the core uses the core thing name as the client ID for these connections\. To resolve this issue, you can change the client ID used by the other device for the connection \(recommended\) or override the default value for the core\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-troubleshooting.html) | 


**Deployment Issues**  

| Symptom | Solution | 
| --- | --- | 
|  You see 403 Forbidden error on deployment in the logs\.  |  Make sure the policy of the AWS IoT Greengrass core in the cloud includes `"greengrass:*"` as an allowed action\.   | 
| A `ConcurrentDeployment` error occurs when you run create\-deployment for the first time\. | A deployment might be in progress\. You can run get\-deployment\-history to see if a deployment was created\. If not, try creating the deployment again\.  | 
| The deployment fails with the following error message: Greengrass is not authorized to assume the Service Role associated with this account or Failed: TES service role is not associated with this account\. | Using the [https://docs.aws.amazon.com/greengrass/latest/apireference/getserviceroleforaccount-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getserviceroleforaccount-get.html) command in the AWS CLI, check that an appropriate service role is associated with your AWS account in the current AWS Region\. To associate a Greengrass service role with your AWS account in the current AWS Region, use [https://docs.aws.amazon.com/greengrass/latest/apireference/associateserviceroletoaccount-put.html](https://docs.aws.amazon.com/greengrass/latest/apireference/associateserviceroletoaccount-put.html)\. For more information, see [Greengrass Service Role](service-role.md)\. | 
| The deployment doesn't finish\. | Make sure that the AWS IoT Greengrass daemon is running on your core device\. Run the following commands in your core device terminal to check whether the daemon is running and start it, if needed\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-troubleshooting.html)  | 
| The deployment doesn't finish, and `runtime.log` contains multiple `wait 1s for container to stop` entries\. | Run the following commands in your core device terminal to restart the AWS IoT Greengrass daemon\. <pre>cd /greengrass/ggc/core/<br />sudo ./greengrassd stop<br />sudo ./greengrassd start</pre>  | 
| You receive the following error: `Deployment guid of type NewDeployment for group guid failed error: Error while processing. group config is invalid: 112 or [119 0] don't have rw permission on the file: path` | Make sure that the owner group of the *path* directory has read and write permissions to the directory\. | 
| The deployment fails with the following error in runtime\.log: `[list of function arns] are configured to run as root but Greengrass is not configured to run Lambda functions with root permissions`\. Either change the value of `allowFunctionsToRunAsRoot` in `greengrass_root/config/config.json` to `yes` or change the Lambda function to run as another user/group\. | Make sure that you have configured AWS IoT Greengrass to allow Lambda functions to run with root permissions\. For more information, see [Running a Lambda Function as Root](lambda-group-config.md#lambda-running-as-root)\. | 
| The deployment fails with the following error message, and you are running AWS IoT Greengrass Core software v1\.7: `Deployment deployment id of type NewDeployment for group group id failed error: process start failed: container_linux.go:259: starting container process caused "process_linux.go:250: running exec setns process for init caused \"wait: no child processes\""` | Retry the deployment\. | 


**Create Group/Create Function Issues**  

| Symptom | Solution | 
| --- | --- | 
| You receive the error: `Your 'IsolationMode' configuration for the group is invalid`\. | This error occurs when the `"IsolationMode"` value in the `"DefaultConfig"` of `function-definition-version` is not supported\. Supported values are `"GreengrassContainer"` and `"NoContainer"`\. | 
| You receive the error: `Your 'IsolationMode' configuration for function with arn function ARN is invalid`\. | This error occurs when the `"IsolationMode"` value in the *function ARN* of the `function-definition-version` is not supported\. Supported values are `"GreengrassContainer"` and `"NoContainer"`\. | 
| You receive the error: `MemorySize configuration for function with arn function ARN is not allowed in IsolationMode=NoContainer`\. | This error occurs when you specify a `MemorySize` value and you choose to run without containerization\. Lambda functions that are run without containerization cannot have memory limits\. You can either remove the limit or you can change the Lambda function to run in an AWS IoT Greengrass container\. | 
| You receive the error: `Access Sysfs configuration for function with arn function ARN is not allowed in IsolationMode=NoContainer`\. | This error occurs when you specify `true` for `AccessSysfs` and you choose to run without containerization\. Lambda functions run without containerization must have their code updated to access the file system directly and cannot use `AccessSysfs`\. You can either specify a value of `false` for `AccessSysfs` or you can change the Lambda function to run in an AWS IoT Greengrass container\. | 
| You receive the error: `MemorySize configuration for function with arn function ARN is required in IsolationMode=GreengrassContainer` | This error occurs because you did not specify a `MemorySize` limit for a Lambda function that you are running in an AWS IoT Greengrass container\. Specify a `MemorySize` value to resolve the error\. | 
| You receive the error: `Function function ARN refers to resource of type resourceType that is not allowed in IsolationMode=NoContainer`\. | You cannot access `Local.Device`, `Local.Volume`, `ML_Model.SageMaker.Job`, `ML_Model.S3_Object`, or `S3_Object.Generic_Archive` resource types when you run a Lambda function without containerization\. If you need those resource types, you must run in an AWS IoT Greengrass container\. You can also access local devices directly when running without containerization by changing the code in your Lambda function\. | 
| You receive the error: `Execution configuration for function with arn function ARN is not allowed`\. | This error occurs when you create a system Lambda function with `GGIPDetector` or `GGCloudSpooler` and you specified `IsolationMode` or `RunAs` configuration\. You must omit the `Execution` parameters for this system Lambda function\. | 

**AWS IoT Greengrass Core in Docker Issues**


| Symptom | Solution | 
| --- | --- | 
| <a name="docker-troubleshooting-cli-version"></a> You receive the error: `Unknown options: -no-include-email` when running the `aws ecr get-login` command\.  |  Make sure that you have the latest AWS CLI version installed \(for example, run: `pip install awscli --upgrade --user`\)\. If you're using Windows and you installed the CLI using the MSI installer, you must repeat the installation process\. For more information, see [Installing the AWS Command Line Interface on Microsoft Windows](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html) in the *AWS Command Line Interface User Guide*\.  | 
| <a name="docker-troubleshooting-ipv4-disabled"></a> You receive an error such as: `WARNING: IPv4 is disabled. Networking will not work` on a Linux computer\.  |  Enable IPv4 network forwarding as described in this [step](run-gg-in-docker-container.md#docker-linux-enable-ipv4)\. AWS IoT Greengrass cloud deployment and MQTT communications don't work when IPv4 forwarding isn't enabled\. For more information, see [Configure namespaced kernel parameters \(sysctls\) at runtime](https://docs.docker.com/engine/reference/commandline/run/#configure-namespaced-kernel-parameters-sysctls-at-runtime) in the Docker documentation\.  | 
| <a name="docker-troubleshooting-firewall"></a> You receive the message: `Firewall Detected while Sharing Drives` when running Docker on a Windows computer\.  |  See the [Error: A firewall is blocking file sharing between Windows and the containers](https://success.docker.com/article/error-a-firewall-is-blocking-file-sharing-between-windows-and-the-containers) Docker support issue\. This error can also occur if you are signed in on a virtual private network \(VPN\) and your network settings are preventing the shared drive from being mounted\. In that situation, turn off VPN and re\-run the Docker container\.  | 
|  You receive the error: `Cannot create container for the service greengrass: Conflict. The container name "/aws-iot-greengrass" is already in use.`  |  This can occur when the container name is used by an older run\. To resolve this issue, run the following command to remove the old Docker container: <pre>docker rm -f $(docker ps -a -q -f "name=aws-iot-greengrass")</pre>  | 
|  You receive the following error in `runtime.log`: `[FATAL]-Failed to reset thread's mount namespace due to an unexpected error: "operation not permitted". To maintain consistency, GGC will crash and need to be manually restarted`\.  |  This can occur when you try to deploy a `GreengrassContainer` Lambda function to an AWS IoT Greengrass core running in a Docker container\. Currently, only `NoContainer` Lambda functions can be deployed to a Greengrass Docker container\. To resolve this issue, [make sure that all Lambda functions are in `NoContainer` mode](lambda-group-config.md#change-containerization-lambda) and start a new deployment\. Then, when starting the container, don't bind\-mount the existing `deployment` directory onto the AWS IoT Greengrass core Docker container\. Instead, create an empty `deployment` directory in its place and bind\-mount that in the Docker container\. This allows the new Docker container to receive the latest deployment with Lambda functions running in `NoContainer` mode\.   | 

For more information, see [Running AWS IoT Greengrass in a Docker Container](run-gg-in-docker-container.md)\.

## Troubleshooting with Logs<a name="troubleshooting-logs"></a>

If logs are configured to be stored on the local file system, start looking in the following locations\. Reading the logs on the file system requires root permissions\.

`greengrass-root/ggc/var/log/crash.log`  
Shows messages generated when an AWS IoT Greengrass core crashes\.

`greengrass-root/ggc/var/log/system/runtime.log`  
Shows messages about which component failed\.

`greengrass-root/ggc/var/log/system/`  
Contains all logs from AWS IoT Greengrass system components, such as the certificate manager and the connection manager\. By using the messages in `ggc/var/log/system/` and `ggc/var/log/system/runtime.log`, you should be able to find out which error occurred in AWS IoT Greengrass system components\.

`greengrass-root/ggc/var/log/user/`  
Contains all logs from user\-defined Lambda functions\. Check this folder to find error messages from your local Lambda functions\.

**Note**  
By default, *greengrass\-root* is the `/greengrass` directory\. If a [write directory](gg-core.md#write-directory) is configured, then the logs are under that directory\.

If the logs are configured to be stored on the cloud, use CloudWatch Logs to view log messages\. `crash.log` is found only in file system logs on the AWS IoT Greengrass core device\.

If AWS IoT is configured to write logs to CloudWatch, check those logs if connection errors occur when system components attempt to connect to AWS IoT\.

For more information about AWS IoT Greengrass logging, see [Monitoring with AWS IoT Greengrass Logs](greengrass-logs-overview.md)\.

**Note**  
Logs for AWS IoT Greengrass Core software v1\.0 are stored under the `greengrass-root/var/log` directory\.

## Troubleshooting Storage Issues<a name="troubleshooting-storage"></a>

When the local file storage is full, some components might start failing: 
+ Local shadow updates do not occur\.
+ New AWS IoT Greengrass core MQTT server certificates cannot be downloaded locally\.
+ Deployments fail\.

You should always be aware of the amount of free space available locally\. You can calculate free space based on the sizes of deployed Lambda functions, the logging configuration \(see [Troubleshooting with Logs](#troubleshooting-logs)\), and the number of shadows stored locally\. 

## Troubleshooting Messages<a name="troubleshooting-messages"></a>

All messages sent in AWS IoT Greengrass are sent with QoS 0\. By default, AWS IoT Greengrass stores messages in an in\-memory queue\. Therefore, unprocessed messages are lost when the AWS IoT Greengrass core restarts \(for example, after a group deployment or device reboot\)\. However, you can configure AWS IoT Greengrass \(v1\.6 or later\) to cache messages to the file system so they persist across core restarts\. You can also configure the queue size\. For more information, see [MQTT Message Queue](gg-core.md#mqtt-message-queue)\.

**Note**  
When using the default in\-memory queue, we recommend that you deploy groups or restart the device when the service disruption is the lowest\.

If you configure a queue size, make sure that it's greater than or equal to 262144 bytes \(256 KB\)\. Otherwise, AWS IoT Greengrass might not start properly\.

## Troubleshooting Shadow Synchronization Timeout Issues<a name="troubleshooting-shadow-sync"></a>

Significant delays in communication between a Greengrass core device and the cloud might cause shadow synchronization to fail because of a timeout\. In this case, you should see log entries similar to the following:

```
[2017-07-20T10:01:58.006Z][ERROR]-cloud_shadow_client.go:57,Cloud shadow client error: unable to get cloud shadow what_the_thing_is_named for synchronization. Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][WARN]-sync_manager.go:263,Failed to get cloud copy: Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][ERROR]-sync_manager.go:375,Failed to execute sync operation {what_the_thing_is_named VersionDiscontinued []}"
```

A possible fix is to configure the amount of time that the core device waits for a host response\. Open the [`config.json`](gg-core.md#config-json) file in `greengrass-root/config` and add a `system.shadowSyncTimeout` field with a timeout value in seconds\. For example:

```
{
  "system": {
    "shadowSyncTimeout": 10
  },
  "coreThing": {
    "caPath": "root-ca.pem",
    "certPath": "cloud.pem.crt",
    "keyPath": "cloud.pem.key",
    ...
  },
  ...
}
```

If no `shadowSyncTimeout` value is specified in `config.json`, the default is 5 seconds\.

**Note**  
For AWS IoT Greengrass Core software v1\.6 and earlier, the default `shadowSyncTimeout` is 1 second\.

## Check the AWS IoT Greengrass Forum<a name="troubleshooting-forum"></a>

If you're unable to resolve your issue using the troubleshooting information in this topic, you can search the [AWS IoT Greengrass Forum](https://forums.aws.amazon.com/forum.jspa?forumID=254) for related issues or post a new forum thread\. Members of the AWS IoT Greengrass team actively monitor the forum\.