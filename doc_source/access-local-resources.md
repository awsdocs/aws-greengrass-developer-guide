# Access local resources with Lambda functions and connectors<a name="access-local-resources"></a>

This feature is available for AWS IoT Greengrass Core v1\.3 and later\.

With AWS IoT Greengrass, you can author AWS Lambda functions and configure [connectors](connectors.md) in the cloud and deploy them to core devices for local execution\. On Greengrass cores running Linux, these locally deployed Lambda functions and connectors can access local resources that are physically present on the Greengrass core device\. For example, to communicate with devices that are connected through Modbus or CANbus, you can enable your Lambda function to access the serial port on the core device\. To configure secure access to local resources, you must guarantee the security of your physical hardware and your Greengrass core device OS\.

To get started accessing local resources, see the following tutorials:
+ [How to configure local resource access using the AWS command line interface](lra-cli.md)
+ [How to configure local resource access using the AWS Management Console](lra-console.md)

## Supported resource types<a name="lra-resource-types"></a>

You can access two types of local resources: volume resources and device resources\.

**Volume resources**  
Files or directories on the root file system \(except under `/sys`, `/dev`, or `/var`\)\. These include:  
+ Folders or files used to read or write information across Greengrass Lambda functions \(for example, `/usr/lib/python2.x/site-packages/local`\)\.
+ Folders or files under the host's /proc file system \(for example, `/proc/net` or `/proc/stat`\)\. Supported in v1\.6 or later\. For additional requirements, see [Volume resources under the /proc directory](#lra-proc-resources)\.
To configure the `/var`, `/var/run`, and `/var/lib` directories as volume resources, first mount the directory in a different folder and then configure the folder as a volume resource\.
When you configure volume resources, you specify a *source* path and a *destination* path\. The source path is the absolute path of the resource on the host\. The destination path is the absolute path of the resource inside the Lambda namespace environment\. This is the container that a Greengrass Lambda function or connector runs in\. Any changes to the destination path are reflected in the source path on the host file system\.  
Files in the destination path are visible in the Lambda namespace only\. You can't see them in a regular Linux namespace\.

**Device resources**  
Files under `/dev`\. Only character devices or block devices under `/dev` are allowed for device resources\. These include:  
+ Serial ports used to communicate with devices connected through serial ports \(for example, `/dev/ttyS0`, `/dev/ttyS1`\)\.
+ USB used to connect USB peripherals \(for example, `/dev/ttyUSB0` or `/dev/bus/usb`\)\.
+ GPIOs used for sensors and actuators through GPIO \(for example, `/dev/gpiomem`\)\.
+ GPUs used to accelerate machine learning using on\-board GPUs \(for example, `/dev/nvidia0`\)\.
+ Cameras used to capture images and videos \(for example, `/dev/video0`\)\.
`/dev/shm` is an exception\. It can be configured as a volume resource only\. Resources under `/dev/shm` must be granted `rw` permission\.

AWS IoT Greengrass also supports resource types that are used to perform machine learning inference\. For more information, see [Perform machine learning inference](ml-inference.md)\.

## Requirements<a name="lra-requirements"></a>

The following requirements apply to configuring secure access to local resources:
+ You must be using AWS IoT Greengrass Core Software v1\.3 or later\. To create resources for the host's /proc directory, you must be using v1\.6 or later\.
+ The local resource \(including any required drivers and libraries\) must be correctly installed on the Greengrass core device and consistently available during use\.
+ The desired operation of the resource, and access to the resource, must not require root privileges\. 
+ Only `read` or `read and write` permissions are available\. Lambda functions cannot perform privileged operations on the resources\.
+ You must provide the full path of the local resource on the operating system of the Greengrass core device\.
+ A resource name or ID has a maximum length of 128 characters and must use the pattern `[a-zA-Z0-9:_-]+`\.

### Volume resources under the /proc directory<a name="lra-proc-resources"></a>

The following considerations apply to volume resources that are under the host's /proc directory\.
+ You must be using AWS IoT Greengrass Core Software v1\.6 or later\.
+ You can allow read\-only access for Lambda functions, but not read\-write access\. This level of access is managed by AWS IoT Greengrass\.
+ You might also need to grant OS group permissions to enable read access in the file system\. For example, suppose your source directory or file has a 660 file permission, which means that only the owner or user in the group has read \(and write\) access\. In this case, you must add the OS group owner's permissions to the resource\. For more information, see [Group owner file access permission](#lra-group-owner)\.
+ The host environment and the Lambda namespace both contain a /proc directory, so be sure to avoid naming conflicts when you specify the destination path\. For example, if /proc is the source path, you can specify /host\-proc as the destination path \(or any path name other than "*/proc*"\)\.

## Group owner file access permission<a name="lra-group-owner"></a>

An AWS IoT Greengrass Lambda function process normally runs as `ggc_user` and `ggc_group`\. However, you can give additional file access permissions to the Lambda function process in the local resource definition, as follows:
+ To add the permissions of the Linux group that owns the resource, use the `GroupOwnerSetting#AutoAddGroupOwner` parameter or **Automatically add OS group permissions of the Linux group that owns the resource** console option\.
+ To add the permissions of a different Linux group, use the `GroupOwnerSetting#GroupOwner` parameter or **Specify another OS group to add permission** console option\. The `GroupOwner` value is ignored if `GroupOwnerSetting#AutoAddGroupOwner` is true\.

An AWS IoT Greengrass Lambda function process inherits all of the file system permissions of `ggc_user`, `ggc_group`, and the Linux group \(if added\)\. For the Lambda function to access a resource, the Lambda function process must have the required permissions to the resource\. You can use the `chmod(1)` command to change the permission of the resource, if necessary\.

## See also<a name="lra-seealso"></a>
+ [Service Quotas](https://docs.aws.amazon.com/general/latest/gr/greengrass.html#limits_greengrass) for resources in the *Amazon Web Services General Reference*