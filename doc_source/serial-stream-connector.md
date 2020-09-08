# Serial Stream connector<a name="serial-stream-connector"></a>

The Serial Stream [connector](connectors.md) reads and writes to a serial port on an AWS IoT Greengrass core device\.

This connector supports two modes of operation:
+ **Read\-On\-Demand**\. Receives read and write requests on MQTT topics and publishes the response of the read operation or the status of the write operation\.
+ **Polling\-Read**\. Reads from the serial port at regular intervals\. This mode also supports Read\-On\-Demand requests\.

**Note**  
Read requests are limited to a maximum read length of 63994 bytes\. Write requests are limited to a maximum data length of 128000 bytes\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 3 | `arn:aws:greengrass:region::/connectors/SerialStream/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/SerialStream/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/SerialStream/versions/1` | 

For information about version changes, see the [Changelog](#serial-stream-connector-changelog)\.

## Requirements<a name="serial-stream-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 3 ]
+ <a name="conn-req-ggc-v1.9.3"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-serial-stream-req-serial-port-resource"></a>A [local device resource](access-local-resources.md) in the Greengrass group that points to the target serial port\.
**Note**  
Before you deploy this connector, we recommend that you set up the serial port and verify that you can read and write to it\. 

------
#### [ Versions 1 \- 2 ]
+ <a name="conn-req-ggc-v1.7.0"></a>AWS IoT Greengrass Core software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-serial-stream-req-serial-port-resource"></a>A [local device resource](access-local-resources.md) in the Greengrass group that points to the target serial port\.
**Note**  
Before you deploy this connector, we recommend that you set up the serial port and verify that you can read and write to it\. 

------

## Connector Parameters<a name="serial-stream-connector-param"></a>

This connector provides the following parameters:

`BaudRate`  
The baud rate of the serial connection\.  
Display name in the AWS IoT console: **Baud rate**  
Required: `true`  
Type: `string`  
Valid values: `110, 300, 600, 1200, 2400, 4800, 9600, 14400, 19200, 28800, 38400, 56000, 57600, 115200, 230400`  
Valid pattern: `^110$|^300$|^600$|^1200$|^2400$|^4800$|^9600$|^14400$|^19200$|^28800$|^38400$|^56000$|^57600$|^115200$|^230400$`

`Timeout`  
The timeout \(in seconds\) for a read operation\.  
Display name in the AWS IoT console: **Timeout**  
Required: `true`  
Type: `string`  
Valid values: `1 - 59`  
Valid pattern: `^([1-9]|[1-5][0-9])$`

`SerialPort`  
The absolute path to the physical serial port on the device\. This is the source path that's specified for the local device resource\.  
Display name in the AWS IoT console: **Serial port**  
Required: `true`  
Type: `string`  
Valid pattern: `[/a-zA-Z0-9_-]+`

`SerialPort-ResourceId`  
The ID of the local device resource that represents the physical serial port\.  
This connector is granted read\-write access to the resource\.
Display name in the AWS IoT console: **Serial port resource**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9_-]+`

`PollingRead`  
Sets the read mode: Polling\-Read or Read\-On\-Demand\.  
+ For Polling\-Read mode, specify `true`\. In this mode, the `PollingInterval`, `PollingReadType`, and `PollingReadLength` properties are required\.
+ For Read\-On\-Demand mode, specify `false`\. In this mode, the type and length values are specified in the read request\.
Display name in the AWS IoT console: **Read mode**  
Required: `true`  
Type: `string`  
Valid values: `true, false`  
Valid pattern: `^([Tt][Rr][Uu][Ee]|[Ff][Aa][Ll][Ss][Ee])$`

`PollingReadLength`  
The length of data \(in bytes\) to read in each polling read operation\. This applies only when using Polling\-Read mode\.  
Display name in the AWS IoT console: **Polling read length**  
Required: `false`\. This property is required when `PollingRead` is `true`\.  
Type: `string`  
Valid pattern: `^(|[1-9][0-9]{0,3}|[1-5][0-9]{4}|6[0-2][0-9]{3}|63[0-8][0-9]{2}|639[0-8][0-9]|6399[0-4])$`

`PollingReadInterval`  
The interval \(in seconds\) at which the polling read takes place\. This applies only when using Polling\-Read mode\.  
Display name in the AWS IoT console: **Polling read interval**  
Required: `false`\. This property is required when `PollingRead` is `true`\.  
Type: `string`  
Valid values: 1 \- 999  
Valid pattern: `^(|[1-9]|[1-9][0-9]|[1-9][0-9][0-9])$`

`PollingReadType`  
The type of data that the polling thread reads\. This applies only when using Polling\-Read mode\.  
Display name in the AWS IoT console: **Polling read type**  
Required: `false`\. This property is required when `PollingRead` is `true`\.  
Type: `string`  
Valid values: `ascii, hex`  
Valid pattern: `^(|[Aa][Ss][Cc][Ii][Ii]|[Hh][Ee][Xx])$`

`RtsCts`  
Indicates whether to enable the RTS/CTS flow control\. The default value is `false`\. For more information, see [RTS, CTS, and RTR](https://en.wikipedia.org/wiki/RS-232#RTS,_CTS,_and_RTR)\.   
Display name in the AWS IoT console: **RTS/CTS flow control**  
Required: `false`  
Type: `string`  
Valid values: `true, false`  
Valid pattern: `^(|[Tt][Rr][Uu][Ee]|[Ff][Aa][Ll][Ss][Ee])$`

`XonXoff`  
Indicates whether to enable the software flow control\. The default value is `false`\. For more information, see [Software flow control](https://en.wikipedia.org/wiki/Software_flow_control)\.  
Display name in the AWS IoT console: **Software flow control**  
Required: `false`  
Type: `string`  
Valid values: `true, false`  
Valid pattern: `^(|[Tt][Rr][Uu][Ee]|[Ff][Aa][Ll][Ss][Ee])$`

`Parity`  
The parity of the serial port\. The default value is `N`\. For more information, see [Parity](https://en.wikipedia.org/wiki/Serial_port#Parity)\.   
Display name in the AWS IoT console: **Serial port parity**  
Required: `false`  
Type: `string`  
Valid values: `N, E, O, S, M`  
Valid pattern: `^(|[NEOSMneosm])$`

### Create Connector Example \(AWS CLI\)<a name="serial-stream-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the Serial Stream connector\. It configures the connector for Polling\-Read mode\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MySerialStreamConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/SerialStream/versions/3",
            "Parameters": {
                "BaudRate" : "9600",
                "Timeout" : "25",
                "SerialPort" : "/dev/serial1",
                "SerialPort-ResourceId" : "my-serial-port-resource",
                "PollingRead" : "true",
                "PollingReadLength" : "30",
                "PollingReadInterval" : "30",
                "PollingReadType" : "hex"
            }
        }
    ]
}'
```

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="serial-stream-connector-data-input"></a>

