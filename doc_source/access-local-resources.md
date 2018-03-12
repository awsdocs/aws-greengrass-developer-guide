# Access Local Resources with Lambda Functions<a name="access-local-resources"></a>

This feature is available for AWS Greengrass Core v1\.3\.0 only\.

 Developers who use AWS Greengrass can author AWS Lambda functions in the cloud and deploy them to core devices for local execution\. On Greengrass cores running Linux, these locally deployed Lambda functions can access local resources that are physically present on the host Greengrass core device\. For example, to communicate with devices that are connected through Modbus or CANbus, you can enable your Lambda function to access the serial port on the core device\. To configure secure access to local resources, you must guarantee the security of your physical hardware and your Greengrass core device OS\.

To get started accessing local resources, see the following tutorials:

+ [How to Configure Local Resource Access Using the AWS Command Line Interface](lra-cli.md)

+ [How to Configure Local Resource Access Using the AWS Management Console](lra-console.md)

## Supported Resource Types<a name="lra-resource-types"></a>

You can access two types of resources—volumes and devices:

**Volume resources**  
Files or directories on the root file system \(except under `/sys`, `/proc`, `/dev`, and `/var`\)\. Only a regular file or directory is allowed for a volume resource\.  
For example:  

+ Folders or files used to read or write information across Greengrass Lambda functions \(e\.g\. `/usr/lib/python2.x/site-packages/local`\)
To configure the `/var`, `/var/run`, and `/var/lib` directories as volume resources, first mount the directory in a different folder and then configure the folder as a volume resource\.

**Device resources**  
Files under `/dev`\. Only character devices or block devices under `/dev` are allowed for device resources\.  
For example:  

+ Serial ports used to communicate with devices connected through serial ports \(e\.g\. `/dev/ttyS0`, `/dev/ttyS1`\)

+ USB used to connect USB peripherals \(e\.g\. `/dev/ttyUSB0` or `/dev/bus/usb`\)

+ GPIOs used for sensors and actuators through GPIO \(e\.g\. `/dev/gpiomem`\)

+ GPUs used to accelerate machine learning using on\-board GPUs \(e\.g\. `/dev/nvidia0`\)

+ Cameras used to capture images and videos \(e\.g\. `/dev/video0`\)
An exception is `/dev/shm`, which can be configured as a volume resource only\. Resources under `/dev/shm` must be granted `rw` permission\.

## Requirements<a name="lra-requirements"></a>

The following requirements apply for configuring secure access to local resources:

+ You must be using AWS Greengrass Core version 1\.3\.0\.

+ The local resource \(including any required drivers and libraries\) must be properly installed on the Greengrass core device and consistently available during use\.

+ The desired operation of the resource, and access to the resource, must not require root privileges\. 

+ Only `read` or `read and write` permissions are available\. Lambdas cannot perform privileged operations on the resources\.

+ You must provide the full path of the local resource on the host operating system\.

+ A resource name or ID has a maximum length of 128 characters and must use the pattern `[a-zA-Z0-9:_-]+`\.

## Group Owner File Access Permission<a name="lra-group-owner"></a>

An AWS Greengrass Lambda function process normally runs as `ggc_user` and `ggc_group`\. However, you can give additional file access permissions to the Lambda function process in the local resource definition, as follows:

+ To add the permissions of the Linux group that owns the resource, use the `GroupOwnerSetting#AutoAddGroupOwner` parameter or **Automatically add OS group** console option\.

+ To add the permissions of a different Linux group, use the `GroupOwnerSetting#GroupOwner` parameter or **Specify another OS group** console option\. The `GroupOwner` value is ignored if `GroupOwnerSetting#AutoAddGroupOwner` is true\.

An AWS Greengrass Lambda function process inherits all the file system permissions of `ggc_user`, `ggc_group`, and the Linux group \(if added\)\. In order for the Lambda function to access a resource, you need to make sure that the Lambda function process has the required permissions to the resource, using the `chmod(1)` command to change the permission of the resource, if necessary\.

### See Also<a name="lra-seealso"></a>

+ [ AWS Greengrass Limits](http://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#limits_greengrass) in the *AWS General Reference*