# What Is AWS Greengrass?<a name="what-is-gg"></a>

AWS Greengrass is software that extends AWS Cloud capabilities to local devices, making it possible for those devices to collect and analyze data closer to the source of information, while also securely communicating with each other on local networks\. More specifically, developers who use AWS Greengrass can author serverless code \(AWS Lambda functions\) in the cloud and conveniently deploy it to devices for local execution of applications\.

The following diagram shows the basic architecture of AWS Greengrass\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/greengrass.png)

AWS Greengrass makes it possible for customers to use Lambda functions to build IoT devices and application logic\. Specifically, AWS Greengrass provides cloud\-based management of applications that can be deployed for local execution\. Locally deployed Lambda functions are triggered by local events, messages from the cloud, or other sources\.

In AWS Greengrass, devices securely communicate on a local network and exchange messages with each other without having to connect to the cloud\. AWS Greengrass provides a local pub/sub message manager that can intelligently buffer messages if connectivity is lost so that inbound and outbound messages to the cloud are preserved\.

AWS Greengrass protects user data:
+ Through the secure authentication and authorization of devices\.
+ Through secure connectivity in the local network\.
+ Between local devices and the cloud\.

Device security credentials function in a group until they are revoked, even if connectivity to the cloud is disrupted, so that the devices can continue to securely communicate locally\.

AWS Greengrass provides secure, over\-the\-air software updates of Lambda functions\.

AWS Greengrass consists of:
+ Software distributions
  + AWS Greengrass core software
  + AWS Greengrass core SDK
+ Cloud service
  + AWS Greengrass API
+ Features
  + Lambda runtime
  + Shadows implementation
  + Message manager
  + Group management
  + Discovery service
  + Over\-the\-air update agent
  + Local resource access
  + Machine learning inference

## AWS Greengrass Core Software<a name="gg-core"></a>

The AWS Greengrass core software provides the following functionality:
+ Allows deployment and execution of local applications created using Lambda functions and managed through the deployment API\.
+ Enables local messaging between devices over a secure network using a managed subscription scheme through the MQTT protocol\.
+ Ensures secure connections between devices and the cloud using device authentication and authorization\.
+ Provides secure, over\-the\-air software updates of user\-defined Lambda functions\.

The AWS Greengrass core software consists of:
+ A message manager that routes messages between devices, Lambda functions, and AWS IoT\.
+ A Lambda runtime that runs user\-defined Lambda functions\.
+ An implementation of the Device Shadow service that provides a local copy of shadows, which represent your devices\. Shadows can be configured to sync with the cloud\. 
+ A deployment agent that is notified of new or updated AWS Greengrass group configuration\. When a new or updated configuration is detected, the deployment agent downloads the configuration data and restarts the AWS Greengrass core\.

AWS Greengrass core instances are configured through AWS Greengrass APIs that create and update AWS Greengrass group definitions stored in the cloud\.

### AWS Greengrass Core Versions<a name="ggc-versions"></a>

The following tabs describe what's new and changed in AWS Greengrass core software versions\.

------
#### [ GGC v1\.6\.0 ]

Current version\.