This connector accepts read or write requests for serial ports on two MQTT topics\. Input messages must be in JSON format\.
+ Read requests on the `serial/+/read/#` topic\.
+ Write requests on the `serial/+/write/#` topic\.

To publish to these topics, replace the `+` wildcard with the core thing name and `#` wildcard with the path to the serial port\. For example:

```
serial/core-thing-name/read/dev/serial-port
```

**Topic filter:** `serial/+/read/#`  
Use this topic to send on\-demand read requests to a serial pin\. Read requests are limited to a maximum read length of 63994 bytes\.    
**Message properties**    
`readLength`  
The length of data to read from the serial port\.  
Required: `true`  
Type: `string`  
Valid pattern: `^[1-9][0-9]*$`  
`type`  
The type of data to read\.  
Required: `true`  
Type: `string`  
Valid values: `ascii, hex`  
Valid pattern: `(?i)^(ascii|hex)$`  
`id`  
An arbitrary ID for the request\. This property is used to map an input request to an output response\.  
Required: `false`  
Type: `string`  
Valid pattern: `.+`  
**Example input**  

```
{
    "readLength": "30",
    "type": "ascii",
    "id": "abc123"
}
```

**Topic filter:** `serial/+/write/#`  
Use this topic to send write requests to a serial pin\. Write requests are limited to a maximum data length of 128000 bytes\.    
**Message properties**    
`data`  
The string to write to the serial port\.  
Required: `true`  
Type: `string`  
Valid pattern: `^[1-9][0-9]*$`  
`type`  
The type of data to read\.  
Required: `true`  
Type: `string`  
Valid values: `ascii, hex`  
Valid pattern: `^(ascii|hex|ASCII|HEX)$`  
`id`  
An arbitrary ID for the request\. This property is used to map an input request to an output response\.  
Required: `false`  
Type: `string`  
Valid pattern: `.+`  
**Example input: ASCII request**  

```
{
    "data": "random serial data",
    "type": "ascii",
    "id": "abc123"
}
```  
**Example input: hex request**  

```
{
    "data": "base64 encoded data",
    "type": "hex",
    "id": "abc123"
}
```

## Output data<a name="serial-stream-connector-data-output"></a>

