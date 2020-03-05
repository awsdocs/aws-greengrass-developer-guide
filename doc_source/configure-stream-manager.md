# Configure AWS IoT Greengrass Stream Manager<a name="configure-stream-manager"></a>

On the AWS IoT Greengrass core, stream manager can store, process, and export data sent from IoT devices\. Stream manager provides parameters that you use to configure group\-level runtime settings\. These settings apply to all streams on the AWS IoT Greengrass core\. You can use the AWS IoT console or AWS IoT Greengrass API to configure stream manager settings\. Changes take effect after the group is deployed\.

## Stream Manager Parameters<a name="stream-manager-parameters"></a>

Stream manager provides the following parameters that allow you to define group\-level settings\. All parameters are optional\.

**Storage directory**  <a name="STREAM_MANAGER_STORE_ROOT_DIR"></a>
Parameter name: `STREAM_MANAGER_STORE_ROOT_DIR`  
The absolute path of the local directory used to store streams\. This value must start with a forward slash \(for example, `/data`\)\.  
For information about securing stream data, see [Local Data Security](stream-manager.md#stream-manager-security-stream-data)\.

**Server port**  
Parameter name: `STREAM_MANAGER_SERVER_PORT`  
The local port number used to communicate with stream manager\. The default is `8088`\.

**Authenticate client**  <a name="STREAM_MANAGER_AUTHENTICATE_CLIENT"></a>
Parameter name: `STREAM_MANAGER_AUTHENTICATE_CLIENT`  
Indicates whether clients must be authenticated to interact with stream manager\. All interaction between clients and stream manager is controlled by the AWS IoT Greengrass Core SDK\. This parameter determines which clients can call the AWS IoT Greengrass Core SDK to work with streams\. For more information, see [Client Authentication](stream-manager.md#stream-manager-security-client-authentication)\.  
Valid values are `true` or `false`\. The default is `true` \(recommended\)\.  
+ `true`\. Allows only Greengrass Lambda functions as clients\. Lambda function clients use internal AWS IoT Greengrass core protocols to authenticate with the AWS IoT Greengrass Core SDK\.
+ `false`\. Allows any process that runs on the AWS IoT Greengrass core to be a client\. Do not set to `false` unless your business case requires it\. For example, set this value to `false` only if non\-Lambda processes on the core device must communicate directly with stream manager, such as [Docker containers](docker-app-connector.md) running on the core\.

**Maximum bandwidth**  
Parameter name: `STREAM_MANAGER_EXPORTER_MAX_BANDWIDTH`  
The average maximum bandwidth \(in kilobits per second\) that can be used to export data\. The default allows unlimited use of available bandwidth\.

**Thread pool size**  
Parameter name: `STREAM_MANAGER_EXPORTER_THREAD_POOL_SIZE`  
The maximum number of active threads that can be used to export data\. The default is `5`\.  
The optimal size depends on your hardware, stream volume, and planned number of export streams\. If your export speed is slow, you can adjust this setting to find the optimal size for your hardware and business case\. The CPU and memory of your core device hardware are limiting factors\. To start, you might try setting this value equal to the number of processor cores on the device\.  
Be careful not to set a size that's higher than your hardware can support\. Each stream consumes hardware resources, so you should try to limit the number of export streams on constrained devices\.

**JVM arguments**  
Parameter name: `JVM_ARGS`  
Custom Java Virtual Machine arguments to pass to stream manager at startup\. Multiple arguments should be separated by spaces\.  
Use this parameter only when you must override the default settings used by the JVM\. For example, you might need to increase the default heap size if you plan to export a large number of streams\.

## Configure Stream Manager Settings \(Console\)<a name="configure-stream-manager-console"></a>

You can use the AWS IoT console for the following management tasks:
+ [Check If Stream Manager Is Enabled](#check-stream-manager-console)
+ [Enable or Disable Stream Manager During Group Creation](#enable-stream-manager-console-new-group)
+ [Enable or Disable Stream Manager for an Existing Group](#enable-stream-manager-console-existing-group)
+ [Change Stream Manager Settings](#change-stream-manager-console)

Changes take effect after the Greengrass group is deployed\.

**Note**  
When you use the console to enable stream manager and deploy the group, the memory limit for stream manager is set to 4 GB\.

 

### To check if stream manager is enabled \(console\)<a name="check-stream-manager-console"></a>

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="group-choose-settings"></a>Choose **Settings**\.

1. Under **Stream manager**, check the enabled or disabled status\. Any custom stream manager settings that are configured are also displayed\.  
![\[The Stream manager section on the Settings page of the AWS IoT console.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-settings-stream-manager.png)

 

### To enable or disable stream manager during group creation \(console\)<a name="enable-stream-manager-console-new-group"></a>

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. Choose **Create Group**\. Your choice on the next page determines how you configure stream manager for the group\.

1. To create the group with default group settings, which also enables stream manager with default stream manager settings:

   1. Choose **Use default creation**\.

   1. Skip to [step 5](#continue-create-group)\.

1. To create the group with custom group settings:

   1. Choose **Customize**\.

   1. Proceed through the **Name your Group** and **Attach an IAM Role to your Group** pages\.

   1. On the **Stream manager** page, configure stream manager for the group:
      + To enable stream manager with default settings, choose **Use defaults**\.

         
      + To enable stream manager with custom settings, choose **Customize settings**\.

        1. On the **Configure stream manager** page, choose **Enable**\.

        1. Under **Custom settings**, enter values for stream manager parameters\. For more information, see [Stream Manager Parameters](#stream-manager-parameters)\. Leave fields empty to allow AWS IoT Greengrass to use their default values\.

            
      + To disable stream manager, choose **Customize settings**\.

        1. On the **Configure stream manager** page, choose **Disable**\.

              
![\[The Stream manager page in the create group workflow.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/create-group-stream-manager.png)

1. Choose **Next**\.

1. <a name="continue-create-group"></a>Continue through the remaining pages to create your group\.

1. On the **Connect your Core device** page, download your security resources, review the information, and then choose **Finish**\.
**Note**  
When stream manager is enabled, you must [install the Java 8 runtime](stream-manager.md#stream-manager-requirements) on the core device before you deploy the group\.

 

### To enable or disable stream manager for an existing group \(console\)<a name="enable-stream-manager-console-existing-group"></a>

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="group-choose-settings"></a>Choose **Settings**\.

1. The enabled or disabled status is displayed under **Stream manager**, along with any custom stream manager settings\. Choose **Edit**\.

1. Choose **Enable** or **Disable**\.

1. Choose **Save**\.

 

### To change stream manager settings \(console\)<a name="change-stream-manager-console"></a>

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="group-choose-settings"></a>Choose **Settings**\.

1. The enabled or disabled status is displayed under **Stream manager**, along with any custom stream manager settings\. Choose **Edit**\.

1. Edit values for [stream manager parameters](#stream-manager-parameters)\. Leave fields empty to allow AWS IoT Greengrass to use default values for the corresponding parameters\.

1. Choose **Save**\.

## Configure Stream Manager Settings \(CLI\)<a name="configure-stream-manager-cli"></a>

In the AWS CLI, you use the system `GGStreamManager` Lambda function to configure stream manager\. System Lambda functions are components of the AWS IoT Greengrass Core software\. In some cases, you can configure Greengrass functionality by managing the corresponding `Function` and `FunctionDefinitionVersion` objects in the AWS IoT Greengrass group object model\. For more information, see [Overview of the AWS IoT Greengrass Group Object Model](deployments.md#api-overview)\.

You can use the CLI for the following management tasks:
+ [Check if Stream Manager Is Enabled](#check-stream-manager-cli)
+ [Enable, Disable, or Configure Stream Manager Settings](#enable-stream-manager-cli)

Changes take effect after the group is deployed\.

**Tip**  
To see if stream manager is enabled and running, you can run the following command in a terminal on your core device\.  

```
ps aux | grep -i 'streammanager'
```

 

### To check if stream manager is enabled \(CLI\)<a name="check-stream-manager-cli"></a>

Stream manager is enabled if your deployed function definition version includes the system `GGStreamManager` Lambda function\. To check, do the following;

1. <a name="get-group-id-latestversion"></a>Get the IDs of the target Greengrass group and group version\. In this procedure, we assume this is the latest group and group version\. The following command returns the most recently created group\.

   ```
   aws greengrass list-groups --query "reverse(sort_by(Groups, &CreationTimestamp))[0]"
   ```

   Or, you can query by name\. Group names are not required to be unique, so multiple groups might be returned\.

   ```
   aws greengrass list-groups --query "Groups[?Name=='MyGroup']"
   ```
**Note**  
<a name="find-group-ids-console"></a>You can also find these values in the AWS IoT console\. The group ID is displayed on the group's **Settings** page\. Group version IDs are displayed on the group's **Deployments** page\.

1. <a name="copy-group-id-latestversion"></a>Copy the `Id` and `LatestVersion` values from the target group in the output\.

1. <a name="get-latest-group-version"></a>Get the latest group version\.
   + Replace *group\-id* with the `Id` that you copied\.
   + Replace *latest\-group\-version\-id* with the `LatestVersion` that you copied\.

   ```
   aws greengrass get-group-version \
   --group-id group-id \
   --group-version-id latest-group-version-id
   ```

1. From the `FunctionDefinitionVersionArn` in the output, get the IDs of the function definition and function definition version\.
   + The function definition ID is the GUID that follows the `functions` segment in the ARN\.
   + The function definition version ID is the GUID that follows the `versions` segment in the ARN\.

   ```
   arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/functions/function-definition-id/versions/function-definition-version-id
   ```

1. Get the function definition version\.
   + Replace *function\-definition\-id* with the function definition ID\.
   + Replace *function\-definition\-version\-id* with the function definition version ID\.

   ```
   aws greengrass get-function-definition-version \
   --function-definition-id function-definition-id \
   --function-definition-version-id function-definition-version-id
   ```

If the `functions` array in the output includes the `GGStreamManager` function, then stream manager is enabled\. Any environment variables defined for the function represent custom settings for stream manager\.

 

### To enable, disable, or configure stream manager settings \(CLI\)<a name="enable-stream-manager-cli"></a>

In the AWS CLI, you use the system `GGStreamManager` Lambda function to configure stream manager\. Changes take effect after you deploy the group\.
+ To enable stream manager, include `GGStreamManager` in the `functions` array of your function definition version\.
+ To disable stream manager, remove `GGStreamManager` from the `functions` array of your function definition version\.
+ To configure custom settings when stream manager is enabled, include `GGStreamManager` in the `functions` array of your function definition version\. Also include environment variables for the function that set the corresponding [parameters](#stream-manager-parameters)\.  
**Enable stream manager with default settings**  
The following example configuration enables stream manager with default settings\. For a tutorial that enables stream manager with default settings, see [Export Data Streams to the AWS Cloud \(CLI\)](stream-manager-cli.md)\.  

  ```
  {
      "FunctionArn": "arn:aws:lambda:::function:GGStreamManager:1",
      "FunctionConfiguration": {
          "MemorySize": 128000,
          "Pinned": true,
  >       "Timeout": 3
      },
      "Id": "arbitrary-function-id"
  }
  ```  
**Enable stream manager with custom settings**  
The following example configuration enables stream manager with custom settings for the storage directory, server port, and thread pool size\.  

  ```
  {
      "FunctionArn": "arn:aws:lambda:::function:GGStreamManager:1",
      "FunctionConfiguration": {
          "Environment": {
              "Variables": {
                  "STREAM_MANAGER_STORE_ROOT_DIR": "/data",
                  "STREAM_MANAGER_SERVER_PORT": "1234",
                  "STREAM_MANAGER_EXPORTER_THREAD_POOL_SIZE": "4"
              }
          },
          "MemorySize": 128000,
          "Pinned": true,
          "Timeout": 3
      },
      "Id": "arbitrary-function-id"
  }
  ```

  For the `FunctionConfiguration` parameters, `MemorySize` should be at least `128000`\. `Pinned` must be set to `true`\.
**Note**  
`Timeout` is required by the function definition version, but `GGStreamManager` doesn't use it\.

 

1. <a name="get-group-id-latestversion"></a>Get the IDs of the target Greengrass group and group version\. In this procedure, we assume this is the latest group and group version\. The following command returns the most recently created group\.

   ```
   aws greengrass list-groups --query "reverse(sort_by(Groups, &CreationTimestamp))[0]"
   ```

   Or, you can query by name\. Group names are not required to be unique, so multiple groups might be returned\.

   ```
   aws greengrass list-groups --query "Groups[?Name=='MyGroup']"
   ```
**Note**  
<a name="find-group-ids-console"></a>You can also find these values in the AWS IoT console\. The group ID is displayed on the group's **Settings** page\. Group version IDs are displayed on the group's **Deployments** page\.

1. <a name="copy-group-id-latestversion"></a>Copy the `Id` and `LatestVersion` values from the target group in the output\.

1. <a name="get-latest-group-version"></a>Get the latest group version\.
   + Replace *group\-id* with the `Id` that you copied\.
   + Replace *latest\-group\-version\-id* with the `LatestVersion` that you copied\.

   ```
   aws greengrass get-group-version \
   --group-id group-id \
   --group-version-id latest-group-version-id
   ```

1. <a name="parse-function-def-id"></a>From the `FunctionDefinitionVersionArn` in the output, copy the ID of the function definition\. The ID is the GUID that follows the `functions` segment in the ARN, as shown in the following example\.

   ```
   arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/functions/bcfc6b49-beb0-4396-b703-6dEXAMPLEcu5/versions/0f7337b4-922b-45c5-856f-1aEXAMPLEsf6
   ```
**Note**  
Or, you can create a function definition by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html) command, and then copy the ID from the output\.

1. Add a function definition version to the function definition\.
   + Replace *function\-definition\-id* with the `Id` that you copied for the function definition\.
   + Replace *arbitrary\-function\-id* with a name for the function, such as **stream\-manager**\.
   + In the `functions` array, include all other functions that you want to make available on the core\. You can use the `get-function-definition-version` command to get the list of existing functions\.

   The following example uses environment variables to set custom values for stream manager parameters\. AWS IoT Greengrass uses default values for properties that are omitted\.

   ```
   aws greengrass create-function-definition-version \
   --function-definition-id function-definition-id \
   --functions '[{"FunctionArn": "arn:aws:lambda:::function:GGStreamManager:1", "FunctionConfiguration": {"Environment": {"Variables":{"STREAM_MANAGER_STORE_ROOT_DIR": "/data","STREAM_MANAGER_SERVER_PORT": "1234","STREAM_MANAGER_EXPORTER_MAX_BANDWIDTH": "20000"}},"Timeout": 3,"MemorySize": 128000,"Pinned": true},"Id": "arbitrary-function-id"}]'
   ```

   `MemorySize` should be at least `128000`\. `Pinned` must be set to `true`\.
**Note**  
`Timeout` is required by the function definition version, but `GGStreamManager` doesn't use it\.

1. <a name="copy-function-def-version-arn"></a>Copy the `Arn` of the function definition version from the output\.

1. <a name="create-group-version-with-sys-lambda"></a>Create a group version that contains the system Lambda function\.
   + Replace *group\-id* with the `Id` for the group\.
   + Replace *core\-definition\-version\-arn* with the `CoreDefinitionVersionArn` that you copied from the latest group version\.
   + Replace *function\-definition\-version\-arn* with the `Arn` that you copied for the new function definition version\.
   + Replace the ARNs for other group components \(for example, `SubscriptionDefinitionVersionArn` or `DeviceDefinitionVersionArn`\) that you copied from the latest group version\.
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

1. <a name="create-group-deployment"></a>Deploy the group with the new group version\.
   + Replace *group\-id* with the `Id` that you copied for the group\.
   + Replace *group\-version\-id* with the `Version` that you copied for the new group version\.

   ```
   aws greengrass create-deployment \
   --group-id group-id \
   --group-version-id group-version-id \
   --deployment-type NewDeployment
   ```

To update these settings, you create a function definition version that includes the `GGStreamManager` function with the updated configuration\. The `functions` array in the `FunctionDefinitionVersion` must include all Lambda functions that you want to deploy to the core\. You can use the [https://docs.aws.amazon.com/greengrass/latest/apireference/get-function-definition-version.html](https://docs.aws.amazon.com/greengrass/latest/apireference/get-function-definition-version.html) command to get the Greengrass Lambda functions from an existing function definition version\. Changes take effect after the group is deployed\.

## See Also<a name="configure-stream-manager-see-also"></a>
+ [Manage Data Streams on the AWS IoT Greengrass Core](stream-manager.md)
+ [Use StreamManagerClient to Work with Streams](work-with-streams.md)
+ [Export Data Streams to the AWS Cloud \(Console\)](stream-manager-console.md)
+ [Export Data Streams to the AWS Cloud \(CLI\)](stream-manager-cli.md)