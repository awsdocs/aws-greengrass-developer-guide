# Troubleshooting AWS IoT Greengrass<a name="gg-troubleshooting"></a>

This section provides troubleshooting information and possible solutions to help resolve issues with AWS IoT Greengrass\.

<a name="gg-limits-genref"></a>For information about AWS IoT Greengrass quotas \(limits\), see [Service Quotas](https://docs.aws.amazon.com/general/latest/gr/greengrass.html#limits_greengrass) in the *Amazon Web Services General Reference*\.

## AWS IoT Greengrass Core issues<a name="gg-troubleshooting-coreissues"></a>

If the AWS IoT Greengrass Core software does not start, try the following general troublehooting steps:
+ Make sure that you install the binaries that are appropriate for your architecture\. For more information, see [AWS IoT Greengrass Core Software](what-is-gg.md#gg-core-download-tab)\.
+ Make sure that your core device has local storage available\. For more information, see [Troubleshooting storage issues](#troubleshooting-storage)\.
+ Check `runtime.log` and `crash.log` for error messages\. For more information, see [Troubleshooting with logs](#troubleshooting-logs)\.

Search the following symptoms and errors to find information to help troubleshoot issues with an AWS IoT Greengrass core\.

**Topics**
+ [Error: The configuration file is missing the CaPath, CertPath or KeyPath\. The Greengrass daemon process with \[pid = <pid>\] died\.](#troubleshoot-config-file-version)
+ [Error: Failed to parse /<greengrass\-root>/config/config\.json\.](#troubleshoot-config-file-invalid)
+ [Error: Error occurred while generating TLS config: ErrUnknownURIScheme](#troubleshooting-unknown-uri-scheme)
+ [Error: Runtime failed to start: unable to start workers: container test timed out\.](#troubleshoot-post-start-health-check)
+ [Error: Failed to invoke PutLogEvents on local Cloudwatch, logGroup: /GreengrassSystem/connection\_manager, error: RequestError: send request failed caused by: Post http://<path>/cloudwatch/logs/: dial tcp <address>: getsockopt: connection refused, response: \{ \}\.](#troubleshoot-cloudwatch-logs)
+ [Error: Unable to create server due to: failed to load group: chmod /<greengrass\-root>/ggc/deployment/lambda/arn:aws:lambda:<region>:<account\-id>:function:<function\-name>:<version>/<file\-name>: no such file or directory\.](#troubleshoot-lambda-executable-handler)
+ [The AWS IoT Greengrass Core software doesn't start after you changed from running with no containerization to running in a Greengrass container\.](#troubleshoot-no-container)
+ [Error: Spool size should be at least 262144 bytes\.](#troubleshoot-spool-size)
+ [Error: \[ERROR\]\-Cloud messaging error: Error occurred while trying to publish a message\. \{"errorString": "operation timed out"\}](#troubleshoot-mqtt-operation-timed-out)
+ [Error: container\_linux\.go:344: starting container process caused "process\_linux\.go:424: container init caused \\"rootfs\_linux\.go:64: mounting \\\\\\"/greengrass/ggc/socket/greengrass\_ipc\.sock\\\\\\" to rootfs \\\\\\"/greengrass/ggc/packages/<version>/rootfs/merged\\\\\\" at \\\\\\"/greengrass\_ipc\.sock\\\\\\" caused \\\\\\"stat /greengrass/ggc/socket/greengrass\_ipc\.sock: permission denied\\\\\\"\\""\.](#troubleshoot-umask-permission)
+ [Error: Greengrass daemon running with PID: <process\-id>\. Some system components failed to start\. Check 'runtime\.log' for errors\.](#troubleshoot-system-components)
+ [Device shadow does not sync with the cloud\.](#troubleshoot-shadow-sync)
+ [ERROR: unable to accept TCP connection\. accept tcp \[::\]:8000: accept4: too many open files\.](#troubleshoot-file-descriptor-limit)
+ [Error: Runtime execution error: unable to start lambda container\. container\_linux\.go:259: starting container process caused "process\_linux\.go:345: container init caused \\"rootfs\_linux\.go:50: preparing rootfs caused \\\\\\"permission denied\\\\\\"\\""\.](#troubleshoot-execute-permissions)
+ [Warning: \[WARN\]\-\[5\]GK Remote: Error retrieving public key data: ErrPrincipalNotConfigured: private key for MqttCertificate is not set\.](#troubleshoot-mqttcertificate-warning)
+ [Error: Permission denied when attempting to use role arn:aws:iam::<account\-id>:role/<role\-name> to access s3 url https://<region>\-greengrass\-updates\.s3\.<region>\.amazonaws\.com/core/<architecture>/greengrass\-core\-<distribution\-version>\.tar\.gz\.](#troubleshoot-ota-region-access)
+ [The AWS IoT Greengrass core is configured to use a [network proxy](gg-core.md#alpn-network-proxy) and your Lambda function can't make outgoing connections\.](#troubleshoot-lambda-proxy-network-connections)
+ [The core is in an infinite connect\-disconnect loop\. The runtime\.log file contains a continuous series of connect and disconnect entries\.](#config-client-id)
+ [Error: unable to start lambda container\. container\_linux\.go:259: starting container process caused "process\_linux\.go:345: container init caused \\"rootfs\_linux\.go:62: mounting \\\\\\"proc\\\\\\" to rootfs \\\\\\"](#troubleshoot-mount-proc-lambda-container)
+ [\[ERROR\]\-runtime execution error: unable to start lambda container\. \{"errorString": "failed to initialize container mounts: failed to mask greengrass root in overlay upper dir: failed to create mask device at directory <ggc\-path>: file exists"\}](#troubleshoot-usr-access-root)
+ [\[ERROR\]\-Deployment failed\. \{"deploymentId": "<deployment\-id>", "errorString": "container test process with pid <pid> failed: container process state: exit status 1"\}](#troubleshoot-usr-access-root-canary)
+ [Error: \[ERROR\]\-runtime execution error: unable to start lambda container\. \{"errorString": "failed to initialize container mounts: failed to create overlay fs for container: mounting overlay at /greengrass/ggc/packages/<ggc\-version>/rootfs/merged failed: failed to mount with args source=\\"no\_source\\" dest=\\"/greengrass/ggc/packages/<ggc\-version>/rootfs/merged\\" fstype=\\"overlay\\" flags=\\"0\\" data=\\"lowerdir=/greengrass/ggc/packages/<ggc\-version>/dns:/,upperdir=/greengrass/ggc/packages/<ggc\-version>/rootfs/upper,workdir=/greengrass/ggc/packages/<ggc\-version>/rootfs/work\\": too many levels of symbolic links"\}](#troubleshoot-symbolic-links)
+ [Error: \[DEBUG\]\-Failed to get routes\. Discarding message\.](#troubleshoot-failed-to-get-routes)
+ [Error: \[Errno 24\] Too many open <lambda\-function>,\[Errno 24\] Too many open files](#troubleshoot-too-many-open-files)

 

### Error: The configuration file is missing the CaPath, CertPath or KeyPath\. The Greengrass daemon process with \[pid = <pid>\] died\.<a name="troubleshoot-config-file-version"></a>

**Solution:** You might see this error in `crash.log` when the AWS IoT Greengrass Core software does not start\. This can occur if you're running v1\.6 or earlier\. Do one of the following:
+ Upgrade to v1\.7 or later\. We recommend that you always run the latest version of the AWS IoT Greengrass Core software\. For download information, see [AWS IoT Greengrass Core Software](what-is-gg.md#gg-core-download-tab)\.
+ Use the correct `config.json` format for your AWS IoT Greengrass Core software version\. For more information, see [AWS IoT Greengrass core configuration file](gg-core.md#config-json)\.
**Note**  
<a name="find-ggc-version-intro"></a>To find which version of the AWS IoT Greengrass Core software is installed on the core device, run the following commands in your device terminal\.  

  ```
  cd /greengrass-root/ggc/core/
  sudo ./greengrassd --version
  ```

 

### Error: Failed to parse /<greengrass\-root>/config/config\.json\.<a name="troubleshoot-config-file-invalid"></a>

**Solution:** You might see this error when the AWS IoT Greengrass Core software does not start\. Make sure the [Greengrass configuration file](gg-core.md#config-json) is using valid JSON format\.

Open `config.json` \(located in `/greengrass-root/config`\) and validate the JSON format\. For example, make sure that commas are used correctly\.

 

### Error: Error occurred while generating TLS config: ErrUnknownURIScheme<a name="troubleshooting-unknown-uri-scheme"></a>

**Solution:** You might see this error when the AWS IoT Greengrass Core software does not start\. Make sure the properties in the [crypto](gg-core.md#config-json-crypto) section of the Greengrass configuration file are valid\. The error message should provide more information\.

Open `config.json` \(located in `/greengrass-root/config`\) and check the `crypto` section\. For example, certificate and key paths must use the correct URI format and point to the correct location\.

 

### Error: Runtime failed to start: unable to start workers: container test timed out\.<a name="troubleshoot-post-start-health-check"></a>

**Solution:** You might see this error when the AWS IoT Greengrass Core software does not start\. Set the `postStartHealthCheckTimeout` property in the [Greengrass configuration file](gg-core.md#config-json)\. This optional property configures the amount of time \(in milliseconds\) that the Greengrass daemon waits for the post\-start health check to finish\. The default value is 30 seconds \(30000 ms\)\.

Open `config.json` \(located in `/greengrass-root/config`\)\. In the `runtime` object, add the `postStartHealthCheckTimeout` property and set the value to a number greater than 30000\. Add a comma where needed to create a valid JSON document\. For example:

```
  ...
  "runtime" : {
    "cgroup" : {
      "useSystemd" : "yes"
    },
    "postStartHealthCheckTimeout" : 40000
  },
  ...
```

 

### Error: Failed to invoke PutLogEvents on local Cloudwatch, logGroup: /GreengrassSystem/connection\_manager, error: RequestError: send request failed caused by: Post http://<path>/cloudwatch/logs/: dial tcp <address>: getsockopt: connection refused, response: \{ \}\.<a name="troubleshoot-cloudwatch-logs"></a>

**Solution:** You might see this error when the AWS IoT Greengrass Core software does not start\. This can occur if you're running AWS IoT Greengrass on a Raspberry Pi and the required memory setup has not been completed\. For more information, see [this step](setup-filter.rpi.md#stretch-step)\.

 

### Error: Unable to create server due to: failed to load group: chmod /<greengrass\-root>/ggc/deployment/lambda/arn:aws:lambda:<region>:<account\-id>:function:<function\-name>:<version>/<file\-name>: no such file or directory\.<a name="troubleshoot-lambda-executable-handler"></a>

**Solution:** You might see this error when the AWS IoT Greengrass Core software does not start\. If you deployed a [Lambda executable](lambda-functions.md#lambda-executables) to the core, check the function's `Handler` property in the `group.json` file \(located in /*greengrass\-root*/ggc/deployment/group\)\. If the handler is not the exact name of your compiled executable, replace the contents of the `group.json` file with an empty JSON object \(`{}`\), and run the following commands to start AWS IoT Greengrass:

```
cd /greengrass/ggc/core/
sudo ./greengrassd start
```

Then, use the [AWS Lambda API](https://docs.aws.amazon.com/cli/latest/reference/lambda/) to update the function configuration's `handler` parameter, publish a new function version, and update the alias\. For more information, see [AWS Lambda function versioning and aliases](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.

Assuming that you added the function to your Greengrass group by alias \(recommended\), you can now redeploy your group\. \(If not, you must point to the new function version or alias in your group definition and subscriptions before you deploy the group\.\)

 

### The AWS IoT Greengrass Core software doesn't start after you changed from running with no containerization to running in a Greengrass container\.<a name="troubleshoot-no-container"></a>

**Solution:** Check that you are not missing any container dependencies\.

 

### Error: Spool size should be at least 262144 bytes\.<a name="troubleshoot-spool-size"></a>

**Solution:** You might see this error when the AWS IoT Greengrass Core software does not start\. Open the `group.json` file \(located in `/greengrass-root/ggc/deployment/group`\), replace the contents of the file with an empty JSON object \(`{}`\), and run the following commands to start AWS IoT Greengrass:

```
cd /greengrass/ggc/core/
sudo ./greengrassd start
```

Then follow the steps in the [To cache messages in local storage](gg-core.md#configure-local-storage-cache) procedure\. For the `GGCloudSpooler` function, make sure to specify a `GG_CONFIG_MAX_SIZE_BYTES` value that's greater than or equal to 262144\.

 

### Error: \[ERROR\]\-Cloud messaging error: Error occurred while trying to publish a message\. \{"errorString": "operation timed out"\}<a name="troubleshoot-mqtt-operation-timed-out"></a>

**Solution:** You might see this error in `GGCloudSpooler.log` when the Greengrass core is unable to send MQTT messages to AWS IoT Core\. This can occur if the core environment has limited bandwidth and high latency\. If you're running AWS IoT Greengrass v1\.10\.2 or later, try increasing the `mqttOperationTimeout` value in the [config\.json](gg-core.md#config-json) file\. If the property is not present, add it to the `coreThing` object\. For example:

```
{
  "coreThing": {
    "mqttOperationTimeout": 10,
    "caPath": "root-ca.pem",
    "certPath": "hash.cert.pem",
    "keyPath": "hash.private.key",
    ...
  },
  ...
}
```

The default value is `5` and the minimum value is `5`\.

 

### Error: container\_linux\.go:344: starting container process caused "process\_linux\.go:424: container init caused \\"rootfs\_linux\.go:64: mounting \\\\\\"/greengrass/ggc/socket/greengrass\_ipc\.sock\\\\\\" to rootfs \\\\\\"/greengrass/ggc/packages/<version>/rootfs/merged\\\\\\" at \\\\\\"/greengrass\_ipc\.sock\\\\\\" caused \\\\\\"stat /greengrass/ggc/socket/greengrass\_ipc\.sock: permission denied\\\\\\"\\""\.<a name="troubleshoot-umask-permission"></a>

**Solution:** You might see this error in `runtime.log` when the AWS IoT Greengrass Core software does not start\. This occurs if your `umask` is higher than `0022`\. To resolve this issue, you must set the `umask` to `0022` or lower\. A value of `0022` grants everyone read permission to new files by default\.

 

### Error: Greengrass daemon running with PID: <process\-id>\. Some system components failed to start\. Check 'runtime\.log' for errors\.<a name="troubleshoot-system-components"></a>

**Solution:** You might see this error when the AWS IoT Greengrass Core software does not start\. Check `runtime.log` and `crash.log` for specific error information\. For more information, see [Troubleshooting with logs](#troubleshooting-logs)\.

 

### Device shadow does not sync with the cloud\.<a name="troubleshoot-shadow-sync"></a>

**Solution:** Make sure that AWS IoT Greengrass has permissions for `iot:UpdateThingShadow` and `iot:GetThingShadow` actions in the [Greengrass service role](service-role.md)\. If the service role uses the `AWSGreengrassResourceAccessRolePolicy` managed policy, these permissions are included by default\.

See [Troubleshooting shadow synchronization timeout issues](#troubleshooting-shadow-sync)\.

 

### ERROR: unable to accept TCP connection\. accept tcp \[::\]:8000: accept4: too many open files\.<a name="troubleshoot-file-descriptor-limit"></a>

**Solution:** You might see this error in the `greengrassd` script output\. This can occur if the file descriptor limit for the AWS IoT Greengrass Core software has reached the threshold and must be increased\.

Use the following command and then restart the AWS IoT Greengrass Core software\.

```
ulimit -n 2048
```

**Note**  
In this example, the limit is increased to 2048\. Choose a value appropriate for your use case\.

 

### Error: Runtime execution error: unable to start lambda container\. container\_linux\.go:259: starting container process caused "process\_linux\.go:345: container init caused \\"rootfs\_linux\.go:50: preparing rootfs caused \\\\\\"permission denied\\\\\\"\\""\.<a name="troubleshoot-execute-permissions"></a>

**Solution:** Either install AWS IoT Greengrass directly under the root directory, or make sure that the directory where the AWS IoT Greengrass Core software is installed and its parent directories have `execute` permissions for everyone\.

 

### Warning: \[WARN\]\-\[5\]GK Remote: Error retrieving public key data: ErrPrincipalNotConfigured: private key for MqttCertificate is not set\.<a name="troubleshoot-mqttcertificate-warning"></a>

**Solution:** AWS IoT Greengrass uses a common handler to validate the properties of all security principals\. This warning in `runtime.log` is expected unless you specified a custom private key for the local MQTT server\. For more information, see [AWS IoT Greengrass core security principals](gg-sec.md#gg-principals)\.

 

### Error: Permission denied when attempting to use role arn:aws:iam::<account\-id>:role/<role\-name> to access s3 url https://<region>\-greengrass\-updates\.s3\.<region>\.amazonaws\.com/core/<architecture>/greengrass\-core\-<distribution\-version>\.tar\.gz\.<a name="troubleshoot-ota-region-access"></a>

**Solution:** You might see this error when an over\-the\-air \(OTA\) update fails\. In the signer role policy, add the target AWS Region as a `Resource`\. This signer role is used to presign the S3 URL for the AWS IoT Greengrass software update\. For more information, see [S3 URL signer role](core-ota-update.md#s3-url-signer-role)\.

 

### The AWS IoT Greengrass core is configured to use a [network proxy](gg-core.md#alpn-network-proxy) and your Lambda function can't make outgoing connections\.<a name="troubleshoot-lambda-proxy-network-connections"></a>

**Solution:** Depending on your runtime and the executables used by the Lambda function to create connections, you might also receive connection timeout errors\. Make sure your Lambda functions use the appropriate proxy configuration to connect through the network proxy\. AWS IoT Greengrass passes the proxy configuration to user\-defined Lambda functions through the `http_proxy`, `https_proxy`, and `no_proxy` environment variables\. They can be accessed as shown in the following Python snippet\.

```
import os
print(os.environ['http_proxy'])
```

Use the same case as the variable defined in your environment, for example, all lower case `http_proxy` or all upper case `HTTP_PROXY`\. For these variables, AWS IoT Greengrass supports both\.

**Note**  
Most common libraries used to make connections \(such as boto3 or cURL and python `requests` packages\) use these environment variables by default\.

 

### The core is in an infinite connect\-disconnect loop\. The runtime\.log file contains a continuous series of connect and disconnect entries\.<a name="config-client-id"></a>

**Solution:** This can happen when another device is hard\-coded to use the core thing name as the client ID for MQTT connections to AWS IoT\. Simultaneous connections in the same AWS Region and AWS account must use unique client IDs\. By default, the core uses the core thing name as the client ID for these connections\.

To resolve this issue, you can change the client ID used by the other device for the connection \(recommended\) or override the default value for the core\.

**To override the default client ID for the core device**

1. Run the following command to stop the Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd stop
   ```

1. Open `greengrass-root/config/config.json` for editing as the su user\.

1. In the `coreThing` object, add the `coreClientId` property, and set the value to your custom client ID\. The value must be between 1 and 128 characters\. It must be unique in the current AWS Region for the AWS account\.

   ```
   "coreClientId": "MyCustomClientId"
   ```

1. Start the daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

 

### Error: unable to start lambda container\. container\_linux\.go:259: starting container process caused "process\_linux\.go:345: container init caused \\"rootfs\_linux\.go:62: mounting \\\\\\"proc\\\\\\" to rootfs \\\\\\"<a name="troubleshoot-mount-proc-lambda-container"></a>

**Solution:** On some platforms, you might see this error in `runtime.log` when AWS IoT Greengrass tries to mount the `/proc` file system to create a Lambda container\. Or, you might see similar errors, such as `operation not permitted` or `EPERM`\. These errors can occur even if tests run on the platform by the dependency checker script pass\.

Try one of the following possible solutions:
+ Enable the `CONFIG_DEVPTS_MULTIPLE_INSTANCES` option in the Linux kernel\.
+ Set the `/proc` mount options on the host to `rw,relatim` only\.
+ Upgrade the Linux kernel to 4\.9 or later\.

**Note**  
This issue is not related to mounting `/proc` for local resource access\.

 

### \[ERROR\]\-runtime execution error: unable to start lambda container\. \{"errorString": "failed to initialize container mounts: failed to mask greengrass root in overlay upper dir: failed to create mask device at directory <ggc\-path>: file exists"\}<a name="troubleshoot-usr-access-root"></a>

**Solution:** You might see this error in runtime\.log when the deployment fails\. This error occurs if a Lambda function in the AWS IoT Greengrass group cannot access the `/usr` directory in the core's file system\.

To resolve this issue, add a local volume resource to the group and then deploy the group\. This resource must:
+ Specify `/usr` as the **Source path** and **Destination path**\.
+ Automatically add OS group permissions of the Linux group that owns the resource\.
+ Be affiliated with the Lambda function and allow read\-only access\.

 

### \[ERROR\]\-Deployment failed\. \{"deploymentId": "<deployment\-id>", "errorString": "container test process with pid <pid> failed: container process state: exit status 1"\}<a name="troubleshoot-usr-access-root-canary"></a>

**Solution:** You might see this error in runtime\.log when the deployment fails\. This error occurs if a Lambda function in the AWS IoT Greengrass group cannot access the `/usr` directory in the core's file system\.

You can confirm that this is is the case by checking `GGCanary.log` for additional errors\. If the Lambda function cannot access the `/usr` directory, `GGCanary.log` will contain the following error:

```
[ERROR]-standard_init_linux.go:207: exec user process caused "no such file or directory"
```

To resolve this issue, add a local volume resource to the group and then deploy the group\. This resource must:
+ Specify `/usr` as the **Source path** and **Destination path**\.
+ Automatically add OS group permissions of the Linux group that owns the resource\.
+ Be affiliated with the Lambda function and allow read\-only access\.

 

### Error: \[ERROR\]\-runtime execution error: unable to start lambda container\. \{"errorString": "failed to initialize container mounts: failed to create overlay fs for container: mounting overlay at /greengrass/ggc/packages/<ggc\-version>/rootfs/merged failed: failed to mount with args source=\\"no\_source\\" dest=\\"/greengrass/ggc/packages/<ggc\-version>/rootfs/merged\\" fstype=\\"overlay\\" flags=\\"0\\" data=\\"lowerdir=/greengrass/ggc/packages/<ggc\-version>/dns:/,upperdir=/greengrass/ggc/packages/<ggc\-version>/rootfs/upper,workdir=/greengrass/ggc/packages/<ggc\-version>/rootfs/work\\": too many levels of symbolic links"\}<a name="troubleshoot-symbolic-links"></a>

**Solution:** You might see this error in `runtime.log` when the AWS IoT Greengrass Core software doesn't start and your Linux kernel version is 4\.19\.57 or earlier\. This issue might be more common on Debian operating systems\.

To resolve this issue, do one of the following:
+ If you're running AWS IoT Greengrass Core software v1\.9\.3 or later, add the `system.useOverlayWithTmpfs` property to [config\.json](gg-core.md#config-json), and set the value to `true`\. For example:

  ```
  {
    "system": {
      "useOverlayWithTmpfs": true
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
+ If you're running AWS IoT Greengrass Core software v1\.9\.2 or earlier on a Raspberry Pi, update to AWS IoT Greengrass Core software v1\.9\.3 or later\. For information about over\-the\-air updates for AWS IoT Greengrass software, see [OTA updates of AWS IoT Greengrass Core software](core-ota-update.md)\.
+ Upgrade the Linux kernel on your device\. We recommend version 4\.4 or later\.

**Note**  
Your AWS IoT Greengrass Core software version is shown in the error message\. To find your Linux kernel version, run `uname -r`\.

 

### Error: \[DEBUG\]\-Failed to get routes\. Discarding message\.<a name="troubleshoot-failed-to-get-routes"></a>

**Solution:** Check the subscriptions in your group and make sure that the subscription listed in the `[DEBUG]` message exists\. 

 

### Error: \[Errno 24\] Too many open <lambda\-function>,\[Errno 24\] Too many open files<a name="troubleshoot-too-many-open-files"></a>

**Solution:** You might see this error in your Lambda function log file if the function instantiates `StreamManagerClient` in the function handler\. We recommend that you create the client outside the handler\. For more information, see [Use StreamManagerClient to work with streams](work-with-streams.md)\. 

 

## Deployment issues<a name="gg-troubleshooting-deploymentissues"></a>

Use the following information to help troubleshoot deployment issues\.

**Topics**
+ [Your current deployment does not work and you want to revert to a previous working deployment\.](#troubleshoot-revert-deployment)
+ [You see a 403 Forbidden error on deployment in the logs\.](#troubleshoot-forbidden-deployment)
+ [A ConcurrentDeployment error occurs when you run the create\-deployment command for the first time\.](#troubleshoot-concurrent-deployment)
+ [Error: Greengrass is not authorized to assume the Service Role associated with this account, or the error: Failed: TES service role is not associated with this account\.](#troubleshoot-assume-service-role)
+ [Error: unable to execute download step in deployment\. error while downloading: error while downloading the Group definition file: \.\.\. x509: certificate has expired or is not yet valid](#troubleshoot-x509-certificate-expired)
+ [Error: An error occurred during the signature verification\. The repository is not updated and the previous index files will be used\. GPG error: https://dnw9lb6lzp2d8\.cloudfront\.net stable InRelease: The following signatures couldn't be verified because the public key is not available: NO\_PUBKEY 68D644ABDEXAMPLE](#troubleshoot-expired-missing-key)
+ [The deployment doesn't finish\.](#troubleshoot-stuck-deployment)
+ [Error: Unable to find java or java8 executables, or the error: Deployment <deployment\-id> of type NewDeployment for group <group\-id> failed error: worker with <worker\-id> failed to initialize with reason Installed Java version must be greater than or equal to 8](#java-8-runtime-requirement)
+ [The deployment doesn't finish, and runtime\.log contains multiple "wait 1s for container to stop" entries\.](#troubleshoot-wait-container-stop)
+ [The deployment doesn't finish, and `runtime.log` contains "\[ERROR\]\-Greengrass deployment error: failed to report deployment status back to cloud \{"deploymentId": "<deployment\-id>", "errorString": "Failed to initiate PUT, endpoint: https://<deployment\-status>, error: Put https://<deployment\-status>: proxyconnect tcp: x509: certificate signed by unknown authority"\}"](#troubleshoot-failed-to-report-deployment-status)
+ [Error: Deployment <deployment\-id> of type NewDeployment for group <group\-id> failed error: Error while processing\. group config is invalid: 112 or \[119 0\] don't have rw permission on the file: <path>\.](#troubleshoot-access-permissions-deployment)
+ [Error: <list\-of\-function\-arns> are configured to run as root but Greengrass is not configured to run Lambda functions with root permissions\.](#troubleshoot-root-permissions-lambda)
+ [Error: Deployment <deployment\-id> of type NewDeployment for group <group\-id> failed error: Greengrass deployment error: unable to execute download step in deployment\. error while processing: unable to load the group file downloaded: could not find UID based on user name, userName: ggc\_user: user: unknown user ggc\_user\.](#troubleshoot-could-not-find-uid)
+ [Error: \[ERROR\]\-runtime execution error: unable to start lambda container\. \{"errorString": "failed to initialize container mounts: failed to mask greengrass root in overlay upper dir: failed to create mask device at directory <ggc\-path>: file exists"\}](#troubleshoot-failed-to-initialize-container-mounts)
+ [Error: Deployment <deployment\-id> of type NewDeployment for group <group\-id> failed error: process start failed: container\_linux\.go:259: starting container process caused "process\_linux\.go:250: running exec setns process for init caused \\"wait: no child processes\\""\.](#troubleshoot-wait-child-processes)
+ [Error: \[WARN\]\-MQTT\[client\] dial tcp: lookup <host\-prefix>\-ats\.iot\.<region>\.amazonaws\.com: no such host \.\.\. \[ERROR\]\-Greengrass deployment error: failed to report deployment status back to cloud \.\.\. net/http: request canceled while waiting for connection \(Client\.Timeout exceeded while awaiting headers\)](#troubleshoot-dnssec-validation-failed)

 

### Your current deployment does not work and you want to revert to a previous working deployment\.<a name="troubleshoot-revert-deployment"></a>

**Solution:** Use the AWS IoT console or AWS IoT Greengrass API to redeploy a previous working deployment\. This deploys the corresponding group version to your core device\.

**To redeploy a deployment \(console\)**

1. On the group configuration page, choose **Deployments**\. This page displays the deployment history for the group, including the date and time, group version, and status of each deployment attempt\.

1. Find the row that contains the deployment you want to redeploy\. In the **Status** column, choose the ellipsis \(**…**\), and then choose **Re\-deploy**\.  
![\[Deployments page showing the Re-Deploy action for a deployment.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-redeployment.png)

**To redeploy a deployment \(CLI\)**

1. Use [ListDeployments](https://docs.aws.amazon.com/greengrass/latest/apireference/listdeployments-get.html) to find the ID of the deployment you want to redeploy\. For example:

   ```
   aws greengrass list-deployments --group-id 74d0b623-c2f2-4cad-9acc-ef92f61fcaf7
   ```

   The command returns the list of deployments for the group\.

   ```
   {
       "Deployments": [
           {
               "DeploymentId": "8d179428-f617-4a77-8a0c-3d61fb8446a6",
               "DeploymentType": "NewDeployment",
               "GroupArn": "arn:aws:greengrass:us-west-2:123456789012:/greengrass/groups/74d0b623-c2f2-4cad-9acc-ef92f61fcaf7/versions/8dd1d899-4ac9-4f5d-afe4-22de086efc62",
               "CreatedAt": "2019-07-01T20:56:49.641Z"
           },
           {
               "DeploymentId": "f8e4c455-8ac4-453a-8252-512dc3e9c596",
               "DeploymentType": "NewDeployment",
               "GroupArn": "arn:aws:greengrass:us-west-2::123456789012:/greengrass/groups/74d0b623-c2f2-4cad-9acc-ef92f61fcaf7/versions/4ad66e5d-3808-446b-940a-b1a788898382",
               "CreatedAt": "2019-07-01T20:41:47.048Z"
           },
           {
               "DeploymentId": "e4aca044-bbd8-41b4-b697-930ca7c40f3e",
               "DeploymentType": "NewDeployment",
               "GroupArn": "arn:aws:greengrass:us-west-2::123456789012:/greengrass/groups/74d0b623-c2f2-4cad-9acc-ef92f61fcaf7/versions/1f3870b6-850e-4c97-8018-c872e17b235b",
               "CreatedAt": "2019-06-18T15:16:02.965Z"
           }
       ]
   }
   ```
**Note**  
These AWS CLI commands use example values for the group and deployment ID\. When you run the commands, make sure to replace the example values\.

1. Use [CreateDeployment](https://docs.aws.amazon.com/greengrass/latest/apireference/createdeployment-post.html) to redeploy the target deployment\. Set the deployment type to `Redeployment`\. For example:

   ```
   aws greengrass create-deployment --deployment-type Redeployment \ 
     --group-id 74d0b623-c2f2-4cad-9acc-ef92f61fcaf7 \
     --deployment-id f8e4c455-8ac4-453a-8252-512dc3e9c596
   ```

   The command returns the ARN and ID of the new deployment\.

   ```
   {
       "DeploymentId": "f9ed02b7-c28e-4df6-83b1-e9553ddd0fc2",
       "DeploymentArn": "arn:aws:greengrass:us-west-2::123456789012:/greengrass/groups/74d0b623-c2f2-4cad-9acc-ef92f61fcaf7/deployments/f9ed02b7-c28e-4df6-83b1-e9553ddd0fc2"
   }
   ```

1. Use [GetDeploymentStatus](https://docs.aws.amazon.com/greengrass/latest/apireference/getdeploymentstatus-get.html) to get the status of the deployment\.

 

### You see a 403 Forbidden error on deployment in the logs\.<a name="troubleshoot-forbidden-deployment"></a>

**Solution:** Make sure the policy of the AWS IoT Greengrass core in the cloud includes `"greengrass:*"` as an allowed action\.

 

### A ConcurrentDeployment error occurs when you run the create\-deployment command for the first time\.<a name="troubleshoot-concurrent-deployment"></a>

**Solution:** A deployment might be in progress\. You can run [get\-deployment\-status](https://docs.aws.amazon.com/greengrass/latest/apireference/getdeploymentstatus-get.html) to see if a deployment was created\. If not, try creating the deployment again\.

 

### Error: Greengrass is not authorized to assume the Service Role associated with this account, or the error: Failed: TES service role is not associated with this account\.<a name="troubleshoot-assume-service-role"></a>

**Solution:** You might see this error when the deployment fails\. Check that a Greengrass service role is associated with your AWS account in the current AWS Region\. For more information, see [Managing the Greengrass service role \(CLI\)](service-role.md#manage-service-role-cli) or [Managing the Greengrass service role \(console\)](service-role.md#manage-service-role-console)\.

 

### Error: unable to execute download step in deployment\. error while downloading: error while downloading the Group definition file: \.\.\. x509: certificate has expired or is not yet valid<a name="troubleshoot-x509-certificate-expired"></a>

**Solution:** You might see this error in `runtime.log` when the deployment fails\. If you receive a `Deployment failed` error that contains the message `x509: certificate has expired or is not yet valid`, check the device clock\. TLS and X\.509 certificates provide a secure foundation for building IoT systems, but they require accurate times on servers and clients\. IoT devices should have the correct time \(within 15 minutes\) before they attempt to connect to AWS IoT Greengrass or other TLS services that use server certificates\. For more information, see [Using Device Time to Validate AWS IoT Server Certificates](http://aws.amazon.com/blogs/iot/using-device-time-to-validate-aws-iot-server-certificates/) on *The Internet of Things on AWS Official Blog*\.

 

### Error: An error occurred during the signature verification\. The repository is not updated and the previous index files will be used\. GPG error: https://dnw9lb6lzp2d8\.cloudfront\.net stable InRelease: The following signatures couldn't be verified because the public key is not available: NO\_PUBKEY 68D644ABDEXAMPLE<a name="troubleshoot-expired-missing-key"></a>

**Solution:** You might see this error when the trusted keys used to authenticate the APT repository packages for AWS IoT Greengrass are missing, expired, or invalid\. To resolve this issue, install the keyring package:

```
wget -O aws-iot-greengrass-keyring.deb https://d1onfpft10uf5o.cloudfront.net/greengrass-apt/downloads/aws-iot-greengrass-keyring.deb
sudo dpkg -i aws-iot-greengrass-keyring.deb
```

For more information, see [Using apt to install the AWS IoT Greengrass Core software](install-ggc.md#ggc-package-manager-install)\.

 

### The deployment doesn't finish\.<a name="troubleshoot-stuck-deployment"></a>

**Solution:** Do the following:
+ Make sure that the AWS IoT Greengrass daemon is running on your core device\. Run the following commands in your core device terminal to check whether the daemon is running and start it, if needed\.

  1. To check whether the daemon is running:

     ```
     ps aux | grep -E 'greengrass.*daemon'
     ```

     If the output contains a `root` entry for `/greengrass/ggc/packages/1.11.0/bin/daemon`, then the daemon is running\.

     The version in the path depends on the AWS IoT Greengrass Core software version that's installed on your core device\.

  1. To start the daemon:

     ```
     cd /greengrass/ggc/core/
     sudo ./greengrassd start
     ```
+ Make sure that your core device is connected and the core connection endpoints are configured properly\.

 

### Error: Unable to find java or java8 executables, or the error: Deployment <deployment\-id> of type NewDeployment for group <group\-id> failed error: worker with <worker\-id> failed to initialize with reason Installed Java version must be greater than or equal to 8<a name="java-8-runtime-requirement"></a>

**Solution:** If stream manager is enabled for the AWS IoT Greengrass core, you must install the Java 8 runtime on the core device before you deploy the group\. For more information, see the [requirements](stream-manager.md#stream-manager-requirements) for stream manager\. Stream manager is enabled by default when you use the **Default Group creation** workflow in the AWS IoT console to create a group\.

Or, disable stream manager and then deploy the group\. For more information, see [Configure stream manager settings \(console\)](configure-stream-manager.md#configure-stream-manager-console)\.

 

### The deployment doesn't finish, and runtime\.log contains multiple "wait 1s for container to stop" entries\.<a name="troubleshoot-wait-container-stop"></a>

**Solution:** Run the following commands in your core device terminal to restart the AWS IoT Greengrass daemon\.

```
cd /greengrass/ggc/core/
sudo ./greengrassd stop
sudo ./greengrassd start
```

 

### The deployment doesn't finish, and `runtime.log` contains "\[ERROR\]\-Greengrass deployment error: failed to report deployment status back to cloud \{"deploymentId": "<deployment\-id>", "errorString": "Failed to initiate PUT, endpoint: https://<deployment\-status>, error: Put https://<deployment\-status>: proxyconnect tcp: x509: certificate signed by unknown authority"\}"<a name="troubleshoot-failed-to-report-deployment-status"></a>

**Solution:** You might see this error in `runtime.log` when the Greengrass core is configured to use an HTTPS proxy connection and the proxy server certificate chain isn't trusted on the system\. To try to resolve this issue, add the certificate chain to the root CA certificate\. The Greengrass core adds the certificates from this file to the certificate pool used for TLS authentication in HTTPS and MQTT connections with AWS IoT Greengrass\.

The following example shows a proxy server CA certificate added to the root CA certificate file:

```
# My proxy CA
-----BEGIN CERTIFICATE-----
MIIEFTCCAv2gAwIQWgIVAMHSAzWG/5YVRYtRQOxXUTEpHuEmApzGCSqGSIb3DQEK
\nCwUAhuL9MQswCQwJVUzEPMAVUzEYMBYGA1UECgwP1hem9uLmNvbSBJbmMuMRww
... content of proxy CA certificate ...
+vHIRlt0e5JAm5\noTIZGoFbK82A0/nO7f/t5PSIDAim9V3Gc3pSXxCCAQoFYnui
GaPUlGk1gCE84a0X\n7Rp/lND/PuMZ/s8YjlkY2NmYmNjMCAXDTE5MTEyN2cM216
gJMIADggEPADf2/m45hzEXAMPLE=
-----END CERTIFICATE-----

# Amazon Root CA 1
-----BEGIN CERTIFICATE-----
MIIDQTCCAimgF6AwIBAgITBmyfz/5mjAo54vB4ikPmljZKyjANJmApzyMZFo6qBg
ADA5MQswCQYDVQQGEwJVUzEPMA0tMVT8QtPHRh8jrdkGA1UEChMGDV3QQDExBBKW
... content of root CA certificate ...
o/ufQJQWUCyziar1hem9uMRkwFwYVPSHCb2XV4cdFyQzR1KldZwgJcIQ6XUDgHaa
5MsI+yMRQ+hDaXJiobldXgjUka642M4UwtBV8oK2xJNDd2ZhwLnoQdeXeGADKkpy
rqXRfKoQnoZsG4q5WTP46EXAMPLE
-----END CERTIFICATE-----
```

By default, the root CA certificate file is located in `/greengrass-root/certs/root.ca.pem`\. To find the location on your core device, check the `crypto.caPath` property in [config\.json](gg-core.md#config-json)\.

**Note**  
*greengrass\-root* represents the path where the AWS IoT Greengrass Core software is installed on your device\. Typically, this is the `/greengrass` directory\.

 

### Error: Deployment <deployment\-id> of type NewDeployment for group <group\-id> failed error: Error while processing\. group config is invalid: 112 or \[119 0\] don't have rw permission on the file: <path>\.<a name="troubleshoot-access-permissions-deployment"></a>

**Solution:** Make sure that the owner group of the *<path>* directory has read and write permissions to the directory\.

 

### Error: <list\-of\-function\-arns> are configured to run as root but Greengrass is not configured to run Lambda functions with root permissions\.<a name="troubleshoot-root-permissions-lambda"></a>

**Solution:** You might see this error in `runtime.log` when the deployment fails\. Make sure that you have configured AWS IoT Greengrass to allow Lambda functions to run with root permissions\. Either change the value of `allowFunctionsToRunAsRoot` in `greengrass_root/config/config.json` to `yes` or change the Lambda function to run as another user/group\. For more information, see [Running a Lambda function as root](lambda-group-config.md#lambda-running-as-root)\.

 

### Error: Deployment <deployment\-id> of type NewDeployment for group <group\-id> failed error: Greengrass deployment error: unable to execute download step in deployment\. error while processing: unable to load the group file downloaded: could not find UID based on user name, userName: ggc\_user: user: unknown user ggc\_user\.<a name="troubleshoot-could-not-find-uid"></a>

**Solution:** If the [default access identity](lambda-group-config.md#lambda-access-identity-groupsettings) of the AWS IoT Greengrass group uses the standard system accounts, the `ggc_user` user and `ggc_group` group must be present on the device\. For instructions that show how to add the user and group, see this [step](setup-filter.rpi.md#add-ggc-user-ggc-group)\. Make sure to enter the names exactly as shown\.

 

### Error: \[ERROR\]\-runtime execution error: unable to start lambda container\. \{"errorString": "failed to initialize container mounts: failed to mask greengrass root in overlay upper dir: failed to create mask device at directory <ggc\-path>: file exists"\}<a name="troubleshoot-failed-to-initialize-container-mounts"></a>

**Solution:** You might see this error in `runtime.log` when the deployment fails\. This error occurs if a Lambda function in the Greengrass group can't access the `/usr` directory in the core's file system\. To resolve this issue, add a [local volume resource](access-local-resources.md) to the group and then deploy the group\. The resource must:
+ Specify `/usr` as the **Source path** and **Destination path**\.
+ Automatically add OS group permissions of the Linux group that owns the resource\.
+ Be affiliated with the Lambda function and allow read\-only access\.

 

### Error: Deployment <deployment\-id> of type NewDeployment for group <group\-id> failed error: process start failed: container\_linux\.go:259: starting container process caused "process\_linux\.go:250: running exec setns process for init caused \\"wait: no child processes\\""\.<a name="troubleshoot-wait-child-processes"></a>

**Solution:** You might see this error when the deployment fails\. Retry the deployment\.

 

### Error: \[WARN\]\-MQTT\[client\] dial tcp: lookup <host\-prefix>\-ats\.iot\.<region>\.amazonaws\.com: no such host \.\.\. \[ERROR\]\-Greengrass deployment error: failed to report deployment status back to cloud \.\.\. net/http: request canceled while waiting for connection \(Client\.Timeout exceeded while awaiting headers\)<a name="troubleshoot-dnssec-validation-failed"></a>

**Solution:** You might see this error if you're using `systemd-resolved`, which enables the `DNSSEC` setting by default\. As a result, many public domains are not recognized\. Attempts to reach the AWS IoT Greengrass endpoint fail to find the host, so your deployments remain in the `In Progress` state\.

You can use the following commands and output to test for this issue\. Replace the *region* placeholder in the endpoints with your AWS Region\.

```
$ ping greengrass-ats.iot.region.amazonaws.com
ping: greengrass-ats.iot.region.amazonaws.com: Name or service not known
```

```
$ systemd-resolve greengrass-ats.iot.region.amazonaws.com
greengrass-ats.iot.region.amazonaws.com: resolve call failed: DNSSEC validation failed: failed-auxiliary
```

One possible solution is to disable `DNSSEC`\. When `DNSSEC` is `false`, DNS lookups are not `DNSSEC` validated\. For more information, see this [known issue](https://github.com/systemd/systemd/issues/9867) for `systemd`\.

1. Add `DNSSEC=false` to `/etc/systemd/resolved.conf`\.

1. Restart `systemd-resolved`\.

For information about `resolved.conf` and `DNSSEC`, run `man resolved.conf` in your terminal\.

 

## Create group and create function issues<a name="gg-troubleshooting-groupcreateissues"></a>

Use the following information to help troubleshoot issues with creating an AWS IoT Greengrass group or Greengrass Lambda function\.

**Topics**
+ [Error: Your 'IsolationMode' configuration for the group is invalid\.](#troubleshoot-invalid-isolation-mode-group)
+ [Error: Your 'IsolationMode' configuration for function with arn <function\-arn> is invalid\.](#troubleshoot-isolation-mode-lambda)
+ [Error: MemorySize configuration for function with arn <function\-arn> is not allowed in IsolationMode=NoContainer\.](#troubleshoot-lambda-memorysize-not-supported)
+ [Error: Access Sysfs configuration for function with arn <function\-arn> is not allowed in IsolationMode=NoContainer\.](#troubleshoot-sysfs-access-not-supported)
+ [Error: MemorySize configuration for function with arn <function\-arn> is required in IsolationMode=GreengrassContainer\.](#troubleshoot-lambda-memorysize-required)
+ [Error: Function <function\-arn> refers to resource of type <resource\-type> that is not allowed in IsolationMode=NoContainer\.](#troubleshoot-resource-access-not-supported)
+ [Error: Execution configuration for function with arn <function\-arn> is not allowed\.](#troubleshoot-execution-parameters-not-supported)

 

### Error: Your 'IsolationMode' configuration for the group is invalid\.<a name="troubleshoot-invalid-isolation-mode-group"></a>

**Solution:** This error occurs when the `IsolationMode` value in the `DefaultConfig` of `function-definition-version` is not supported\. Supported values are `GreengrassContainer` and `NoContainer`\.

 

### Error: Your 'IsolationMode' configuration for function with arn <function\-arn> is invalid\.<a name="troubleshoot-isolation-mode-lambda"></a>

**Solution:** This error occurs when the `IsolationMode` value in the <function\-arn> of the `function-definition-version` is not supported\. Supported values are `GreengrassContainer` and `NoContainer`\.

 

### Error: MemorySize configuration for function with arn <function\-arn> is not allowed in IsolationMode=NoContainer\.<a name="troubleshoot-lambda-memorysize-not-supported"></a>

**Solution:** This error occurs when you specify a `MemorySize` value and you choose to run without containerization\. Lambda functions that are run without containerization cannot have memory limits\. You can either remove the limit or you can change the Lambda function to run in an AWS IoT Greengrass container\.

 

### Error: Access Sysfs configuration for function with arn <function\-arn> is not allowed in IsolationMode=NoContainer\.<a name="troubleshoot-sysfs-access-not-supported"></a>

**Solution:** This error occurs when you specify `true` for `AccessSysfs` and you choose to run without containerization\. Lambda functions run without containerization must have their code updated to access the file system directly and cannot use `AccessSysfs`\. You can either specify a value of `false` for `AccessSysfs` or you can change the Lambda function to run in an AWS IoT Greengrass container\.

 

### Error: MemorySize configuration for function with arn <function\-arn> is required in IsolationMode=GreengrassContainer\.<a name="troubleshoot-lambda-memorysize-required"></a>

**Solution:** This error occurs because you did not specify a `MemorySize` limit for a Lambda function that you are running in an AWS IoT Greengrass container\. Specify a `MemorySize` value to resolve the error\.

 

### Error: Function <function\-arn> refers to resource of type <resource\-type> that is not allowed in IsolationMode=NoContainer\.<a name="troubleshoot-resource-access-not-supported"></a>

**Solution:** You cannot access `Local.Device`, `Local.Volume`, `ML_Model.SageMaker.Job`, `ML_Model.S3_Object`, or `S3_Object.Generic_Archive` resource types when you run a Lambda function without containerization\. If you need those resource types, you must run in an AWS IoT Greengrass container\. You can also access local devices directly when running without containerization by changing the code in your Lambda function\.

 

### Error: Execution configuration for function with arn <function\-arn> is not allowed\.<a name="troubleshoot-execution-parameters-not-supported"></a>

**Solution:** This error occurs when you create a system Lambda function with `GGIPDetector` or `GGCloudSpooler` and you specified `IsolationMode` or `RunAs` configuration\. You must omit the `Execution` parameters for this system Lambda function\.

 

## Discovery issues<a name="gg-troubleshooting-discovery-issues"></a>

Use the following information to help troubleshoot issues with the AWS IoT Greengrass Discovery service\.

**Topics**
+ [Error: Device is a member of too many groups, devices may not be in more than 10 groups](#troubleshoot-device-in-too-many-groups)

 

### Error: Device is a member of too many groups, devices may not be in more than 10 groups<a name="troubleshoot-device-in-too-many-groups"></a>

**Solution:** This is a known limitation\. A [Greengrass device](what-is-gg.md#greengrass-devices) can be a member of up to 10 groups\.

 

## Machine learning resource issues<a name="ml-resources-troubleshooting"></a>

Use the following information to help troubleshoot issues with machine learning resources\.

**Topics**
+ [InvalidMLModelOwner \- GroupOwnerSetting is provided in ML model resource, but GroupOwner or GroupPermission is not present](#nocontainer-lambda-invalid-ml-model-owner)
+ [NoContainer function cannot configure permission when attaching Machine Learning resources\. <function\-arn> refers to Machine Learnin resource <resource\-id> with permission <ro/rw> in resource access policy\.](#nocontainer-lambda-invalid-resource-access-policy)
+ [Function <function\-arn> refers to Machine Learning resource <resource\-id> with missing permission in both ResourceAccessPolicy and resource OwnerSetting\.](#nocontainer-lambda-missing-access-permission)
+ [Function <function\-arn> refers to Machine Learning resource <resource\-id> with permission \\"rw\\", while resource owner setting GroupPermission only allows \\"ro\\"\.](#container-lambda-invalid-rw-permissions)
+ [NoContainer Function <function\-arn> refers to resources of nested destination path\.](#nocontainer-lambda-nested-destination-path)
+ [Lambda <function\-arn> gains access to resource <resource\-id> by sharing the same group owner id](#lambda-runas-and-resource-owner)

 

### InvalidMLModelOwner \- GroupOwnerSetting is provided in ML model resource, but GroupOwner or GroupPermission is not present<a name="nocontainer-lambda-invalid-ml-model-owner"></a>

**Solution:** You receive this error if a machine learning resource contains the [ResourceDownloadOwnerSetting](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-resourcedownloadownersetting.html) object but the required `GroupOwner` or `GroupPermission` property isn't defined\. To resolve this issue, define the missing property\.

 

### NoContainer function cannot configure permission when attaching Machine Learning resources\. <function\-arn> refers to Machine Learnin resource <resource\-id> with permission <ro/rw> in resource access policy\.<a name="nocontainer-lambda-invalid-resource-access-policy"></a>

**Solution:** You receive this error if a non\-containerized Lambda function specifies function\-level permissions to a machine learning resource\. Non\-containerized functions must inherit permissions from the resource owner permissions defined on the machine learning resource\. To resolve this issue, choose to [inherit resource owner permissions](access-ml-resources.md#non-container-config-console) \(console\) or [remove the permissions from the Lambda function's resource access policy](access-ml-resources.md#non-container-config-api) \(API\)\.

 

### Function <function\-arn> refers to Machine Learning resource <resource\-id> with missing permission in both ResourceAccessPolicy and resource OwnerSetting\.<a name="nocontainer-lambda-missing-access-permission"></a>

**Solution:** You receive this error if permissions to the machine learning resource aren't configured for the attached Lambda function or the resource\. To resolve this issue, configure permissions in the [ResourceAccessPolicy](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-resourceaccesspolicy.html) property for the Lambda function or the [OwnerSetting](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-ownersetting.html) property for the resource\.

 

### Function <function\-arn> refers to Machine Learning resource <resource\-id> with permission \\"rw\\", while resource owner setting GroupPermission only allows \\"ro\\"\.<a name="container-lambda-invalid-rw-permissions"></a>

**Solution:** You receive this error if the access permissions defined for the attached Lambda function exceed the resource owner permissions defined for the machine learning resource\. To resolve this issue, set more restrictive permissions for the Lambda function or less restrictive permissions for the resource owner\.

 

### NoContainer Function <function\-arn> refers to resources of nested destination path\.<a name="nocontainer-lambda-nested-destination-path"></a>

**Solution:** You receive this error if multiple machine learning resources attached to a non\-containerized Lambda function use the same destination path or a nested destination path\. To resolve this issue, specify separate destination paths for the resources\.

 

### Lambda <function\-arn> gains access to resource <resource\-id> by sharing the same group owner id<a name="lambda-runas-and-resource-owner"></a>

**Solution:** You receive this error in `runtime.log` if the same OS group is specified as the Lambda function's [Run as](lambda-group-config.md#lambda-access-identity) identity and the [resource owner](access-ml-resources.md#ml-resource-owner) for a machine learning resource, but the resource is not attached to the Lambda function\. This configuration gives the Lambda function implicit permissions that it can use to access the resource without AWS IoT Greengrass authorization\.

To resolve this issue, use a different OS group for one of the properties or attach the machine learning resource to the Lambda function\.

## AWS IoT Greengrass core in Docker issues<a name="gg-troubleshooting-dockerissues"></a>

Use the following information to help troubleshoot issues with running an AWS IoT Greengrass core in a Docker container\.

**Topics**
+ [Error: Unknown options: \-no\-include\-email\.](#docker-troubleshooting-cli-version)
+ [Warning: IPv4 is disabled\. Networking will not work\.](#docker-troubleshooting-ipv4-disabled)
+ [Error: A firewall is blocking file Sharing between windows and the containers\.](#docker-troubleshooting-firewall)
+ [Error: An error occurred \(AccessDeniedException\) when calling the GetAuthorizationToken operation: User: arn:aws:iam::<account\-id>:user/<user\-name> is not authorized to perform: ecr:GetAuthorizationToken on resource: \*](#docker-troubleshooting-ecr-perms)
+ [Error: Cannot create container for the service greengrass: Conflict\. The container name "/aws\-iot\-greengrass" is already in use\.](#troubleshoot-docker-name-conflict)
+ [Error: \[FATAL\]\-Failed to reset thread's mount namespace due to an unexpected error: "operation not permitted"\. To maintain consistency, GGC will crash and need to be manually restarted\.](#troubleshoot-docker-container-lambda)

 

### Error: Unknown options: \-no\-include\-email\.<a name="docker-troubleshooting-cli-version"></a>

**Solution:** This error can occur when you run the `aws ecr get-login` command\. Make sure that you have the latest AWS CLI version installed \(for example, run: `pip install awscli --upgrade --user`\)\. If you're using Windows and you installed the CLI using the MSI installer, you must repeat the installation process\. For more information, see [Installing the AWS Command Line Interface on Microsoft Windows](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-windows.html) in the *AWS Command Line Interface User Guide*\.

 

### Warning: IPv4 is disabled\. Networking will not work\.<a name="docker-troubleshooting-ipv4-disabled"></a>

**Solution:** You might receive this warning or a similar message when running AWS IoT Greengrass on a Linux computer\. Enable IPv4 network forwarding as described in this [step](run-gg-in-docker-container.md#docker-linux-enable-ipv4)\. AWS IoT Greengrass cloud deployment and MQTT communications don't work when IPv4 forwarding isn't enabled\. For more information, see [Configure namespaced kernel parameters \(sysctls\) at runtime](https://docs.docker.com/engine/reference/commandline/run/#configure-namespaced-kernel-parameters-sysctls-at-runtime) in the Docker documentation\.

 

### Error: A firewall is blocking file Sharing between windows and the containers\.<a name="docker-troubleshooting-firewall"></a>

**Solution:** You might receive this error or a `Firewall Detected` message when running Docker on a Windows computer\. This can also occur if you are signed in on a virtual private network \(VPN\) and your network settings are preventing the shared drive from being mounted\. In that situation, turn off VPN and re\-run the Docker container\.

 

### Error: An error occurred \(AccessDeniedException\) when calling the GetAuthorizationToken operation: User: arn:aws:iam::<account\-id>:user/<user\-name> is not authorized to perform: ecr:GetAuthorizationToken on resource: \*<a name="docker-troubleshooting-ecr-perms"></a>

You might receive this error when running the `aws ecr get-login-password` command if you don't have sufficient permissions to access an Amazon ECR repository\. For more information, see [Amazon ECR Repository Policy Examples](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-policy-examples.html) and [Accessing One Amazon ECR Repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/security_iam_id-based-policy-examples.html) in the *Amazon ECR User Guide*\.

 

### Error: Cannot create container for the service greengrass: Conflict\. The container name "/aws\-iot\-greengrass" is already in use\.<a name="troubleshoot-docker-name-conflict"></a>

**Solution:** This can occur when the container name is used by an older container\. To resolve this issue, run the following command to remove the old Docker container:

```
docker rm -f $(docker ps -a -q -f "name=aws-iot-greengrass")
```

 

### Error: \[FATAL\]\-Failed to reset thread's mount namespace due to an unexpected error: "operation not permitted"\. To maintain consistency, GGC will crash and need to be manually restarted\.<a name="troubleshoot-docker-container-lambda"></a>

**Solution:** This error in `runtime.log` can occur when you try to deploy a `GreengrassContainer` Lambda function to an AWS IoT Greengrass core running in a Docker container\. Currently, only `NoContainer` Lambda functions can be deployed to a Greengrass Docker container\.

To resolve this issue, [make sure that all Lambda functions are in `NoContainer` mode](lambda-group-config.md#change-containerization-lambda) and start a new deployment\. Then, when starting the container, don't bind\-mount the existing `deployment` directory onto the AWS IoT Greengrass core Docker container\. Instead, create an empty `deployment` directory in its place and bind\-mount that in the Docker container\. This allows the new Docker container to receive the latest deployment with Lambda functions running in `NoContainer` mode\. 

For more information, see [Running AWS IoT Greengrass in a Docker container](run-gg-in-docker-container.md)\.

## Troubleshooting with logs<a name="troubleshooting-logs"></a>

You can configure logging settings for a Greengrass group, such as whether to send logs to CloudWatch Logs, store logs on the local file system, or both\. To get detailed information when troubleshooting issues, you can temporarily change the logging level to `DEBUG`\. Changes to logging settings take effect when you deploy the group\. For more information, see [Configure logging for AWS IoT Greengrass](greengrass-logs-overview.md#config-logs)\.

On the local file system, AWS IoT Greengrass stores logs in the following locations\. Reading the logs on the file system requires root permissions\.

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

For more information about AWS IoT Greengrass logging, see [Monitoring with AWS IoT Greengrass logs](greengrass-logs-overview.md)\.

**Note**  
Logs for AWS IoT Greengrass Core software v1\.0 are stored under the `greengrass-root/var/log` directory\.

## Troubleshooting storage issues<a name="troubleshooting-storage"></a>

When the local file storage is full, some components might start failing: 
+ Local shadow updates do not occur\.
+ New AWS IoT Greengrass core MQTT server certificates cannot be downloaded locally\.
+ Deployments fail\.

You should always be aware of the amount of free space available locally\. You can calculate free space based on the sizes of deployed Lambda functions, the logging configuration \(see [Troubleshooting with logs](#troubleshooting-logs)\), and the number of shadows stored locally\. 

## Troubleshooting messages<a name="troubleshooting-messages"></a>

All messages sent locally in AWS IoT Greengrass are sent with QoS 0\. By default, AWS IoT Greengrass stores messages in an in\-memory queue\. Therefore, unprocessed messages are lost when the Greengrass core restarts; for example, after a group deployment or device reboot\. However, you can configure AWS IoT Greengrass \(v1\.6 or later\) to cache messages to the file system so they persist across core restarts\. You can also configure the queue size\. If you configure a queue size, make sure that it's greater than or equal to 262144 bytes \(256 KB\)\. Otherwise, AWS IoT Greengrass might not start properly\. For more information, see [MQTT message queue for cloud targets](gg-core.md#mqtt-message-queue)\.

**Note**  
When using the default in\-memory queue, we recommend that you deploy groups or restart the device when the service disruption is the lowest\.

You can also configure the core to establish persistent sessions with AWS IoT\. This allows the core to receive messages sent from the AWS Cloud while the core is offline\. For more information, see [MQTT persistent sessions with AWS IoT Core](gg-core.md#mqtt-persistent-sessions)\.

## Troubleshooting shadow synchronization timeout issues<a name="troubleshooting-shadow-sync"></a>

Significant delays in communication between a Greengrass core device and the cloud might cause shadow synchronization to fail because of a timeout\. In this case, you should see log entries similar to the following:

```
[2017-07-20T10:01:58.006Z][ERROR]-cloud_shadow_client.go:57,Cloud shadow client error: unable to get cloud shadow what_the_thing_is_named for synchronization. Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][WARN]-sync_manager.go:263,Failed to get cloud copy: Get https://1234567890abcd.iot.us-west-2.amazonaws.com:8443/things/what_the_thing_is_named/shadow: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
[2017-07-20T10:01:58.006Z][ERROR]-sync_manager.go:375,Failed to execute sync operation {what_the_thing_is_named VersionDiscontinued []}"
```

A possible fix is to configure the amount of time that the core device waits for a host response\. Open the [config\.json](gg-core.md#config-json) file in `greengrass-root/config` and add a `system.shadowSyncTimeout` field with a timeout value in seconds\. For example:

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

## Check the AWS IoT Greengrass forum<a name="troubleshooting-forum"></a>

If you're unable to resolve your issue using the troubleshooting information in this topic, you can search the [AWS IoT Greengrass forum](https://forums.aws.amazon.com/forum.jspa?forumID=254) for related issues or post a new forum thread\. Members of the AWS IoT Greengrass team actively monitor the forum\.