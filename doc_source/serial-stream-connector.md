# Serial Stream<a name="serial-stream-connector"></a>

The Serial Stream [connector](connectors.md) reads and writes to a serial port on an AWS IoT Greengrass core device\.

This connector supports two modes of operation:
+ **Read\-On\-Demand**\. Receives read and write requests on MQTT topics and publishes the response of the read operation or the status of the write operation\.
+ **Polling\-Read**\. Reads from the serial port at regular intervals\. This mode also supports Read\-On\-Demand requests\.

**Note**  
Read requests are limited to a maximum read length of 63994 bytes\. Write requests are limited to a maximum data length of 128000 bytes\.

**ARN**: `arn:aws:greengrass:region::/connectors/SerialStream/versions/1`

## Requirements<a name="serial-stream-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ A [local device resource](access-local-resources.md) in the Greengrass group that points to the target serial port\.
**Note**  
Before you deploy this connector, we recommend that you set up the serial port and verify that it can be read from and written to\.

## Connector Parameters<a name="serial-stream-connector-param"></a>

This connector provides the following parameters:

`BaudRate`  
The baud rate of the serial connection\.  
Display name in console: **Baud rate**  
Required: `true`  
Type: `string`  
Valid values: `110, 300, 600, 1200, 2400, 4800, 9600, 14400, 19200, 28800, 38400, 56000, 57600, 115200, 230400`  
Valid pattern: `^110$|^300$|^600$|^1200$|^2400$|^4800$|^9600$|^14400$|^19200$|^28800$|^38400$|^56000$|^57600$|^115200$|^230400$`

`Timeout`  
The timeout \(in seconds\) for a read operation\.  
Display name in console: **Timeout**  
Required: `true`  
Type: `string`  
Valid values: `1 - 59`  
Valid pattern: `^([1-9]|[1-5][0-9])$`

`SerialPort`  
The absolute path to the physical serial port on the device\. This is the source path that's specified for the local device resource\.  
Display name in console: **Serial port**  
Required: `true`  
Type: `string`  
Valid pattern: `[/a-zA-Z0-9_-]+`

`SerialPort-ResourceId`  
The ID of the local device resource that represents the physical serial port\.  
This connector is granted read\-write access to the resource\.
Display name in console: **Serial port resource**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9_-]+`

`PollingRead`  
Sets the read mode: Polling\-Read or Read\-On\-Demand\.  
+ For Polling\-Read mode, specify `true`\. In this mode, the `PollingInterval`, `PollingReadType`, and `PollingReadLength` properties are required\.
+ For Read\-On\-Demand mode, specify `false`\. In this mode, the type and length values are specified in the read request\.
Display name in console: **Read mode**  
Required: `true`  
Type: `string`  
Valid values: `true, false`  
Valid pattern: `^([Tt][Rr][Uu][Ee]|[Ff][Aa][Ll][Ss][Ee])$`

`PollingReadLength`  
The length of data \(in bytes\) to read in each polling read operation\. This applies only when using Polling\-Read mode\.  
Display name in console: **Polling read length**  
Required: `false`\. This property is required when `PollingRead` is `true`\.  
Type: `string`  
Valid pattern: `^(|[1-9][0-9]{0,3}|[1-5][0-9]{4}|6[0-2][0-9]{3}|63[0-8][0-9]{2}|639[0-8][0-9]|6399[0-4])$`

`PollingReadInterval`  
The interval \(in seconds\) at which the polling read takes place\. This applies only when using Polling\-Read mode\.  
Display name in console: **Polling read interval**  
Required: `false`\. This property is required when `PollingRead` is `true`\.  
Type: `string`  
Valid values: 1 \- 999  
Valid pattern: `^(|[1-9]|[1-9][0-9]|[1-9][0-9][0-9])$`

`PollingReadType`  
The type of data that the polling thread reads\. This applies only when using Polling\-Read mode\.  
Display name in console: **Polling read type**  
Required: `false`\. This property is required when `PollingRead` is `true`\.  
Type: `string`  
Valid values: `ascii, hex`  
Valid pattern: `^(|[Aa][Ss][Cc][Ii][Ii]|[Hh][Ee][Xx])$`

`RtsCts`  
Indicates whether to enable the RTS/CTS flow control\. The default value is `false`\. For more information, see [RTS, CTS, and RTR](https://en.wikipedia.org/wiki/RS-232#RTS,_CTS,_and_RTR)\.   
Display name in console: **RTS/CTS flow control**  
Required: `false`  
Type: `string`  
Valid values: `true, false`  
Valid pattern: `^(|[Tt][Rr][Uu][Ee]|[Ff][Aa][Ll][Ss][Ee])$`

`XonXoff`  
Indicates whether to enable the software flow control\. The default value is `false`\. For more information, see [Software flow control](https://en.wikipedia.org/wiki/Software_flow_control)\.  
Display name in console: **Software flow control**  
Required: `false`  
Type: `string`  
Valid values: `true, false`  
Valid pattern: `^(|[Tt][Rr][Uu][Ee]|[Ff][Aa][Ll][Ss][Ee])$`

`Parity`  
The parity of the serial port\. The default value is `N`\. For more information, see [Parity](https://en.wikipedia.org/wiki/Serial_port#Parity)\.   
Display name in console: **Serial port parity**  
Required: `false`  
Type: `string`  
Valid values: `N, E, O, S, M`  
Valid pattern: `^(|[NEOSMneosm])$`

### Create Connector Example \(CLI\)<a name="serial-stream-connector-create"></a>

The following CLI command creates an `ConnectorDefinition` with an initial version that contains the Serial Stream connector\. It configures the connector for Polling\-Read mode\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MySerialStreamConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/SerialStream/versions/1",
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

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="serial-stream-connector-data-input"></a>

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

## Output Data<a name="serial-stream-connector-data-output"></a>

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

The following example Lambda function sends an input message to the connector\.

**Note**  
This Python function uses the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) to publish an MQTT message\. You can use the following [pip](https://pypi.org/project/pip/) command to install the Python version of the SDK on your core device:   

```
pip install greengrasssdk
```

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

def function_handler(event, context):
	return
```

## Licenses<a name="serial-stream-connector-license"></a>

The Serial Stream connector includes the following third\-party software/licensing:
+ [pyserial](https://github.com/pyserial/pyserial) / BSD

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## See Also<a name="serial-stream-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)