New features:
+ Lambda executables that run binary code on the Greengrass core\. Use the new AWS Greengrass Core SDK for C to write Lambda executables in C and C\+\+\. For more information, see [Lambda Executables](lambda-functions.md#lambda-executables)\.
+ Optional local storage message cache that can persist across restarts\. You can configure the storage settings for MQTT messages that are queued for processing\. For more information, see [MQTT Message Queue](gg-core.md#mqtt-message-queue)\.
+ Configurable maximum reconnect retry interval for when the core device is disconnected\. For more information, see the `mqttMaxConnectionRetryInterval` property in [AWS Greengrass Core Configuration File](gg-core.md#config-json)\.
+ Local resource access to the host /proc directory\. For more information, see [Access Local Resources with Lambda Functions](access-local-resources.md)\.
+ Configurable write directory\. The AWS Greengrass core software can be deployed to read\-only and read\-write locations\. For more information, see [Configure a Write Directory for AWS Greengrass](gg-core.md#write-directory)\.

Bug fixes and improvements:
+ Performance improvement for publishing messages in the Greengrass core and between devices and the core\.
+ Reduced the compute resources required to process logs generated by user\-defined Lambda functions\.

------
#### [ GGC v1\.5\.0 ]

New features:
+ AWS Greengrass Machine Learning \(ML\) Inference is generally available\. You can perform ML inference locally on AWS Greengrass devices using models that are built and trained in the cloud\. For more information, see [Perform Machine Learning Inference](ml-inference.md)\.
+ Greengrass Lambda functions now support binary data as input payload, in addition to JSON\. To use this feature, you must upgrade to AWS Greengrass Core SDK version 1\.1\.0, which you can download from the **Software** page in the AWS IoT console\.

Bug fixes and improvements:
+ Reduced the overall memory footprint\.
+ Performance improvements for sending messages to the cloud\.
+ Performance and stability improvements for the download agent, Device Certificate Manager, and OTA update agent\.
+ Minor bug fixes\.

------
#### [ GGC v1\.3\.0 ]

New features:
+ Over\-the\-air \(OTA\) update agent capable of handling cloud\-deployed, Greengrass update jobs\. The agent is found under the new `/greengrass/ota` directory\. For more information, see [OTA Updates of AWS Greengrass Core Software](core-ota-update.md)\.
+ Local resource access feature allows Greengrass Lambda functions to access local resources, such as peripheral devices and volumes\. For more information, see [Access Local Resources with Lambda Functions](access-local-resources.md)\.

------
#### [ GGC v1\.1\.0 ]

New features:
+ Deployed AWS Greengrass groups can be reset by deleting Lambda functions, subscriptions, and configurations\. For more information, see [Reset Deployments](reset-deployments-scenario.md)\.
+ Support for Node\.js 6\.10 and Java 8 Lambda runtimes, in addition to Python 2\.7\.

To migrate from the previous version of the AWS Greengrass core:
+ Copy certificates from the `/greengrass/configuration/certs` folder to `/greengrass/certs`\.
+ Copy `/greengrass/configuration/config.json` to `/greengrass/config/config.json`\.
+ Run `/greengrass/ggc/core/greengrassd` instead of `/greengrass/greengrassd`\.
+ Deploy the group to the new core\.

------
#### [ GGC v1\.0\.0 ]

Initial version\.

------

## AWS Greengrass Groups<a name="gg-group"></a>

An AWS Greengrass group definition is a collection of settings for AWS Greengrass core devices and the devices that communicate with them\. The following diagram shows the objects that make up an AWS Greengrass group\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-group.png)

In the preceding diagram:

A: AWS Greengrass group definition  
A collection of information about the AWS Greengrass group\.

B: AWS Greengrass group settings  
These include:   
+ AWS Greengrass group role\.
+ Log configuration\.
+ Certification authority and local connection configuration\.
+ AWS Greengrass core connectivity information\.

C: AWS Greengrass core  
The AWS IoT thing that represents the AWS Greengrass core\.

D: Lambda function definition  
A list of Lambda functions to be deployed to the AWS Greengrass core of the group, with associated configuration data\. For more information, see [Run Lambda Functions on the AWS Greengrass Core](lambda-functions.md)\.

E: Subscription definition  
A list of subscriptions that enable communication using MQTT messages\. A subscription defines:  
+ A message source\. This can be an AWS IoT thing \(device\), Lambda function, AWS IoT, or the local AWS Greengrass shadow service\.
+ A subject, which is an MQTT topic or topic filter that's used to filter message data\.
+ A message target, which identifies the destination for messages published by the message source\. This can be an AWS IoT thing \(device\), Lambda function, AWS IoT, or the local AWS Greengrass shadow service\.
For more information, see [Greengrass Messaging Workflow](gg-sec.md#gg-msg-workflow)\.

F: Device definition  
A list containing an AWS Greengrass core and AWS IoT things that are members of the AWS Greengrass group, with associated configuration data\. This data specifies which devices are AWS Greengrass cores and which devices should sync shadow data with AWS IoT\.

G: Resource definition  
A list of local resources and machine learning resources on the AWS Greengrass core, with associated configuration data\. For more information, see [Access Local Resources with Lambda Functions](access-local-resources.md) and [Perform Machine Learning Inference](ml-inference.md)\.

When deployed, the AWS Greengrass group definition, Lambda functions, resources, and subscription table are copied to an AWS Greengrass core device\.

## Devices in AWS Greengrass<a name="devices"></a>

There are two types of devices:
+ AWS Greengrass cores\.
+ AWS IoT devices connected to an AWS Greengrass core\.

An AWS Greengrass core is an AWS IoT device that runs specialized AWS Greengrass software that communicates directly with the AWS IoT and AWS Greengrass cloud services\. It is an AWS IoT device with its own certificate used for authenticating with AWS IoT\. It has a device shadow and exists in the AWS IoT device registry\. AWS Greengrass cores run a local Lambda runtime, a deployment agent, and an IP address tracker that sends IP address information to the AWS Greengrass cloud service to allow AWS IoT devices to automatically discover their group and core connection information\.

Any AWS IoT device can connect to an AWS Greengrass core\. These devices run software written with the AWS IoT Device SDK\. 

The following table shows how these device types are related\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/devices.png)

The AWS Greengrass core device stores certificates in two locations:
+ Core device certificate in /greengrass/certs \- The core device certificate is named `hash.cert.pem` \(for example, `86c84488a5.cert.pem`\)\. This certificate is used to authenticate the core when connecting to the AWS IoT and AWS Greengrass services\.
+ MQTT core server certificate in /greengrass/ggc/var/state/server \- The MQTT core server certificate is named `server.crt`\. This certificate is used for mutual authentication between the local MQTT service \(that's on the Greengrass core\) and Greengrass devices before messages are exchanged\.

## SDKs<a name="gg-sdks"></a>

The following SDKs are used when working with AWS Greengrass:

------
#### [ GGC 1\.6\.0 ]

AWS SDKs  
Using the AWS SDKs, you can build applications that work with any AWS service, including Amazon S3, Amazon DynamoDB, AWS IoT, AWS Greengrass, and more\. In the context of AWS Greengrass, you can use the AWS SDK in deployed Lambda functions to make direct calls to any AWS service\. For more information, see [SDKs for Greengrass Lambda Functions](lambda-functions.md#lambda-sdks)\.

AWS IoT Device SDKs  
The AWS IoT Device SDKs helps devices connect to AWS IoT or AWS Greengrass services\. Devices must know which AWS Greengrass group they belong to and the IP address of the AWS Greengrass core that they should connect to\.   
Although you can use any of the AWS IoT Device SDKs to connect to an AWS Greengrass core, only the C\+\+ and Python Device SDKs provide AWS Greengrass\-specific functionality, such as access to the AWS Greengrass Discovery Service and AWS Greengrass core root CA downloads\. For more information, see [AWS IoT Device SDK](https://aws.amazon.com/iot/sdk/)\.

AWS Greengrass Core SDK  
The AWS Greengrass Core SDK enables Lambda functions to interact with the AWS Greengrass core on which they run in order to publish messages, interact with the local Device Shadow service, or invoke other deployed Lambda functions\. This SDK is used exclusively for writing Lambda functions running in the Lambda runtime on an AWS Greengrass core\. For more information, see [SDKs for Greengrass Lambda Functions](lambda-functions.md#lambda-sdks)\.

------
#### [ GGC v1\.5\.0 ]

AWS SDKs  
Using the AWS SDKs, you can build applications that work with any AWS service, including Amazon S3, Amazon DynamoDB, AWS IoT, AWS Greengrass, and more\. In the context of AWS Greengrass, you can use the AWS SDK inside deployed Lambda functions to make direct calls to any AWS service\.

AWS IoT Device SDKs  
The AWS IoT Device SDKs helps devices connect to AWS IoT or AWS Greengrass services\. Devices must know which AWS Greengrass group they belong to and the IP address of the AWS Greengrass core that they should connect to\.   
Although you can use any of the AWS IoT Device SDKs to connect to an AWS Greengrass core, only the C\+\+ and Python Device SDKs provide AWS Greengrass\-specific functionality, such as access to the AWS Greengrass Discovery Service and AWS Greengrass core root CA downloads\. For more information, see [AWS IoT Device SDK](https://aws.amazon.com/iot/sdk/)\.

AWS Greengrass Core SDK  
The AWS Greengrass Core SDK enables Lambda functions to interact with the AWS Greengrass core on which they run in order to publish messages, interact with the local Device Shadow service, or invoke other deployed Lambda functions\. This SDK is used exclusively for writing Lambda functions running in the Lambda runtime on an AWS Greengrass core\. Lambda functions running on an AWS Greengrass core can interact with AWS cloud services directly using the AWS SDK\. The AWS Greengrass Core SDK and the AWS SDK are contained in different packages, so you can use both packages simultaneously\. You can download the AWS Greengrass Core SDK from the **Software** page of the [AWS IoT console](https://console.aws.amazon.com/iot/home)\.  
The AWS Greengrass Core SDK follows the AWS SDK programming model\. It allows you to easily port Lambda functions developed for the cloud to Lambda functions that run on an AWS Greengrass core\. For example, using the AWS SDK, the following Lambda function publishes a message to the topic `"/some/topic"` in the cloud:   

```
import boto3
        
client = boto3.client('iot-data')
response = client.publish(
	topic = "/some/topic",
	qos = 0,
	payload = "Some payload".encode()
)
```
To port this Lambda function for execution on an AWS Greengrass core, replace the `import boto3` statement with the `import greengrasssdk`, as shown in the following snippet:  
The AWS Greengrass Core SDK only supports sending MQTT messages with QoS = 0\.

```
import greengrasssdk
                        
client = greengrasssdk.client('iot-data')
response = client.publish(
    topic='/some/topic',
    qos=0,
    payload='some payload'.encode()
)
```
This allows you to test your Lambda functions in the cloud and migrate them to AWS Greengrass with minimal effort\.

**Note**  
The AWS SDK is natively part of the environment of a Lambda function that runs in the AWS cloud\. If you want to use `boto3` in a Lambda function that's deployed to an AWS Greengrass core, make sure to include the AWS SDK in your package\. In addition, if you use both the AWS Greengrass Core SDK and the AWS SDK in the same package, make sure that your Lambda functions use the correct namespaces\. For more information about how to create your deployment package, see:  
[Creating a Deployment Package \(Python\)](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html)
[ Creating a Deployment Package \(Node\.js\)](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-create-deployment-pkg.html)
[ Creating a Deployment Package \(Java\)](https://docs.aws.amazon.com/lambda/latest/dg/lambda-java-how-to-create-deployment-package.html)

------
#### [ GGC v1\.3\.0 ]

AWS SDKs  
Using the AWS SDKs, you can build applications that work with any AWS service, including Amazon S3, Amazon DynamoDB, AWS IoT, AWS Greengrass, and more\. In the context of AWS Greengrass, you can use the AWS SDK inside deployed Lambda functions to make direct calls to any AWS service\.

AWS IoT Device SDKs  
The AWS IoT Device SDKs helps devices connect to AWS IoT or AWS Greengrass services\. Devices must know which AWS Greengrass group they belong to and the IP address of the AWS Greengrass core that they should connect to\.   
Although you can use any of the AWS IoT Device SDKs to connect to an AWS Greengrass core, only the C\+\+ and Python Device SDKs provide AWS Greengrass\-specific functionality, such as access to the AWS Greengrass Discovery Service and AWS Greengrass core root CA downloads\. For more information, see [AWS IoT Device SDK](https://aws.amazon.com/iot/sdk/)\.

AWS Greengrass Core SDK  
The AWS Greengrass Core SDK enables Lambda functions to interact with the AWS Greengrass core on which they run in order to publish messages, interact with the local Device Shadow service, or invoke other deployed Lambda functions\. This SDK is used exclusively for writing Lambda functions running in the Lambda runtime on an AWS Greengrass core\. Lambda functions running on an AWS Greengrass core can interact with AWS cloud services directly using the AWS SDK\. The AWS Greengrass Core SDK and the AWS SDK are contained in different packages, so you can use both packages simultaneously\. You can download the AWS Greengrass Core SDK from the **Software** page of the [AWS IoT console](https://console.aws.amazon.com/iot/home)\.  
The AWS Greengrass Core SDK follows the AWS SDK programming model\. It allows you to easily port Lambda functions developed for the cloud to Lambda functions that run on an AWS Greengrass core\. For example, using the AWS SDK, the following Lambda function publishes a message to the topic `"/some/topic"` in the cloud:   

```
import boto3
        
client = boto3.client('iot-data')
response = client.publish(
	topic = "/some/topic",
	qos = 0,
	payload = "Some payload".encode()
)
```
To port this Lambda function for execution on an AWS Greengrass core, replace the `import boto3` statement with the `import greengrasssdk`, as shown in the following snippet:  
The AWS Greengrass Core SDK only supports sending MQTT messages with QoS = 0\.

```
import greengrasssdk
                        
client = greengrasssdk.client('iot-data')
response = client.publish(
    topic='/some/topic',
    qos=0,
    payload='some payload'.encode()
)
```
This allows you to test your Lambda functions in the cloud and migrate them to AWS Greengrass with minimal effort\.

**Note**  
The AWS SDK is natively part of the environment of a Lambda function that runs in the AWS cloud\. If you want to use `boto3` in a Lambda function that's deployed to an AWS Greengrass core, make sure to include the AWS SDK in your package\. In addition, if you use both the AWS Greengrass Core SDK and the AWS SDK in the same package, make sure that your Lambda functions use the correct namespaces\. For more information about how to create your deployment package, see:  
[Creating a Deployment Package \(Python\)](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html)
[ Creating a Deployment Package \(Node\.js\)](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-create-deployment-pkg.html)
[ Creating a Deployment Package \(Java\)](https://docs.aws.amazon.com/lambda/latest/dg/lambda-java-how-to-create-deployment-package.html)

------
#### [ GGC v1\.1\.0 ]

AWS SDKs  
Using the AWS SDKs, you can build applications that work with any AWS service, including Amazon S3, Amazon DynamoDB, AWS IoT, AWS Greengrass, and more\. In the context of AWS Greengrass, you can use the AWS SDK inside deployed Lambda functions to make direct calls to any AWS service\.

AWS IoT Device SDKs  
The AWS IoT Device SDKs helps devices connect to AWS IoT or AWS Greengrass services\. Devices must know which AWS Greengrass group they belong to and the IP address of the AWS Greengrass core that they should connect to\.   
Although you can use any of the AWS IoT Device SDKs to connect to an AWS Greengrass core, only the C\+\+ and Python Device SDKs provide AWS Greengrass\-specific functionality, such as access to the AWS Greengrass Discovery Service and AWS Greengrass core root CA downloads\. For more information, see [AWS IoT Device SDK](https://aws.amazon.com/iot/sdk/)\.

AWS Greengrass Core SDK  
The AWS Greengrass Core SDK enables Lambda functions to interact with the AWS Greengrass core on which they run in order to publish messages, interact with the local Device Shadow service, or invoke other deployed Lambda functions\. This SDK is used exclusively for writing Lambda functions running in the Lambda runtime on an AWS Greengrass core\. Lambda functions running on an AWS Greengrass core can interact with AWS cloud services directly using the AWS SDK\. The AWS Greengrass Core SDK and the AWS SDK are contained in different packages, so you can use both packages simultaneously\. You can download the AWS Greengrass Core SDK from the **Software** page of the [AWS IoT console](https://console.aws.amazon.com/iot/home)\.  
The AWS Greengrass Core SDK follows the AWS SDK programming model\. It allows you to easily port Lambda functions developed for the cloud to Lambda functions that run on an AWS Greengrass core\. For example, using the AWS SDK, the following Lambda function publishes a message to the topic `"/some/topic"` in the cloud:   

```
import boto3
        
client = boto3.client('iot-data')
response = client.publish(
	topic = "/some/topic",
	qos = 0,
	payload = "Some payload".encode()
)
```
To port this Lambda function for execution on an AWS Greengrass core, replace the `import boto3` statement with the `import greengrasssdk`, as shown in the following snippet:  
The AWS Greengrass Core SDK only supports sending MQTT messages with QoS = 0\.

```
import greengrasssdk
                        
client = greengrasssdk.client('iot-data')
response = client.publish(
    topic='/some/topic',
    qos=0,
    payload='some payload'.encode()
)
```
This allows you to test your Lambda functions in the cloud and migrate them to AWS Greengrass with minimal effort\.

**Note**  
The AWS SDK is natively part of the environment of a Lambda function that runs in the AWS cloud\. If you want to use `boto3` in a Lambda function that's deployed to an AWS Greengrass core, make sure to include the AWS SDK in your package\. In addition, if you use both the AWS Greengrass Core SDK and the AWS SDK in the same package, make sure that your Lambda functions use the correct namespaces\. For more information about how to create your deployment package, see:  
[Creating a Deployment Package \(Python\)](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html)
[ Creating a Deployment Package \(Node\.js\)](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-create-deployment-pkg.html)
[ Creating a Deployment Package \(Java\)](https://docs.aws.amazon.com/lambda/latest/dg/lambda-java-how-to-create-deployment-package.html)

------
#### [ GGC v1\.0\.0 ]

AWS SDKs  
Using the AWS SDKs, you can build applications that work with any AWS service, including Amazon S3, Amazon DynamoDB, AWS IoT, AWS Greengrass, and more\. In the context of AWS Greengrass, you can use the AWS SDK inside deployed Lambda functions to make direct calls to any AWS service\.

AWS IoT Device SDKs  
The AWS IoT Device SDKs helps devices connect to AWS IoT or AWS Greengrass services\. Devices must know which AWS Greengrass group they belong to and the IP address of the AWS Greengrass core that they should connect to\.   
Although you can use any of the AWS IoT Device SDKs to connect to an AWS Greengrass core, only the C\+\+ and Python Device SDKs provide AWS Greengrass\-specific functionality, such as access to the AWS Greengrass Discovery Service and AWS Greengrass core root CA downloads\. For more information, see [AWS IoT Device SDK](https://aws.amazon.com/iot/sdk/)\.

AWS Greengrass Core SDK  
The AWS Greengrass Core SDK enables Lambda functions to interact with the AWS Greengrass core on which they run in order to publish messages, interact with the local Device Shadow service, or invoke other deployed Lambda functions\. This SDK is used exclusively for writing Lambda functions running in the Lambda runtime on an AWS Greengrass core\. Lambda functions running on an AWS Greengrass core can interact with AWS cloud services directly using the AWS SDK\. The AWS Greengrass Core SDK and the AWS SDK are contained in different packages, so you can use both packages simultaneously\. You can download the AWS Greengrass Core SDK from the **Software** page of the [AWS IoT console](https://console.aws.amazon.com/iot/home)\.  
The AWS Greengrass Core SDK follows the AWS SDK programming model\. It allows you to easily port Lambda functions developed for the cloud to Lambda functions that run on an AWS Greengrass core\. For example, using the AWS SDK, the following Lambda function publishes a message to the topic `"/some/topic"` in the cloud:   

```
import boto3
        
client = boto3.client('iot-data')
response = client.publish(
	topic = "/some/topic",
	qos = 0,
	payload = "Some payload".encode()
)
```
To port this Lambda function for execution on an AWS Greengrass core, replace the `import boto3` statement with the `import greengrasssdk`, as shown in the following snippet:  
The AWS Greengrass Core SDK only supports sending MQTT messages with QoS = 0\.

```
import greengrasssdk
                        
client = greengrasssdk.client('iot-data')
response = client.publish(
    topic='/some/topic',
    qos=0,
    payload='some payload'.encode()
)
```
This allows you to test your Lambda functions in the cloud and migrate them to AWS Greengrass with minimal effort\.

**Note**  
The AWS SDK is natively part of the environment of a Lambda function that runs in the AWS cloud\. If you want to use `boto3` in a Lambda function that's deployed to an AWS Greengrass core, make sure to include the AWS SDK in your package\. In addition, if you use both the AWS Greengrass Core SDK and the AWS SDK in the same package, make sure that your Lambda functions use the correct namespaces\. For more information about how to create your deployment package, see:  
[Creating a Deployment Package \(Python\)](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html)

------

## Supported Platforms and Requirements<a name="gg-platforms"></a>

The AWS Greengrass core software is supported on the platforms listed here, and requires a few dependencies\.

------
#### [ GGC v1\.6\.0 ]
+ Supported platforms:
  + Architecture: ARMv7l; OS: Linux; Distribution: [Raspbian Jessie, 2017\-03\-02](https://downloads.raspberrypi.org/raspbian/images/raspbian-2017-03-03/)
**Note**  
 You can choose to use **Raspbian Stretch** instead of Jessie, but this will not support [OTA Updates](core-ota-update.md)\. Using Stretch will also require additional memory set\-up\. For more information, see [this step](module1.md#stretch-step)\. 
  + Architecture: x86\_64; OS: Linux; Distribution: Amazon Linux \(amzn\-ami\-hvm\-2016\.09\.1\.20170119\-x86\_64\-ebs\)
  + Architecture: x86\_64; OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04
  + Architecture: ARMv8 \(AArch64\); OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04 \(Annapurna Alpine V2\)
+ The following items are required:
  + Minimum 128 MB RAM allocated to the AWS Greengrass core device\.
  + Linux kernel version 4\.4 or greater: while several versions may work with AWS Greengrass, for optimal security and performance, we recommend version 4\.4 or greater\.
  + [Glibc library](https://www.gnu.org/software/libc/) version 2\.14 or greater\.
  + The `/var/run` directory must be present on the device\.
  + AWS Greengrass requires hardlink and softlink protection to be enabled on the device\. Without this, AWS Greengrass can only be run in insecure mode, using the `-i` flag\.
  + The `ggc_user` and `ggc_group` user and group must be present on the device\.
  + The following Linux kernel configurations must be enabled on the device: 
    + Namespace: CONFIG\_IPC\_NS, CONFIG\_UTS\_NS, CONFIG\_USER\_NS, CONFIG\_PID\_NS
    + CGroups: CONFIG\_CGROUP\_DEVICE, CONFIG\_CGROUPS, CONFIG\_MEMCG
    + Others: CONFIG\_POSIX\_MQUEUE, CONFIG\_OVERLAY\_FS, CONFIG\_HAVE\_ARCH\_SECCOMP\_FILTER, CONFIG\_SECCOMP\_FILTER, CONFIG\_KEYS, CONFIG\_SECCOMP
  + `/dev/stdin`, `/dev/stdout`, and `/dev/stderr` must be enabled\.
  + The Linux kernel must support [cgroups](https://en.wikipedia.org/wiki/Cgroups)\.
  + The *memory* `cgroup` must be enabled and mounted to allow AWS Greengrass to set the memory limit for Lambda functions\.
  + The root certificate for Amazon S3 and AWS IoT must be present in the system trust store\.
+ The following items may be optional:
  + The *devices* `cgroup` must be enabled and mounted if Lambda functions with [Local Resource Access \(LRA\)](access-local-resources.md) are used to open files on the AWS Greengrass core device\.
  + [Python](https://www.python.org/) version 2\.7 is required if Python Lambda functions are used\. If so, ensure that it's added to your `PATH` environment variable\.
  + [NodeJS](https://www.nodejs.org/) version 6\.10 or greater is required if Node\.JS Lambda functions are used\. If so, ensure that it's added to your `PATH` environment variable\.
  + [ Java](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) version 8 or greater is required if Java Lambda functions are used\. If so, ensure that it's added to your `PATH` environment variable\.
  + [OpenSSL](https://www.openssl.org/) 1\.01 or greater is required for [Greengrass OTA Agent](core-ota-update.md#ota-agent) as well as the following commands: `wget`, `realpath`, `tar`, `readlink`, `basename`, `dirname`, `pidof`, `df`, `grep`, and `umount`\.

------
#### [ GGC v1\.5\.0 ]
+ Supported platforms:
  + Architecture: ARMv7l; OS: Linux; Distribution: [Raspbian Jessie, 2017\-03\-02](https://downloads.raspberrypi.org/raspbian/images/raspbian-2017-03-03/)
**Note**  
 You can choose to use **Raspbian Stretch** instead of Jessie, but this will not support [OTA Updates](core-ota-update.md)\. Using Stretch will also require additional memory set\-up\. For more information, see [this step](module1.md#stretch-step)\. 
  + Architecture: x86\_64; OS: Linux; Distribution: Amazon Linux \(amzn\-ami\-hvm\-2016\.09\.1\.20170119\-x86\_64\-ebs\)
  + Architecture: x86\_64; OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04
  + Architecture: ARMv8 \(AArch64\); OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04 \(Annapurna Alpine V2\)
+ The following items are required:
  + Minimum 128 MB RAM allocated to the AWS Greengrass core device\.
  + Linux kernel version 4\.4 or greater: while several versions may work with AWS Greengrass, for optimal security and performance, we recommend version 4\.4 or greater\.
  + [Glibc library](https://www.gnu.org/software/libc/) version 2\.14 or greater\.
  + The `/var/run` directory must be present on the device\.
  + AWS Greengrass requires hardlink and softlink protection to be enabled on the device\. Without this, AWS Greengrass can only be run in insecure mode, using the `-i` flag\.
  + The `ggc_user` and `ggc_group` user and group must be present on the device\.
  + The following Linux kernel configurations must be enabled on the device: 
    + Namespace: CONFIG\_IPC\_NS, CONFIG\_UTS\_NS, CONFIG\_USER\_NS, CONFIG\_PID\_NS
    + CGroups: CONFIG\_CGROUP\_DEVICE, CONFIG\_CGROUPS, CONFIG\_MEMCG
    + Others: CONFIG\_POSIX\_MQUEUE, CONFIG\_OVERLAY\_FS, CONFIG\_HAVE\_ARCH\_SECCOMP\_FILTER, CONFIG\_SECCOMP\_FILTER, CONFIG\_KEYS, CONFIG\_SECCOMP
  + `/dev/stdin`, `/dev/stdout`, and `/dev/stderr` must be enabled\.
  + The Linux kernel must support [cgroups](https://en.wikipedia.org/wiki/Cgroups)\.
  + The *memory* `cgroup` must be enabled and mounted to allow AWS Greengrass to set the memory limit for Lambda functions\.
  + The root certificate for Amazon S3 and AWS IoT must be present in the system trust store\.
+ The following items may be optional:
  + The *devices* `cgroup` must be enabled and mounted if Lambda functions with [Local Resource Access \(LRA\)](access-local-resources.md) are used to open files on the AWS Greengrass core device\.
  + [Python](https://www.python.org/) version 2\.7 is required if Python Lambda functions are used\. If so, ensure that it's added to your `PATH` environment variable\.
  + [NodeJS](https://www.nodejs.org/) version 6\.10 or greater is required if Node\.JS Lambda functions are used\. If so, ensure that it's added to your `PATH` environment variable\.
  + [ Java](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) version 8 or greater is required if Java Lambda functions are used\. If so, ensure that it's added to your `PATH` environment variable\.
  + [OpenSSL](https://www.openssl.org/) 1\.01 or greater is required for [Greengrass OTA Agent](core-ota-update.md#ota-agent) as well as the following commands: `wget`, `realpath`, `tar`, `readlink`, `basename`, `dirname`, `pidof`, `df`, `grep`, and `umount`\.

------
#### [ GGC v1\.3\.0 ]
+ Supported platforms:
  + Architecture: ARMv7l; OS: Linux; Distribution: [Raspbian Jessie, 2017\-03\-02](https://downloads.raspberrypi.org/raspbian/images/raspbian-2017-03-03/)
  + Architecture: x86\_64; OS: Linux; Distribution: Amazon Linux \(amzn\-ami\-hvm\-2016\.09\.1\.20170119\-x86\_64\-ebs\)
  + Architecture: x86\_64; OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04
  + Architecture: ARMv8 \(AArch64\); OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04 \(Annapurna Alpine V2\)
+ The following items are required:
  + Minimum 128 MB RAM allocated to the AWS Greengrass core device\.
  + Linux kernel version 4\.4 or later\. Although several versions might work with AWS Greengrass, for optimal security and performance, we recommend version 4\.4 or later\.
  + [Glibc library](https://www.gnu.org/software/libc/) version 2\.14 or later\.
  + The `/var/run` directory must be present on the device\.
  + AWS Greengrass requires hardlink and softlink protection to be enabled on the device\. Without this, AWS Greengrass can only be run in insecure mode, using the `-i` flag\.
  + The `ggc_user` and `ggc_group` user and group must be present on the device\.
  + The following Linux kernel configurations must be enabled on the device: 
    + Namespace: CONFIG\_IPC\_NS, CONFIG\_UTS\_NS, CONFIG\_USER\_NS, CONFIG\_PID\_NS
    + CGroups: CONFIG\_CGROUP\_DEVICE, CONFIG\_CGROUPS, CONFIG\_MEMCG
    + Others: CONFIG\_POSIX\_MQUEUE, CONFIG\_OVERLAY\_FS, CONFIG\_HAVE\_ARCH\_SECCOMP\_FILTER, CONFIG\_SECCOMP\_FILTER, CONFIG\_KEYS, CONFIG\_SECCOMP
  +  The [https://www.sqlite.org/](https://www.sqlite.org/) package is required for AWS IoT device shadows\. Make sure it’s added to your `PATH` environment variable\.
  + `/dev/stdin`, `/dev/stdout`, and `/dev/stderr` must be enabled\.
  + The Linux kernel must support [cgroups](https://en.wikipedia.org/wiki/Cgroups)\.
  + The *memory* `cgroup` must be enabled and mounted to allow AWS Greengrass to set the memory limit for Lambda functions\.
  + The root certificate for Amazon S3 and AWS IoT must be present in the system trust store\.
+ The following items might be optional:
  + The *devices* `cgroup` must be enabled and mounted if Lambda functions with [local resource access \(LRA\)](access-local-resources.md) are used to open files on the AWS Greengrass core device\.
  + [Python](https://www.python.org/) version 2\.7 is required if Python Lambda functions are used\. If so, make sure that it's added to your `PATH` environment variable\.
  + [Node\.js](https://www.nodejs.org/) version 6\.10 or later is required if Node\.js Lambda functions are used\. If so, make sure that it's added to your `PATH` environment variable\.
  + [ Java](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) version 8 or later is required if Java Lambda functions are used\. If so, make sure that it's added to your `PATH` environment variable\.
  + [OpenSSL](https://www.openssl.org/) 1\.01 or later is required for [Greengrass OTA Agent](core-ota-update.md#ota-agent) as are the following commands: `wget`, `realpath`, `tar`, `readlink`, `basename`, `dirname`, `pidof`, `df`, `grep`, and `umount`\.

------
#### [ GGC v1\.1\.0 ]
+ Supported platforms:
  + Architecture: ARMv7l; OS: Linux; Distribution: [Raspbian Jessie, 2017\-03\-02](https://downloads.raspberrypi.org/raspbian/images/raspbian-2017-03-03/)
  + Architecture: x86\_64; OS: Linux; Distribution: Amazon Linux \(amzn\-ami\-hvm\-2016\.09\.1\.20170119\-x86\_64\-ebs\)
  + Architecture: x86\_64; OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04
  + Architecture: ARMv8 \(AArch64\); OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04 \(Annapurna Alpine V2\)
+ The following items are required:
  + Minimum 128 MB RAM allocated to the AWS Greengrass core device\.
  + Linux kernel version 4\.4 or later\. Although several versions might work with AWS Greengrass, for optimal security and performance, we recommend version 4\.4 or later\.
  + [Glibc library](https://www.gnu.org/software/libc/) version 2\.14 or later\.
  + The `/var/run` directory must be present on the device\.
  + AWS Greengrass requires hardlink and softlink protection to be enabled on the device\. Without this, AWS Greengrass can only be run in insecure mode, using the `-i` flag\.
  + The `ggc_user` and `ggc_group` user and group must be present on the device\.
  + The following Linux kernel configurations must be enabled on the device: 
    + Namespace: CONFIG\_IPC\_NS, CONFIG\_UTS\_NS, CONFIG\_USER\_NS, CONFIG\_PID\_NS
    + CGroups: CONFIG\_CGROUP\_DEVICE, CONFIG\_CGROUPS, CONFIG\_MEMCG
    + Others: CONFIG\_POSIX\_MQUEUE, CONFIG\_OVERLAY\_FS, CONFIG\_HAVE\_ARCH\_SECCOMP\_FILTER, CONFIG\_SECCOMP\_FILTER, CONFIG\_KEYS, CONFIG\_SECCOMP
  +  The [https://www.sqlite.org/](https://www.sqlite.org/) package is required for AWS IoT device shadows\. Make sure it’s added to your `PATH` environment variable\.
  + `/dev/stdin`, `/dev/stdout`, and `/dev/stderr` must be enabled\.
  + The Linux kernel must support [cgroups](https://en.wikipedia.org/wiki/Cgroups)\.
  + The *memory* `cgroup` must be enabled and mounted to allow AWS Greengrass to set the memory limit for Lambda functions\.
  + The root certificate for Amazon S3 and AWS IoT must be present in the system trust store\.
+ The following items might be optional:
  + [Python](https://www.python.org/) version 2\.7 is required if Python Lambda functions are used\. If so, make sure that it's added to your `PATH` environment variable\.
  + [Node\.js](https://www.nodejs.org/) version 6\.10 or later is required if Node\.js Lambda functions are used\. If so, make sure that it's added to your `PATH` environment variable\.
  + [ Java](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) version 8 or later is required if Java Lambda functions are used\. If so, make sure that it's added to your `PATH` environment variable\.

------
#### [ GGC v1\.0\.0 ]
+ Supported platforms:
  + Architecture: ARMv7l; OS: Linux; Distribution: [Raspbian Jessie, 2017\-03\-02](https://downloads.raspberrypi.org/raspbian/images/raspbian-2017-03-03/)
  + Architecture: x86\_64; OS: Linux; Distribution: Amazon Linux \(amzn\-ami\-hvm\-2016\.09\.1\.20170119\-x86\_64\-ebs\)
  + Architecture: x86\_64; OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04
  + Architecture: ARMv8 \(AArch64\); OS: Linux; Distribution: Ubuntu 14\.04 – 16\.04 \(Annapurna Alpine V2\)
+ The following items are required:
  + Minimum 128 MB RAM allocated to the AWS Greengrass core device\.
  + Linux kernel version 4\.4 or later\. Although several versions might work with AWS Greengrass, for optimal security and performance, we recommend version 4\.4 or later\.
  + [Glibc library](https://www.gnu.org/software/libc/) version 2\.14 or later\.
  + The `/var/run` directory must be present on the device\.
  + AWS Greengrass requires hardlink and softlink protection to be enabled on the device\. Without this, AWS Greengrass can only be run in insecure mode, using the `-i` flag\.
  + The `ggc_user` and `ggc_group` user and group must be present on the device\.
  + The following Linux kernel configurations must be enabled on the device: 
    + Namespace: CONFIG\_IPC\_NS, CONFIG\_UTS\_NS, CONFIG\_USER\_NS, CONFIG\_PID\_NS
    + CGroups: CONFIG\_CGROUP\_DEVICE, CONFIG\_CGROUPS, CONFIG\_MEMCG
    + Others: CONFIG\_POSIX\_MQUEUE, CONFIG\_OVERLAY\_FS, CONFIG\_HAVE\_ARCH\_SECCOMP\_FILTER, CONFIG\_SECCOMP\_FILTER, CONFIG\_KEYS, CONFIG\_SECCOMP
  +  The [https://www.sqlite.org/](https://www.sqlite.org/) package is required for AWS IoT device shadows\. Make sure it’s added to your `PATH` environment variable\.
  + `/dev/stdin`, `/dev/stdout`, and `/dev/stderr` must be enabled\.
  + The Linux kernel must support [cgroups](https://en.wikipedia.org/wiki/Cgroups)\.
  + The *memory* `cgroup` must be enabled and mounted to allow AWS Greengrass to set the memory limit for Lambda functions\.
  + The root certificate for Amazon S3 and AWS IoT must be present in the system trust store\.
+ The following items might be optional:
  + [Python](https://www.python.org/) version 2\.7 is required if Python Lambda functions are used\. If so, make sure that it's added to your `PATH` environment variable\.

------

## We Want to Hear from You<a name="contact-us"></a>

We welcome your feedback\. To contact us, visit the [AWS Greengrass Forum](https://forums.aws.amazon.com/forum.jspa?forumID=254)\.