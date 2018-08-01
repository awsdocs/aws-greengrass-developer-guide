# Configure the AWS Greengrass Core<a name="gg-core"></a>

An AWS Greengrass core is an AWS IoT thing\. Like other AWS IoT devices, a core exists in the registry, has a device shadow, and uses a device certificate to authenticate with AWS IoT\. The core device runs the AWS Greengrass core software, which enables it to manage local processes for Greengrass groups, such as communication, shadow sync, and token exchange\.

The AWS Greengrass core software consists of:
+ A message manager that routes messages between devices, Lambda functions, and AWS IoT\.
+ A Lambda runtime that runs user\-defined Lambda functions\.
+ An implementation of the Device Shadow service that provides a local copy of shadows, which represent your devices\. Shadows can be configured to sync with the cloud\. 
+ A deployment agent that is notified of new or updated AWS Greengrass group configuration\. When a new or updated configuration is detected, the deployment agent downloads the configuration data and restarts the AWS Greengrass core\.

**Note**  
*greengrass\-root* represents the path to where the AWS Greengrass core software is installed on your device\. If you installed the software by following the [Getting Started](gg-gs.md) tutorial, then this is the `/greengrass` directory\.

## AWS Greengrass Core Configuration File<a name="config-json"></a>

The configuration file for the AWS Greengrass core software is the `config.json` file, which is located in the /*greengrass\-root*/config directory\. You can review the contents of this file by running the following command:

```
cat /greengrass-root/config/config.json
```

**Note**  
In AWS Greengrass Core v1\.0\.0, `config.json` is deployed to /*greengrass\-root*/configuration\.

The following is an example `config.json` file\.

------
#### [ GGC v1\.6\.0 ]

```
{
   "coreThing": {
       "caPath": "root-ca-pem",
       "certPath": "cloud-pem-crt",
       "keyPath": "cloud-pem-key",
       "thingArn": "thing-arn",
       "iotHost": "host-prefix.iot.aws-region.amazonaws.com",
       "ggHost": "greengrass.iot.aws-region.amazonaws.com",
       "keepAlive": 600,
       "mqttMaxConnectionRetryInterval": 60
   },
   "runtime": {
       "cgroup": {
           "useSystemd": "yes"
       }
   },
   "managedRespawn": true,
   "writeDirectory": "/write-directory"
}
```

**Note**  
If you use the **Easy group creation** option from the AWS Greengrass console, then the config\.json file is deployed to the core device in a working state that specifies the default configuration\.

The `config.json` file supports the following properties:


****  

| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the [AWS IoT root CA](http://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem) relative to the `/greengrass-root/certs` directory\.  |  Save the file under `/greengrass-root/certs`\.  | 
| certPath |  The path to the AWS Greengrass core certificate relative to the `/greengrass-root/certs` directory\.  | Save the file under /greengrass\-root/certs\. | 
| keyPath | The path to the AWS Greengrass core private key relative to /greengrass/certs directory\. | Save the file under /greengrass\-root/certs\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS Greengrass core device\. | Find this for your core in the AWS Greengrass console under Cores, or by running the [http://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html](http://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html) CLI command\. | 
| iotHost | Your AWS IoT endpoint\. | Find this in the AWS IoT console under Settings, or by running the [http://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](http://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) CLI command\. | 
| ggHost | Your AWS Greengrass endpoint\. | This value uses the format greengrass\.iot\.region\.amazonaws\.com\. Use the same region as iotHost\. | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default is 600\. | 
|  `mqttMaxConnectionRetryInterval`  |  The maximum interval \(in seconds\) between MQTT connection retries if the connection is dropped\.  |  Specify this value as an unsigned integer\. This is an optional value\. The default is `60`\.  | 
| useSystemd | Indicates whether your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Valid values are yes or no\. Run the check\_ggc\_dependencies script in [Module 1](module1.md) to see if your device uses systemd\. | 
|  `managedRespawn`  |  An optional over\-the\-air \(OTA\) updates feature, this indicates that the OTA agent needs to run custom code before an update\.  |  Valid values are `true` or `false`\. For more information, see [OTA Updates of AWS Greengrass Core Software](core-ota-update.md)\.  | 
|  `writeDirectory`  |  The write directory where AWS Greengrass creates all read\-write resources\.  |  This is an optional value\. For more information, see [Configure a Write Directory for AWS Greengrass](#write-directory)\.  | 

------
#### [ GGC v1\.5\.0 ]

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
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS Greengrass core\. | Find this for your core in the AWS Greengrass console under Cores, or by running the [http://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html](http://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html) command\. | 
| iotHost | Your AWS IoT endpoint\. | Find this in the AWS IoT console under Settings, or by running the [http://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](http://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) command\. | 
| ggHost | Your AWS Greengrass endpoint\. | This value uses the format greengrass\.iot\.region\.amazonaws\.com\. Use the same region as iotHost\. | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default value is 600 seconds\. | 
| useSystemd | Indicates whether your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Values are yes or no\. Use the dependency script in [Module 1](module1.md) to see if your device uses systemd\. | 
|  `managedRespawn`  |  An optional over\-the\-air \(OTA\) updates feature, this indicates that the OTA agent needs to run custom code before an update\.  |  For more information, see [OTA Updates of AWS Greengrass Core Software](core-ota-update.md)\.  | 

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
| ggHost | Your AWS Greengrass endpoint\. | You can find it in the AWS IoT console under Settings with greengrass\. prepended\. | 
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
| ggHost | Your AWS Greengrass endpoint\. | You can find it in the AWS IoT console under Settings with greengrass\. prepended\. | 
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
| ggHost | Your AWS Greengrass endpoint\. |  You can find it in the AWS IoT console under **Settings** with `greengrass.` prepended\.  | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default value is 600 seconds\. | 
| useSystemd | A binary flag if your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Values are yes or no\. Use the dependency script in [Module 1](module1.md) to see if your device uses systemd\. | 

------

## Configure a Write Directory for AWS Greengrass<a name="write-directory"></a>

This feature is available for AWS Greengrass Core v1\.6\.0 only\.

By default, the AWS Greengrass core software is deployed under a single root directory where AWS Greengrass performs all read and write operations\. However, you can configure AWS Greengrass to use a separate directory for all write operations, including creating directories and files\. In this case, AWS Greengrass uses two top\-level directories:
+ The *greengrass\-root* directory, which you can leave as read\-write or optionally make read\-only\. This contains the AWS Greengrass core software and other critical components that should remain immutable during runtime, such as certificates and config\.json\.
+ The specified write directory\. This contains writable content, such as logs, state information, and deployed user\-defined Lambda functions\.

This configuration results in the following directory structure\.

**Greengrass root directory**  

```
greengrass-root/
|-- certs/
|   |-- root.ca.pem
|   |-- hash.cert.pem
|   |-- hash.private.key
|   |-- hash.public.key
|-- config/
|   |-- config.json
|-- ggc/
|   |-- packages/
|       |-- package-version/
|           |-- bin/
|               |-- daemon 
|           |-- greengrassd
|           |-- lambda/
|           |-- LICENSE/
|           |-- release_notes_package-version.html
|               |-- runtime/
|                   |-- java8/
|                   |-- nodejs6.10/
|                   |-- python2.7/
|   |-- core/
```

**Write Directory**  

```
write-directory/
|-- packages/
|   |-- package-version/
|       |-- ggc_root/
|       |-- rootfs_nosys/
|       |-- rootfs_sys/
|       |-- var/
|-- deployment/
|   |-- group/
|       |-- group.json
|   |-- lambda/
|   |-- mlmodel/
|-- var/
|   |-- log/
|   |-- state/
```

 

**To Configure a Write Directory**

1. Run the following command to stop the AWS Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd stop
   ```

1. Open `greengrass-root/config/config.json` for editing as the su user\.

1. Add `writeDirectory` as a parameter and specify the path to the target directory, as shown in the following example\.

   ```
   {
       "coreThing": {
           "caPath": "root-CA.pem",
           "certPath": "hash.pem.crt",
           ...
       },
       ...
       "writeDirectory" : "/write-directory"
   }
   ```
**Note**  
You can update the `writeDirectory` setting as often as you want\. After the setting is updated, AWS Greengrass uses the newly specified write directory at the next start, but doesn't migrate content from the previous write directory\.

1. Now that your write directory is configured, you can optionally make the *greengrass\-root* directory read\-only\. For instructions, see [To Make the Greengrass Root Directory Read\-Only](#configure-ro-directory)\.

   Otherwise, start the AWS Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

 <a name="configure-ro-directory"></a>

**To Make the Greengrass Root Directory Read\-Only**

1. Grant required access to the AWS Greengrass:
**Note**  
This step is required only if you want to make the Greengrass root directory read\-only\. The write directory must be configured before you begin this procedure\.

   1. Give read and write permissions to the `config.json` owner\.

      ```
      sudo chmod 0600 /greengrass-root/config/config.json
      ```

   1. Make ggc\_user the owner of the certs and system Lambda directories\.

      ```
      sudo chown -R ggc_user:ggc_group /greengrass-root/certs/
      sudo chown -R ggc_user:ggc_group /greengrass-root/ggc/packages/1.6.0/lambda/
      ```

1. Make the *greengrass\-root* directory read\-only by using your preferred mechanism\.
**Note**  
One way to make the *greengrass\-root* directory read\-only is to mount the directory as read\-only\. However, to apply over\-the\-air \(OTA\) updates to core software in a mounted directory, the directory must first be unmounted, and then remounted after the update\. You can add these `umount` and `mount` operations to the `ota_pre_update` and `ota_post_update` scripts\. For more information about OTA updates, see [Greengrass OTA Agent](core-ota-update.md#ota-agent) and [AWS Greengrass Core Update with Managed Respawn](core-ota-update.md#managed-respawn-ggc)\.

1. Start the daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

   If the permissions from step 1 aren't set correctly, then the daemon won't start\.

## MQTT Message Queue<a name="mqtt-message-queue"></a>

MQTT messages that are destined for cloud targets are queued to await processing\. Queued messages are processed in first in first out \(FIFO\) order\. After a message is processed and published to the cloud, the message is removed from the queue\. AWS Greengrass manages the queue by using a system\-defined `GGCloudSpooler` Lambda function\.

### Configure the MQTT Message Queue<a name="configure-message-queue"></a>

This feature is available for AWS Greengrass Core v1\.6\.0 only\.

You can configure AWS Greengrass to store unprocessed messages in memory or in a local storage cache\. Unlike in\-memory storage, the local storage cache has the ability to persist across core restarts \(for example, after a group deployment or a device reboot\), so AWS Greengrass can continue to process the messages\. You can also configure the storage size\.

**Note**  
Versions 1\.5\.0 and earlier use in\-memory storage with a queue size of 2\.5 MB\. You cannot configure storage settings for earlier versions\.

The following environment variables for the `GGCloudSpooler` Lambda function are used to define storage settings\.
+ **GG\_CONFIG\_STORAGE\_TYPE**\. The location of the message queue\. The following are valid values:
  + `FileSystem`\. Store unprocessed messages in the local storage cache\. When the core restarts, queued messages are retained for processing\. Messages are removed after they are processed\.
  + `Memory` \(default\)\. Store unprocessed messages in memory\. When the core restarts, queued messages are lost\.

    This option is optimized for devices with restricted hardware capabilities\. When using this configuration, we recommend that you deploy groups or restart the device when the service disruption is the lowest\.
+ **GG\_CONFIG\_MAX\_SIZE\_BYTES**\. The storage size, in bytes\. This value can be any non\-negative integer **greater than or equal to 262144** \(256 KB\); a smaller size prevents the AWS Greengrass core software from starting\. The default size is 2\.5 MB\. When the size limit is reached, the oldest queued messages are replaced by new messages\.

#### To Cache Messages in Local Storage<a name="configure-local-storage-cache"></a>

To configure AWS Greengrass to cache messages to the file system so they persist across core restarts, you create a function definition version where the `GGCloudSpooler` function specifies the `FileSystem` storage type\. You must use the AWS Greengrass API to configure the local storage cache\. You can't do this in the console\.

The following procedure uses the [ `create-function-definition-version`](http://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition-version.html) CLI command to configure the spooler to save queued messages to the file system\. It also configures a 2\.6 MB queue size\.

**Note**  
This procedure assumes that you're updating the configuration of the latest group version of an existing group\.

1. <a name="get-group-id-latestversion"></a>Get the IDs of the target Greengrass group and group version\.

   ```
   aws greengrass list-groups
   ```

1. <a name="copy-group-id-latestversion"></a>Copy the `Id` and `LatestVersion` properties of your target group from the output\.

1. <a name="get-latest-group-version"></a>Get the latest group version\.
   + Replace *group\-id* with the `Id` that you copied\.
   + Replace *latest\-group\-version\-id* with the `LatestVersion` that you copied\.

   ```
   aws greengrass get-group-version \
   --group-id group-id \
   --group-version-id latest-group-version-id
   ```

1. <a name="copy-group-component-arns-except-function"></a>From the `Definition` object in the output, copy the `ComponentDefinitionVersionArn` for each group component except `FunctionDefinitionVersionArn`\. You use these values when you create a new group version\.

1. <a name="parse-function-def-id"></a>From the `FunctionDefinitionVersionArn` in the output, copy the ID of the function definition\. The ID is the GUID that follows the `functions` segment in the ARN\.
**Note**  
You can optionally create a function definition by running the [http://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html](http://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html) command, and then copy the ID from the output\.

1. Add a function definition version to the function definition\.
   + Replace *function\-definition\-id* with the `Id` that you copied for the function definition\.
   + Replace *arbitrary\-function\-id* with a name for the function, such as **spooler\-function**\.
   + Add any Lambda functions that you want to include in this version to the `functions` array\. You can use the [http://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html](http://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html) command to get the Greengrass Lambda functions from an existing function definition version\.
**Warning**  
Make sure that you specify a value for `GG_CONFIG_MAX_SIZE_BYTES` that's **greater than or equal to 262144**\. A smaller size prevents the AWS Greengrass core software from starting\.

   ```
   aws greengrass create-function-definition-version \
   --function-definition-id function-definition-id \
   --functions '[{"FunctionArn": "arn:aws:lambda:::function:GGCloudSpooler:1","FunctionConfiguration": {"Environment": {"Variables":{"GG_CONFIG_MAX_SIZE_BYTES":"2621440","GG_CONFIG_STORAGE_TYPE":"FileSystem"}},"Executable": "spooler","MemorySize": 32768,"Pinned": true,"Timeout": 3},"Id": "arbitrary-function-id"}]'
   ```

1. <a name="copy-function-def-version-arn"></a>Copy the `Arn` of the function definition version from the output\.

1. <a name="create-group-version-with-sys-lambda"></a>Create a group version that contains the system\-defined Lambda function\.
   + Replace *group\-id* with the `Id` for the group\.
   + Replace *core\-definition\-version\-arn* with the `CoreDefinitionVersionArn` that you copied from the latest group version\.
   + Replace *function\-definition\-version\-arn* with the `Arn` that you copied for the new function definition version\.
   + Replace the ARNs for other group components by using the `ComponentDefinitionVersionArn` values that you copied from the latest group version\.
   + Remove any unused parameters\. For example, remove the `--resource-definition-version-arn` if your group version doesn't contain any resources\.

   ```
   aws greengrass create-group-version \
   --group-id group-id \
   --core-definition-version-arn core-definition-version-arn \
   --function-definition-version-arn function-definition-version-arn \
   --device-definition-version-arn device-definition-version-arn \
   --logger-definition-version-arn logger-definition-version-arn \
   --resource-definition-version-arn resource-definition-version-arn \
   --subscription-definition-version-arn subscription-definition-version-arn
   ```

1. <a name="copy-group-version-id"></a>Copy the `Version` from the output\. This is the ID of the new group version\.

1. <a name="create-group-deployment"></a>Deploy the group\.
   + Replace *group\-id* with the `Id` that you copied for the group\.
   + Replace *group\-version\-id* with the `Version` that you copied for the new group version\.

   ```
   aws greengrass create-deployment \
   --group-id group-id \
   --group-version-id group-version-id \
   --deployment-type NewDeployment
   ```

To update the storage settings, you use the AWS Greengrass API to create a new function definition version that contains the `GGCloudSpooler` function with the updated configuration\. Then add the function definition version to a new group version \(along with your other group components\) and deploy the group version\. If you want to restore the default configuration, you can create a function definition version that doesn't include the `GGCloudSpooler` function\.

This system\-defined Lambda function isn't visible in the console\. However, after the function is added to the latest group version, it's included in deployments that you make from the console \(unless you use the API to replace or remove it\)\.

## Configure the Init System to Start the Greengrass Daemon<a name="start-on-boot"></a>

It's a good practice to set up your init system to start the Greengrass daemon during boot, especially when managing large fleets of devices\.

There are different types of init system, such as initd, systemd, and SystemV, and they use similar configuration parameters\. The following example is a service file for systemd\. The `Type` parameter is set to `forking` because greengrassd \(which is used to start Greengrass\) forks the Greengrass daemon process, and the `Restart` parameter is set to `on-failure` to direct systemd to restart Greengrass if Greengrass enters a failed state\.

**Note**  
To see if your device uses systemd, run the `check_ggc_dependencies` script as described in [Module 1](module1.md)\. Then to use systemd, make sure that the `useSystemd` parameter in [`config.json`](#config-json) is set to `yes`\.

```
[Unit]
Description=Greengrass Daemon

[Service]
Type=forking
PIDFile=/var/run/greengrassd.pid
Restart=on-failure
ExecStart=/greengrass/ggc/core/greengrassd start
ExecReload=/greengrass/ggc/core/greengrassd restart
ExecStop=/greengrass/ggc/core/greengrassd stop

[Install]
WantedBy=multi-user.target
```

For information about how to create and enable a service file for systemd on a Raspberry Pi, see [SYSTEMD](https://www.raspberrypi.org/documentation/linux/usage/systemd.md) in the Raspberry Pi documentation\.