The connector publishes output data on two topics:
+ Status information from the connector on the `serial/+/status/#` topic\.
+ Responses from read requests on the `serial/+/read_response/#` topic\.

When publishing to this topic, the connector replaces the `+` wildcard with the core thing name and `#` wildcard with the path to the serial port\. For example:

```
serial/core-thing-name/status/dev/serial-port
```

**Topic filter:** `serial/+/status/#`  
Use this topic to listen for the status of read and write requests\. If an `id` property is included it the request, it's returned in the response\.    
**Example output: Success**  

```
{
    "response": {
        "status": "success"
    },
    "id": "abc123"
}
```  
**Example output: Failure**  
A failure response includes an `error_message` property that describes the error or timeout encountered while performing the read or write operation\.  

```
{
    "response": {
        "status": "fail",
        "error_message": "Could not write to port"
    },
    "id": "abc123"
}
```

**Topic filter:** `serial/+/read_response/#`  
Use this topic to receive response data from a read operation\. The response data is Base64 encoded if the type is `hex`\.    
**Example output**  

```
{
    "data": "output of serial read operation"
    "id": "abc123"
}
```

## Usage Example<a name="serial-stream-connector-usage"></a>

<a name="connectors-setup-intro"></a>Use the following high\-level steps to set up an example Python 3\.7 Lambda function that you can use to try out the connector\.

**Note**  <a name="connectors-setup-get-started-topics"></a>
The [Get started with connectors \(console\)](connectors-console.md) and [Get started with connectors \(CLI\)](connectors-cli.md) topics contain detailed steps that show you how to configure and deploy an example Twilio Notifications connector\.

1. Make sure you meet the [requirements](#serial-stream-connector-req) for the connector\.

1. <a name="connectors-setup-function"></a>Create and publish a Lambda function that sends input data to the connector\.

   Save the [example code](#serial-stream-connector-usage-example) as a PY file\. <a name="connectors-setup-function-sdk"></a>Download and unzip the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core)\. Then, create a zip package that contains the PY file and the `greengrasssdk` folder at the root level\. This zip package is the deployment package that you upload to AWS Lambda\.

   <a name="connectors-setup-function-publish"></a>After you create the Python 3\.7 Lambda function, publish a function version and create an alias\.

1. Configure your Greengrass group\.

   1. <a name="connectors-setup-gg-function"></a>Add the Lambda function by its alias \(recommended\)\. Configure the Lambda lifecycle as long\-lived \(or `"Pinned": true` in the CLI\)\.

   1. <a name="connectors-setup-device-resource"></a>Add the required local device resource and grant read/write access to the Lambda function\.

   1. Add the connector to your group and configure its [parameters](#serial-stream-connector-param)\.

   1. Add subscriptions to the group that allow the connector to receive [input data](#serial-stream-connector-data-input) and send [output data](#serial-stream-connector-data-output) on supported topic filters\.
      + <a name="connectors-setup-subscription-input-data"></a>Set the Lambda function as the source, the connector as the target, and use a supported input topic filter\.
      + <a name="connectors-setup-subscription-output-data"></a>Set the connector as the source, AWS IoT Core as the target, and use a supported output topic filter\. You use this subscription to view status messages in the AWS IoT console\.

1. <a name="connectors-setup-deploy-group"></a>Deploy the group\.

1. <a name="connectors-setup-test-sub"></a>In the AWS IoT console, on the **Test** page, subscribe to the output data topic to view status messages from the connector\. The example Lambda function is long\-lived and starts sending messages immediately after the group is deployed\.

   When you're finished testing, you can set the Lambda lifecycle to on\-demand \(or `"Pinned": false` in the CLI\) and deploy the group\. This stops the function from sending messages\.

### Example<a name="serial-stream-connector-usage-example"></a>

The following example Lambda function sends an input message to the connector\.

```
import greengrasssdk
import json

TOPIC_REQUEST = 'serial/CORE_THING_NAME/write/dev/serial1'

# Creating a greengrass core sdk client
iot_client = greengrasssdk.client('iot-data')

def create_serial_stream_request():
	request = {
		"data": "TEST",
		"type": "ascii",
		"id": "abc123"
	}
	return request

def publish_basic_request():
	iot_client.publish(payload=json.dumps(create_serial_stream_request()), topic=TOPIC_REQUEST)

publish_basic_request()

def lambda_handler(event, context):
	return
```

## Licenses<a name="serial-stream-connector-license"></a>

The Serial Stream connector includes the following third\-party software/licensing:
+ [pyserial](https://github.com/pyserial/pyserial)/BSD

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="serial-stream-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 3 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 2 | Updated connector ARN for AWS Region support\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="serial-stream-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)