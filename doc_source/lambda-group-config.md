# Controlling execution of Greengrass Lambda functions by using group\-specific configuration<a name="lambda-group-config"></a>

AWS IoT Greengrass provides cloud\-based management of Greengrass Lambda functions\. Although a Lambda function's code and dependencies are managed using AWS Lambda, you can configure how the Lambda function behaves when it runs in a Greengrass group\.

## Group\-specific configuration settings<a name="lambda-group-config-properties"></a>

AWS IoT Greengrass provides the following group\-specific configuration settings for Greengrass Lambda functions\.

**Run as**  <a name="lambda-access-identity"></a>
The access identity used to run a Lambda function\. By default, Lambda functions run as the group's [default access identity](#lambda-access-identity-groupsettings)\. Typically, this is the standard AWS IoT Greengrass system accounts \(ggc\_user and ggc\_group\)\. You can change the setting and choose the user ID and group ID that have the permissions required to run the Lambda function\. You can override both UID and GID or just one if you leave the other field blank\. This setting gives you more granular control over access to device resources\. We recommend that you configure your Greengrass hardware with appropriate resource limits, file permissions, and disk quotas for the users and groups whose permissions are used to run Lambda functions\.  
This feature is available for AWS IoT Greengrass Core v1\.7 and later\.  
We recommend that you avoid running as root unless absolutely necessary\. When you run a Lambda function as root, you increase the risk of unintended changes, such as accidentally deleting a critical file\. In addition, running as root increases the risks to your data and device from malicious individuals\. If you do need to run as root, you must update the AWS IoT Greengrass configuration to enable it\. For more information, see [Running a Lambda function as root](#lambda-running-as-root)\.  
**UID \(number\)**  
The user ID for the user that has the permissions required to run the Lambda function\. This setting is only available if you choose **Run as another user ID/group ID**\. You can use the getent passwd command on your AWS IoT Greengrass core device to look up the user ID you want to use to run the Lambda function\.  
If you use the same UID to run processes and the Lambda function on a Greengrass core device, your Greengrass group role can grant the processes temporary credentials\. The processes can use the temporary credentials across Greengrass core deployments\.  
**GID \(number\)**  
The group ID for the group that has the permissions required to run the Lambda function\. This setting is only available if you choose **Run as another user ID/group ID**\. You can use the getent group command on your AWS IoT Greengrass core device to look up the group ID you want to use to run the Lambda function\.

**Containerization**  <a name="lambda-function-containerization"></a>
Choose whether the Lambda function runs with the default containerization for the group, or specify the containerization that should always be used for this Lambda function\.  
A Lambda function's containerization mode determines its level of isolation\.  
+ Containerized Lambda functions run in **Greengrass container** mode\. The Lambda function runs in an isolated runtime environment \(or namespace\) inside the AWS IoT Greengrass container\.
+ Non\-containerized Lambda functions run in **No container** mode\. The Lambda functions runs as a regular Linux process without any isolation\.
This feature is available for AWS IoT Greengrass Core v1\.7 and later\.  
We recommend that you run Lambda functions in a Greengrass container unless your use case requires them to run without containerization\. When your Lambda functions run in a Greengrass container, you can use attached local and device resources and gain the benefits of isolation and increased security\. Before you change the containerization, see [Considerations when choosing Lambda function containerization](#lambda-containerization-considerations)\.  
To run without enabling your device kernel namespace and cgroup, all your Lambda functions must run without containerization\. You can accomplish this easily by setting the default containerization for the group\. For information, see [Setting default containerization for Lambda functions in a group](#lambda-containerization-groupsettings)\.

**Memory limit**  
The memory allocation for the function\. The default is 16 MB\.  
The memory limit setting becomes unavailable when you change the Lambda function to run without containerization\. Lambda functions that run without containerization have no memory limit\. The memory limit setting is discarded when you change the Lambda function or group default containerization setting to run without containerization\.

**Timeout**  
The amount of time before the function or request is terminated\. The default is 3 seconds\.

**Lifecycle**  
A Lambda function lifecycle can be *on\-demand* or *long\-lived*\. The default is on\-demand\.  
An on\-demand Lambda function starts in a new or reused container when invoked\. Requests to the function might be processed by any available container\. A long\-lived—or *pinned*—Lambda function starts automatically after AWS IoT Greengrass starts and keeps running in its own container \(or sandbox\)\. All requests to the function are processed by the same container\. For more information, see [Lifecycle configuration for Greengrass Lambda functions](lambda-functions.md#lambda-lifecycle)\.

**Read access to /sys directory**  
Whether the function can access the host's /sys folder\. Use this when the function must read device information from /sys\. The default is false\.  
This setting is not available when you run a Lambda function without containerization\. The value of this setting is discarded when you change the Lambda function to run without containerization\.

**Input payload data type**  
The expected encoding type of the input payload for the function, either JSON or binary\. The default is JSON\.  
Support for the binary encoding type is available starting in AWS IoT Greengrass Core Software v1\.5\.0 and AWS IoT Greengrass Core SDK v1\.1\.0\. Accepting binary input data can be useful for functions that interact with device data, because the restricted hardware capabilities of devices often make it difficult or impossible for them to construct a JSON data type\.  
[Lambda executables](lambda-functions.md#lambda-executables) support the binary encoding type only, not JSON\.

**Environment variables**  
Key\-value pairs that can dynamically pass settings to function code and libraries\. Local environment variables work the same way as [AWS Lambda function environment variables](https://docs.aws.amazon.com/lambda/latest/dg/env_variables.html), but are available in the core environment\.

**Resource access policies**  
A list of up to 10 [local resources](access-local-resources.md), [secret resources](secrets.md), and [machine learning resources](ml-inference.md) that the Lambda function is allowed to access, and the corresponding `read-only` or `read-write` permission\. In the console, these *affiliated* resources are listed on the function's **Resources** page\.  
The [containerization mode](#lambda-function-containerization) affects how Lambda functions can access local device and volume resources and machine learning resources\.  
+ Non\-containerized Lambda functions must access local device and volume resources directly through the file system on the core device\.
+ To allow non\-containerized Lambda functions to access machine learning resources in the Greengrass group, you must set the resource owner and access permissions properties on the machine learning resource\. For more information, see [Access machine learning resources from Lambda functions](access-ml-resources.md)\.

For information about using the AWS IoT Greengrass API to set group\-specific configuration settings for user\-defined Lambda functions, see [CreateFunctionDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/createfunctiondefinition-post.html) in the *AWS IoT Greengrass API Reference* or [create\-function\-definition](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html) in the *AWS CLI Command Reference*\. To deploy Lambda functions to a Greengrass core, create a function definition version that contains your functions, create a group version that references the function definition version and other group components, and then [deploy the group](deployments.md)\.

## Running a Lambda function as root<a name="lambda-running-as-root"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

Before you can run one or more Lambda functions as root, you must first update the AWS IoT Greengrass configuration to enable support\. Support for running Lambda functions as root is off by default\. The deployment fails if you try to deploy a Lambda function and run it as root \(UID and GID of 0\) and you haven't updated the AWS IoT Greengrass configuration\. An error like the following appears in the runtime log \(*greengrass\_root*/ggc/var/log/system/runtime\.log\):

```
lambda(s)
[list of function arns] are configured to run as root while Greengrass is not configured to run lambdas with root permissions
```

**Important**  
We recommend that you avoid running as root unless absolutely necessary\. When you run a Lambda function as root, you increase the risk of unintended changes, such as accidentally deleting a critical file\. In addition, running as root increases the risks to your data and device from malicious individuals\.

**To allow Lambda functions to run as root**

1. On your AWS IoT Greengrass device, navigate to the *greengrass\-root*/config folder\.
**Note**  
By default, *greengrass\-root* is the /greengrass directory\.

1. Edit the config\.json file to add `"allowFunctionsToRunAsRoot" : "yes"` to the `runtime` field\. For example:

   ```
   {
     "coreThing" : {
       ...
     },
     "runtime" : {
       ...
       "allowFunctionsToRunAsRoot" : "yes"
     },
     ...
   }
   ```

1. Use the following commands to restart AWS IoT Greengrass:

   ```
   cd /greengrass/ggc/core
   sudo ./greengrassd restart
   ```

   Now you can set the user ID and group ID \(UID/GID\) of Lambda functions to 0 to run that Lambda function as root\.

You can change the value of `"allowFunctionsToRunAsRoot"` to `"no"` and restart AWS IoT Greengrass if you want to disallow Lambda functions to run as root\.

## Considerations when choosing Lambda function containerization<a name="lambda-containerization-considerations"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

By default, Lambda functions run inside an AWS IoT Greengrass container\. That container provides isolation between your functions and the host, which offers more security for both the host and the functions in the container\.

We recommend that you run Lambda functions in a Greengrass container unless your use case requires them to run without containerization\. By running your Lambda functions in a Greengrass container, you have more control over restricting access to resources\.

Here are some example use cases for running without containerization:
+ You want to run AWS IoT Greengrass on a device that does not support container mode \(for example, because you are using a special Linux distribution or have a kernel version that is too old\)\.
+ You want to run your Lambda function in another container environment with its own OverlayFS, but encounter OverlayFS conflicts when you run in a Greengrass container\.
+ You need access to local resources with paths that can't be determined at deployment time or whose paths can change after deployment, such as pluggable devices\.
+ You have a legacy application that was written as a process and you have encountered issues when running it as a containerized Lambda function\.


**Containerization differences**  

| Containerization | Notes | 
| --- | --- | 
| Greengrass container | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lambda-group-config.html) | 
| No container | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lambda-group-config.html) | 

**Note**  
The default containerization setting for the Greengrass group doesn't apply to [connectors](connectors.md)\.

Changing the containerization for a Lambda function can cause problems when you deploy it\. If you had assigned local resources to your Lambda function that are no longer available with your new containerization settings, deployment fails\.
+ When you change a Lambda function from running in a Greengrass container to running without containerization, memory limits for the function are discarded\. You must access the file system directly instead of using attached local resources\. You must remove any attached resources before you deploy\.
+ When you change a Lambda function from running without containerization to running in a container, your Lambda function loses direct access to the file system\. You must define a memory limit for each function or accept the default 16 MB\. You can configure those settings for each Lambda function before you deploy\.<a name="change-containerization-lambda"></a>

**To change containerization settings for a Lambda function**

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="lambda-choose-group"></a>Choose the group that contains the Lambda function whose settings you want to change\.

1. <a name="lambda-choose-lambdas"></a>Choose **Lambdas**\.

1. <a name="lambda-edit-lambda"></a>On the Lambda function that you want to change, choose the ellipsis \(**…**\) and then choose **Edit configuration**\.

1. Change the containerization settings\. If you configure the Lambda function to run in a Greengrass container, you must also set **Memory limit** and **Read access to /sys directory**\.

1. <a name="lambda-save-changes"></a>Choose **Update** to save the changes to your Lambda function\.

The changes take effect when the group is deployed\.

You can also use the [CreateFunctionDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/createfunctiondefinition-post.html) and [CreateFunctionDefinitionVersion](https://docs.aws.amazon.com/greengrass/latest/apireference/createfunctiondefinitionversion-post.html) in the *AWS IoT Greengrass API Reference*\. If you are changing the containerization setting, be sure to update the other parameters too\. For example, if you are changing from running a Lambda function in a Greengrass container to running without containerization, be sure to clear the `MemorySize` parameter\.

### Determine the isolation modes supported by your Greengrass device<a name="dependency-checker-tests-isolation"></a>

You can use the AWS IoT Greengrass dependency checker to determine which isolation modes \(Greengrass container/no container\) are supported by your Greengrass device\.

**To run the AWS IoT Greengrass dependency checker**

1. Download and run the AWS IoT Greengrass dependency checker from the [GitHub repository](https://github.com/aws-samples/aws-greengrass-samples)\.

   ```
   wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.11.x.zip
   unzip greengrass-dependency-checker-GGCv1.11.x.zip
   cd greengrass-dependency-checker-GGCv1.11.x
   sudo modprobe configs
   sudo ./check_ggc_dependencies | more
   ```

1. Where `more` appears, press the Spacebar key to display another page of text\.

For information about the modprobe command, run man modprobe in the terminal\. 

## Setting the default access identity for Lambda functions in a group<a name="lambda-access-identity-groupsettings"></a>

This feature is available for AWS IoT Greengrass Core v1\.8 and later\.

For more control over access to device resources, you can configure the default access identity used to run Lambda functions in the group\. This setting determines the default permissions given to your Lambda functions when they run on the core device\. To override the setting for individual functions in the group, you can use the function's **Run as** property\. For more information, see [Run as](#lambda-access-identity)\.

This group\-level setting is also used for running the underlying AWS IoT Greengrass Core software\. This consists of system Lambda functions that manage operations, such as message routing, local shadow sync, and automatic IP address detection\.

The default access identity can be configured to run as the standard AWS IoT Greengrass system accounts \(ggc\_user and ggc\_group\) or use the permissions of another user or group\. We recommend that you configure your Greengrass hardware with appropriate resource limits, file permissions, and disk quotas for any users and groups whose permissions are used to run user\-defined or system Lambda functions\.

**To modify the default access identity for your AWS IoT Greengrass group**

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-group"></a>Choose the group whose settings you want to change\.

1. <a name="group-choose-settings"></a>Choose **Settings**\.

1. Under **Lambda runtime environment**, for **Default Lambda function user ID/ group ID**, choose **Another user ID/group ID**\.

   When you choose this option, the **UID \(number\)** and **GID \(number\)** fields are displayed\.

1. Enter a user ID, group ID, or both\. If you leave a field blank, the respective Greengrass system account \(ggc\_user or ggc\_group\) is used\.
   + For **UID \(number\)**, enter the user ID for the user who has the permissions you want to use by default to run Lambda functions in the group\. You can use the getent passwd command on your AWS IoT Greengrass device to look up the user ID\.
   + For **GID \(number\)**, enter the group ID for the group that has the permissions you want to use by default to run Lambda functions in the group\. You can use the getent group command on your AWS IoT Greengrass device to look up the group ID\.
**Important**  
Running as the root user increases risks to your data and device\. Do not run as root \(UID/GID=0\) unless your business case requires it\. For more information, see [Running a Lambda function as root](#lambda-running-as-root)\.

The changes take effect when the group is deployed\.

## Setting default containerization for Lambda functions in a group<a name="lambda-containerization-groupsettings"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

The containerization setting for a Greengrass group determines the default containerization for the Lambda functions in the group\.
+ In **Greengrass container** mode, Lambda functions run in an isolated runtime environment inside the AWS IoT Greengrass container by default\.
+ In **No container** mode, Lambda functions run as regular Linux processes by default\.

You can modify group settings to specify the default containerization for Lambda functions in the group\. You can override this setting for one or more Lambda functions in the group if you want the Lambda functions to run with containerization different from the group default\. Before you change containerization settings, see [Considerations when choosing Lambda function containerization](#lambda-containerization-considerations)\.

**Important**  
If you want to change the default containerization for the group, but have one or more functions that use a different containerization, change the settings for the Lambda functions before you change the group setting\. If you change the group containerization setting first, the values for the **Memory limit** and **Read access to /sys directory** settings are discarded\.

**To modify containerization settings for your AWS IoT Greengrass group**

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-group"></a>Choose the group whose settings you want to change\.

1. <a name="group-choose-settings"></a>Choose **Settings**\.

1. Under **Lambda runtime environment**, change the containerization setting\.

The changes take effect when the group is deployed\.