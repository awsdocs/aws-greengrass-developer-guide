# Integrate with services and protocols using Greengrass connectors<a name="connectors"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

Greengrass connectors are prebuilt modules that help accelerate the development lifecycle for common edge scenarios\. They make it easier to interact with local infrastructure, device protocols, AWS, and other cloud services\. With connectors, you can spend less time learning new protocols and APIs and more time focusing on the logic that matters to your business\.

The following diagram shows where connectors can fit into the AWS IoT Greengrass landscape\.

![\[Connectors connect to devices, services, and local resources.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/connectors/connectors-arch.png)

Many connectors use MQTT messages to communicate with devices and Greengrass Lambda functions in the group, or with AWS IoT and the local shadow service\. In the following example, the Twilio Notifications connector receives MQTT messages from a user\-defined Lambda function, uses a local reference of a secret from AWS Secrets Manager, and calls the Twilio API\.

![\[A connector receiving an MQTT message from a Lambda function and calling a service.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/connectors/twilio-solution.png)

For tutorials that create this solution, see [Getting started with Greengrass connectors \(console\)](connectors-console.md) and [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)\.

Greengrass connectors can help you quickly extend device capabilities or create single\-purpose devices\. Connectors can make it easier to:
+ Implement reusable business logic\.
+ Interact with cloud and local services, including AWS and third\-party services\.
+ Ingest and process device data\.
+ Enable device\-to\-device calls using MQTT topic subscriptions and user\-defined Lambda functions\.

AWS provides a set of Greengrass connectors that simplify interactions with common services and data sources\. These prebuilt modules enable scenarios for logging and diagnostics, replenishment, industrial data processing, and alarm and messaging\. For more information, see [AWS\-provided Greengrass connectors](connectors-list.md)\.

## Requirements<a name="connectors-reqs"></a>

The following requirements apply for connectors:
+ You must meet the requirements of each connector that you're using\. These requirements might include the minimum AWS IoT Greengrass Core software version, device prerequisites, required permissions, and limits\. For more information, see [AWS\-provided Greengrass connectors](connectors-list.md)\.
+ A Greengrass group can contain only one configured instance of a given connector, but the instance can be used in multiple subscriptions\. For more information, see [Configuration parameters](#connectors-parameters)\.
+ When the [default containerization](lambda-group-config.md#lambda-containerization-groupsettings) for the Greengrass group is set to **No container**, the connectors in the group must run without containerization\. To find connectors that support **No container** mode, see [AWS\-provided Greengrass connectors](connectors-list.md)\.

## Using Greengrass connectors<a name="use-applications"></a>

A connector is a type of group component\. Like other group components, such as devices and user\-defined Lambda functions, you add connectors to groups, configure their settings, and deploy them to the AWS IoT Greengrass core\. Connectors run in the core environment\.

Some connectors can be deployed as simple standalone applications\. For example, the Device Defender connector reads system metrics from the core device and sends them to AWS IoT Device Defender for analysis\.

Other connectors can be used as building blocks in larger solutions\. The following example solution uses the Modbus\-RTU Protocol Adapter connector to process messages from sensors and the Twilio Notifications connector to trigger Twilio messages\.

![\[Data flow from Lambda function to Modbus-RTU Protocol Adapter connector to Lambda function to Twilio Notifications connector to Twilio.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/connectors/modbus-twilio-solution.png)

Solutions often include user\-defined Lambda functions that sit next to connectors and process the data that the connector sends or receives\. In this example, the TempMonitor function receives data from Modbus\-RTU Protocol Adapter, runs some business logic, and then sends data to Twilio Notifications\.

To create and deploy a solution, you follow this general process:

1. Map out the high\-level data flow\. Identify the data sources, data channels, services, protocols, and resources that you need to work with\. In the example solution, this includes data over the Modbus RTU protocol, the physical Modbus serial port, and Twilio\.

1. Identify the connectors to include in the solution, and add them to your group\. The example solution uses Modbus\-RTU Protocol Adapter and Twilio Notifications\. To help you find connectors that apply to your scenario, and to learn about their individual requirements, see [AWS\-provided Greengrass connectors](connectors-list.md)\.

1. Identify whether user\-defined Lambda functions, devices, or resources are needed, and then create and add them to the group\. This might include functions that contain business logic or process data into a format required by another entity in the solution\. The example solution uses functions to send Modbus RTU requests and trigger Twilio notifications\. It also includes a local device resource for the Modbus RTU serial port and a secret resource for the Twilio authentication token\.
**Note**  
Secret resources reference passwords, tokens, and other secrets from AWS Secrets Manager\. Secrets can be used by connectors and Lambda functions to authenticate with services and applications\. By default, AWS IoT Greengrass can access secrets with names that start with "*greengrass\-*"\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

1. Create subscriptions that allow the entities in the solution to exchange MQTT messages\. If a connector is used in a subscription, the connector and the message source or target must use the predefined topic syntax supported by the connector\. For more information, see [Inputs and outputs](#connectors-inputs-outputs)\.

1. Deploy the group to the Greengrass core\.

To learn how to create and deploy a connector, see the following tutorials:
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)

## Configuration parameters<a name="connectors-parameters"></a>

Many connectors provide parameters that let you customize the behavior or output\. These parameters are used during initialization, at runtime, or at other times in the connector lifecycle\.

Parameter types and usage vary by connector\. For example, the SNS connector has a parameter that configures the default SNS topic, and Device Defender has a parameter that configures the data sampling rate\.

A group version can contain multiple connectors, but only one instance of a given connector at a time\. This means that each connector in the group can have only one active configuration\. However, the connector instance can be used in multiple subscriptions in the group\. For example, you can create subscriptions that allow many devices to send data to the Kinesis Firehose connector\.

### Parameters used to access group resources<a name="connectors-parameters-resources"></a>

Greengrass connectors use group resources to access the file system, ports, peripherals, and other local resources on the core device\. If a connector requires access to a group resource, then it provides related configuration parameters\.

Group resources include:
+ [Local resources](access-local-resources.md)\. Directories, files, ports, pins, and peripherals that are present on the Greengrass core device\.
+ [Machine learning resources](ml-inference.md)\. Machine learning models that are trained in the cloud and deployed to the core for local inference\.
+ [Secret resources](secrets.md)\. Local, encrypted copies of passwords, keys, tokens, or arbitrary text from AWS Secrets Manager\. Connectors can securely access these local secrets and use them to authenticate to services or local infrastructure\.

For example, parameters for Device Defender enable access to system metrics in the host `/proc` directory, and parameters for Twilio Notifications enable access to a locally stored Twilio authentication token\.

### Updating connector parameters<a name="update-application-parameters-"></a>

Parameters are configured when the connector is added to a Greengrass group\. You can change parameter values after the connector is added\.
+ In the console: From the group configuration page, open **Connectors**, and from the connector's contextual menu, choose **Edit**\.
**Note**  
If the connector uses a secret resource that's later changed to reference a different secret, you must edit the connector's parameters and confirm the change\.
+ In the API: Create another version of the connector that defines the new configuration\.

  The AWS IoT Greengrass API uses versions to manage groups\. Versions are immutable, so to add or change group components—for example, the group's devices, functions, and resources—you must create versions of new or updated components\. Then, you create and deploy a group version that contains the target version of each component\.

After you make changes to the connector configuration, you must deploy the group to propagate the changes to the core\.

## Inputs and outputs<a name="connectors-inputs-outputs"></a>

Many Greengrass connectors can communicate with other entities by sending and receiving MQTT messages\. MQTT communication is controlled by subscriptions that allow a connector to exchange data with Lambda functions, devices, and other connectors in the Greengrass group, or with AWS IoT and the local shadow service\. To allow this communication, you must create subscriptions in the group that the connector belongs to\. For more information, see [Managed subscriptions in the MQTT messaging workflow](gg-sec.md#gg-msg-workflow)\.

Connectors can be message publishers, message subscribers, or both\. Each connector defines the MQTT topics that it publishes or subscribes to\. These predefined topics must be used in the subscriptions where the connector is a message source or message target\. For tutorials that include steps for configuring subscriptions for a connector, see [Getting started with Greengrass connectors \(console\)](connectors-console.md) and [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)\.

**Note**  
Many connectors also have built\-in modes of communication to interact with cloud or local services\. These vary by connector and might require that you configure parameters or add permissions to the [group role](group-role.md)\. For information about connector requirements, see [AWS\-provided Greengrass connectors](connectors-list.md)\.

### Input topics<a name="connectors-multiple-topics"></a>

Most connectors receive input data on MQTT topics\. Some connectors subscribe to multiple topics for input data\. For example, the Serial Stream connector supports two topics:
+ `serial/+/read/#`
+ `serial/+/write/#`

For this connector, read and write requests are sent to the corresponding topic\. When you create subscriptions, make sure to use the topic that aligns with your implementation\.

The `+` and `#` characters in the previous examples are wildcards\. These wildcards allow subscribers to receive messages on multiple topics and publishers to customize the topics that they publish to\.
+ The `+` wildcard can appear anywhere in the topic hierarchy\. It can be replaced by one hierarchy item\.

  As an example, for topic `sensor/+/input`, messages can be published to topics `sensor/id-123/input` but not to `sensor/group-a/id-123/input`\.
+ The `#` wildcard can appear only at the end of the topic hierarchy\. It can be replaced by zero or more hierarchy items\.

  As an example, for topic `sensor/#`, messages can be published to `sensor/`, `sensor/id-123`, and `sensor/group-a/id-123`, but not to `sensor`\.

Wildcard characters are valid only when subscribing to topics\. Messages can't be published to topics that contain wildcards\. Check the documentation for the connector to learn about its input or output topic requirements\. For more information, see [AWS\-provided Greengrass connectors](connectors-list.md)\.

## Containerization support<a name="connector-containerization"></a>

By default, most connectors run on the Greengrass core in an isolated runtime environment that's managed by AWS IoT Greengrass\. These runtime environments, called *containers*, provide isolation between connectors and the host system, which offers more security for the host and the connector\.

However, this Greengrass containerization isn't supported in some environments, such as when you run AWS IoT Greengrass in a Docker container or on older Linux kernels without cgroups\. In these environments, the connectors must run in **No container** mode\. To find connectors that support **No container** mode, see [AWS\-provided Greengrass connectors](connectors-list.md)\. Some connectors run in this mode natively, and some connectors allow you to set the isolation mode\.

You can also set the isolation mode to **No container** in environments that support Greengrass containerization, but we recommend using **Greengrass container** mode when possible\.

**Note**  
The [default containerization](lambda-group-config.md#lambda-containerization-groupsettings) setting for the Greengrass group doesn't apply to connectors\.

## Upgrading connector versions<a name="upgrade-connector-versions"></a>

Connector providers might release new versions of a connector that add features, fix issues, or improve performance\. For information about available versions and related changes, see the [documentation for each connector](connectors-list.md)\.

In the AWS IoT console, you can check for new versions for the connectors in your Greengrass group\.

1. Choose **Greengrass**, **Groups**, and then choose your group\.

1. Choose **Connectors** to display the connectors in the group\.

   If the connector has a new version, an **Available** button appears in the **Upgrade** column\.

1. To upgrade the connector version:

   1. On the **Connectors** page, in the **Upgrade** column, choose **Available**\. The **Upgrade connector** page opens and displays the current parameter settings, where applicable\.

      Choose the new connector version, define parameters as needed, and then choose **Upgrade**\.

   1. On the **Subscriptions** page, add new subscriptions in the group to replace any that use the connector as a source or target\. Then, remove the old subscriptions\.

      Subscriptions reference connectors by version, so they become invalid if you change the connector version in the group\.

   1. From the **Actions** menu, choose **Deploy** to deploy your changes to the core\.

To upgrade a connector from the AWS IoT Greengrass API, create and deploy a group version that includes the updated connector and subscriptions\. Use the same process as when you add a connector to a group\. For detailed steps that show you how to use the AWS CLI to configure and deploy an example Twilio Notifications connector, see [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)\.

## Logging for connectors<a name="connectors-logging"></a>

Greengrass connectors contain Lambda functions that write events and errors to Greengrass logs\. Depending on your group settings, logs are written to CloudWatch Logs, the local file system, or both\. Logs from connectors include the ARN of the corresponding function\. The following example ARN is from the Kinesis Firehose connector:

```
arn:aws:lambda:aws-region:account-id:function:KinesisFirehoseClient:1
```

The default logging configuration writes info\-level logs to the file system using the following directory structure:

```
greengrass-root/ggc/var/log/user/region/aws/function-name.log
```

For more information about Greengrass logging, see [Monitoring with AWS IoT Greengrass logs](greengrass-logs-overview.md)\.