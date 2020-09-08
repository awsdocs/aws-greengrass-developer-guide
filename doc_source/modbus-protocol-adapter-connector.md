# Modbus\-RTU Protocol Adapter connector<a name="modbus-protocol-adapter-connector"></a>

The Modbus\-RTU Protocol Adapter [connector](connectors.md) polls information from Modbus RTU devices that are in the AWS IoT Greengrass group\.

This connector receives parameters for a Modbus RTU request from a user\-defined Lambda function\. It sends the corresponding request, and then publishes the response from the target device as an MQTT message\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 3 | `arn:aws:greengrass:region::/connectors/ModbusRTUProtocolAdapter/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/ModbusRTUProtocolAdapter/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/ModbusRTUProtocolAdapter/versions/1` | 

For information about version changes, see the [Changelog](#modbus-protocol-adapter-connector-changelog)\.

## Requirements<a name="modbus-protocol-adapter-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 3 ]
+ <a name="conn-req-ggc-v1.9.3"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-modbus-req-physical-connection"></a>A physical connection between the AWS IoT Greengrass core and the Modbus devices\. The core must be physically connected to the Modbus RTU network through a serial port; for example, a USB port\.
+ <a name="conn-modbus-req-serial-port-resource"></a>A [local device resource](access-local-resources.md) in the Greengrass group that points to the physical Modbus serial port\.
+ <a name="conn-modbus-req-user-lambda"></a>A user\-defined Lambda function that sends Modbus RTU request parameters to this connector\. The request parameters must conform to expected patterns and include the IDs and addresses of the target devices on the Modbus RTU network\. For more information, see [Input data](#modbus-protocol-adapter-connector-data-input)\.

------
#### [ Versions 1 \- 2 ]
+ <a name="conn-req-ggc-v1.7.0"></a>AWS IoT Greengrass Core software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-modbus-req-physical-connection"></a>A physical connection between the AWS IoT Greengrass core and the Modbus devices\. The core must be physically connected to the Modbus RTU network through a serial port; for example, a USB port\.
+ <a name="conn-modbus-req-serial-port-resource"></a>A [local device resource](access-local-resources.md) in the Greengrass group that points to the physical Modbus serial port\.
+ <a name="conn-modbus-req-user-lambda"></a>A user\-defined Lambda function that sends Modbus RTU request parameters to this connector\. The request parameters must conform to expected patterns and include the IDs and addresses of the target devices on the Modbus RTU network\. For more information, see [Input data](#modbus-protocol-adapter-connector-data-input)\.

------

## Connector Parameters<a name="modbus-protocol-adapter-connector-param"></a>

This connector supports the following parameters:

`ModbusSerialPort-ResourceId`  
The ID of the local device resource that represents the physical Modbus serial port\.  
This connector is granted read\-write access to the resource\.
Display name in the AWS IoT console: **Modbus serial port resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`ModbusSerialPort`  
The absolute path to the physical Modbus serial port on the device\. This is the source path that's specified for the Modbus local device resource\.  
Display name in the AWS IoT console: **Source path of Modbus serial port resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

### Create Connector Example \(AWS CLI\)<a name="modbus-protocol-adapter-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the Modbus\-RTU Protocol Adapter connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyModbusRTUProtocolAdapterConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ModbusRTUProtocolAdapter/versions/3",
            "Parameters": {
                "ModbusSerialPort-ResourceId": "MyLocalModbusSerialPort",
                "ModbusSerialPort": "/path-to-port"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

**Note**  
After you deploy the Modbus\-RTU Protocol Adapter connector, you can use AWS IoT Things Graph to orchestrate interactions between devices in your group\. For more information, see [Modbus](https://docs.aws.amazon.com/thingsgraph/latest/ug/iot-tg-protocols-modbus.html) in the *AWS IoT Things Graph User Guide*\.

## Input data<a name="modbus-protocol-adapter-connector-data-input"></a>

This connector accepts Modbus RTU request parameters from a user\-defined Lambda function on an MQTT topic\. Input messages must be in JSON format\.

<a name="topic-filter"></a>**Topic filter in subscription**  
`modbus/adapter/request`

**Message properties**  
The request message varies based on the type of Modbus RTU request that it represents\. The following properties are required for all requests:  
+ In the `request` object:
  + `operation`\. The name of the operation to execute\. For example, specify `"operation": "ReadCoilsRequest"` to read coils\. This value must be a Unicode string\. For supported operations, see [Modbus RTU requests and responses](#modbus-protocol-adapter-connector-requests-responses)\.
  + `device`\. The target device of the request\. This value must be between `0 - 247`\.
+ The `id` property\. An ID for the request\. This value is used for data deduplication and is returned as is in the `id` property of all responses, including error responses\. This value must be a Unicode string\.
If your request includes an address field, you must specify the value as an integer\. For example, `"address": 1`\.
The other parameters to include in the request depend on the operation\. All request parameters are required except the CRC, which is handled separately\. For examples, see [Example requests and responses](#modbus-protocol-adapter-connector-examples)\.

**Example input: Read coils request**  

```
{
    "request": {
        "operation": "ReadCoilsRequest",
    	"device": 1,
    	"address": 1,
    	"count": 1
    },
    "id": "TestRequest"
}
```

## Output data<a name="modbus-protocol-adapter-connector-data-output"></a>

This connector publishes responses to incoming Modbus RTU requests\.

<a name="topic-filter"></a>**Topic filter in subscription**  
`modbus/adapter/response`

**Message properties**  
The format of the response message varies based on the corresponding request and the response status\. For examples, see [Example requests and responses](#modbus-protocol-adapter-connector-examples)\.  
A response for a write operation is simply an echo of the request\. Although no meaningful information is returned for write responses, it's a good practice to check the status of the response\.
Every response includes the following properties:  
+ In the `response` object:
  + `status`\. The status of the request\. The status can be one of the following values:
    + `Success`\. The request was valid, sent to the Modbus RTU network, and a response was returned\.
    + `Exception`\. The request was valid, sent to the Modbus RTU network, and an exception response was returned\. For more information, see [Response status: Exception](#modbus-protocol-adapter-connector-response-exception)\.
    + `No Response`\. The request was invalid, and the connector caught the error before the request was sent over the Modbus RTU network\. For more information, see [Response status: No response](#modbus-protocol-adapter-connector-response-noresponse)\.
  + `device`\. The device that the request was sent to\.
  + `operation`\. The request type that was sent\.
  + `payload`\. The response content that was returned\. If the `status` is `No Response`, this object contains only an `error` property with the error description \(for example, `"error": "[Input/Output] No Response received from the remote unit"`\)\.
+ The `id` property\. The ID of the request, used for data deduplication\.

**Example output: Success**  

```
{
    "response" : {
        "status" : "success",
        "device": 1,
    	"operation": "ReadCoilsRequest",
    	"payload": {
        	"function_code": 1,
        	"bits": [1]
    	}
     },
     "id" : "TestRequest"
}
```

**Example output: Failure**  

```
{
    "response" : {
        "status" : "fail",
        "error_message": "Internal Error",
        "error": "Exception",
        "device": 1,
    	"operation": "ReadCoilsRequest",
    	"payload": {
        	"function_code": 129,
        	"exception_code": 2
    	}
     },
     "id" : "TestRequest"
}
```
For more examples, see [Example requests and responses](#modbus-protocol-adapter-connector-examples)\.

## Modbus RTU requests and responses<a name="modbus-protocol-adapter-connector-requests-responses"></a>

This connector accepts Modbus RTU request parameters as [input data](#modbus-protocol-adapter-connector-data-input) and publishes responses as [output data](#modbus-protocol-adapter-connector-data-output)\.

The following common operations are supported\.


| Operation name in request | Function code in response | 
| --- | --- | 
| ReadCoilsRequest | 01 | 
| ReadDiscreteInputsRequest | 02 | 
| ReadHoldingRegistersRequest | 03 | 
| ReadInputRegistersRequest | 04 | 
| WriteSingleCoilRequest | 05 | 
| WriteSingleRegisterRequest | 06 | 
| WriteMultipleCoilsRequest | 15 | 
| WriteMultipleRegistersRequest | 16 | 
| MaskWriteRegisterRequest | 22 | 
| ReadWriteMultipleRegistersRequest | 23 | 

### Example requests and responses<a name="modbus-protocol-adapter-connector-examples"></a>

The following are example requests and responses for supported operations\.

Read Coils  
**Request example:**  

```
{
    "request": {
        "operation": "ReadCoilsRequest",
    	"device": 1,
    	"address": 1,
    	"count": 1
    },
    "id": "TestRequest"
}
```
**Response example:**  

```
{
    "response": {
        "status": "success",
        "device": 1,
    	"operation": "ReadCoilsRequest",
    	"payload": {
        	"function_code": 1,
        	"bits": [1]
    	}
     },
     "id" : "TestRequest"
}
```

Read Discrete Inputs  
**Request example:**  

```
{
    "request": {
        "operation": "ReadDiscreteInputsRequest",
        "device": 1,
        "address": 1,
        "count": 1
    },
    "id": "TestRequest"
}
```
**Response example:**  

```
{
    "response": {
        "status": "success",
        "device": 1,
        "operation": "ReadDiscreteInputsRequest",
        "payload": {
            "function_code": 2,
            "bits": [1]
        }
     },
     "id" : "TestRequest"
}
```

Read Holding Registers  
**Request example:**  

```
{
    "request": {
        "operation": "ReadHoldingRegistersRequest",
    	"device": 1,
    	"address": 1,
    	"count": 1
    },
    "id": "TestRequest"
}
```
**Response example:**  

```
{
    "response": {
        "status": "success",
        "device": 1,
    	"operation": "ReadHoldingRegistersRequest",
    	"payload": {
    	    "function_code": 3,
            "registers": [20,30]
    	}
     },
     "id" : "TestRequest"
}
```

Read Input Registers  
**Request example:**  

```
{
    "request": {
        "operation": "ReadInputRegistersRequest",
    	"device": 1,
    	"address": 1,
    	"value": 1
    },
    "id": "TestRequest"
}
```

Write Single Coil  
**Request example:**  

```
{
    "request": {
        "operation": "WriteSingleCoilRequest",
    	"device": 1,
    	"address": 1,
    	"value": 1
    },
    "id": "TestRequest"
}
```
**Response example:**  

```
{
    "response": {
        "status": "success",
        "device": 1,
    	"operation": "WriteSingleCoilRequest",
    	"payload": {
    	    "function_code": 5,
    	    "address": 1,
    	    "value": true
    	}
     },
     "id" : "TestRequest"
```

Write Single Register  
**Request example:**  

```
{
    "request": {
        "operation": "WriteSingleRegisterRequest",
    	"device": 1,
    	"address": 1,
    	"value": 1
    },
    "id": "TestRequest"
}
```

Write Multiple Coils  
**Request example:**  

```
{
    "request": {
        "operation": "WriteMultipleCoilsRequest",
    	"device": 1,
    	"address": 1,
    	"values": [1,0,0,1]
    },
    "id": "TestRequest"
}
```
**Response example:**  

```
{
    "response": {
        "status": "success",
        "device": 1,
    	"operation": "WriteMultipleCoilsRequest",
    	"payload": {
    	    "function_code": 15,
    	    "address": 1,
    	    "count": 4
    	}
     },
     "id" : "TestRequest"
}
```

Write Multiple Registers  
**Request example:**  

```
{
    "request": {
        "operation": "WriteMultipleRegistersRequest",
    	"device": 1,
    	"address": 1,
    	"values": [20,30,10]
    },
    "id": "TestRequest"
}
```
**Response example:**  

```
{
    "response": {
        "status": "success",
        "device": 1,
    	"operation": "WriteMultipleRegistersRequest",
    	"payload": {
    	    "function_code": 23,
    	    "address": 1,
       		"count": 3
    	}
     },
     "id" : "TestRequest"
}
```

Mask Write Register  
**Request example:**  

```
{
    "request": {
        "operation": "MaskWriteRegisterRequest",
    	"device": 1,
    	"address": 1,
        "and_mask": 175,
        "or_mask": 1
    },
    "id": "TestRequest"
}
```
**Response example:**  

```
{
    "response": {
        "status": "success",
        "device": 1,
    	"operation": "MaskWriteRegisterRequest",
    	"payload": {
    	    "function_code": 22,
            "and_mask": 0,
            "or_mask": 8
    	}
     },
     "id" : "TestRequest"
}
```

Read Write Multiple Registers  
**Request example:**  

```
{
    "request": {
        "operation": "ReadWriteMultipleRegistersRequest",
    	"device": 1,
    	"read_address": 1,
        "read_count": 2,
        "write_address": 3,
        "write_registers": [20,30,40]
    },
    "id": "TestRequest"
}
```
**Response example:**  

```
{
    "response": {
        "status": "success",
        "device": 1,
    	"operation": "ReadWriteMultipleRegistersRequest",
    	"payload": {
    	    "function_code": 23,
    	    "registers": [10,20,10,20]
    	}
     },
     "id" : "TestRequest"
}
```
The registers returned in this response are the registers that are read from\.

### Response status: Exception<a name="modbus-protocol-adapter-connector-response-exception"></a>

Exceptions can occur when the request format is valid, but the request is not completed successfully\. In this case, the response contains the following information:
+ The `status` is set to `Exception`\.
+ The `function_code` equals the function code of the request \+ 128\.
+ The `exception_code` contains the exception code\. For more information, see Modbus exception codes\.

**Example:**

```
{
    "response" : {
        "status" : "fail",
        "error_message": "Internal Error",
        "error": "Exception",
        "device": 1,
    	"operation": "ReadCoilsRequest",
    	"payload": {
        	"function_code": 129,
        	"exception_code": 2
    	}
     },
     "id" : "TestRequest"
}
```

### Response status: No response<a name="modbus-protocol-adapter-connector-response-noresponse"></a>

This connector performs validation checks on the Modbus request\. For example, it checks for invalid formats and missing fields\. If the validation fails, the connector doesn't send the request\. Instead, it returns a response that contains the following information:
+ The `status` is set to `No Response`\.
+ The `error` contains the error reason\.
+ The `error_message` contains the error message\.

**Examples:**

```
{
    "response" : {
        "status" : "fail",
        "error_message": "Invalid address field. Expected <type 'int'>, got <type 'str'>",
        "error": "No Response",
        "device": 1,
    	"operation": "ReadCoilsRequest",
    	"payload": {
        	"error": "Invalid address field. Expected Expected <type 'int'>, got <type 'str'>"
    	}
     },
     "id" : "TestRequest"
}
```

If the request targets a nonexistent device or if the Modbus RTU network is not working, you might get a `ModbusIOException`, which uses the No Response format\.

```
{
    "response" : {
        "status" : "fail",
        "error_message": "[Input/Output] No Response received from the remote unit",
        "error": "No Response",
        "device": 1,
    	"operation": "ReadCoilsRequest",
    	"payload": {
        	"error": "[Input/Output] No Response received from the remote unit"
    	}
     },
     "id" : "TestRequest"
}
```

## Usage Example<a name="modbus-protocol-adapter-connector-usage"></a>

<a name="connectors-setup-intro"></a>Use the following high\-level steps to set up an example Python 3\.7 Lambda function that you can use to try out the connector\.

**Note**  <a name="connectors-setup-get-started-topics"></a>
The [Get started with connectors \(console\)](connectors-console.md) and [Get started with connectors \(CLI\)](connectors-cli.md) topics contain detailed steps that show you how to configure and deploy an example Twilio Notifications connector\.

1. Make sure you meet the [requirements](#modbus-protocol-adapter-connector-req) for the connector\.

1. <a name="connectors-setup-function"></a>Create and publish a Lambda function that sends input data to the connector\.

   Save the [example code](#modbus-protocol-adapter-connector-usage-example) as a PY file\. <a name="connectors-setup-function-sdk"></a>Download and unzip the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core)\. Then, create a zip package that contains the PY file and the `greengrasssdk` folder at the root level\. This zip package is the deployment package that you upload to AWS Lambda\.

   <a name="connectors-setup-function-publish"></a>After you create the Python 3\.7 Lambda function, publish a function version and create an alias\.

1. Configure your Greengrass group\.

   1. <a name="connectors-setup-gg-function"></a>Add the Lambda function by its alias \(recommended\)\. Configure the Lambda lifecycle as long\-lived \(or `"Pinned": true` in the CLI\)\.

   1. <a name="connectors-setup-device-resource"></a>Add the required local device resource and grant read/write access to the Lambda function\.

   1. Add the connector and configure its [parameters](#modbus-protocol-adapter-connector-param)\.

   1. Add subscriptions that allow the connector to receive [input data](#modbus-protocol-adapter-connector-data-input) and send [output data](#modbus-protocol-adapter-connector-data-output) on supported topic filters\.
      + <a name="connectors-setup-subscription-input-data"></a>Set the Lambda function as the source, the connector as the target, and use a supported input topic filter\.
      + <a name="connectors-setup-subscription-output-data"></a>Set the connector as the source, AWS IoT Core as the target, and use a supported output topic filter\. You use this subscription to view status messages in the AWS IoT console\.

1. <a name="connectors-setup-deploy-group"></a>Deploy the group\.

1. <a name="connectors-setup-test-sub"></a>In the AWS IoT console, on the **Test** page, subscribe to the output data topic to view status messages from the connector\. The example Lambda function is long\-lived and starts sending messages immediately after the group is deployed\.

   When you're finished testing, you can set the Lambda lifecycle to on\-demand \(or `"Pinned": false` in the CLI\) and deploy the group\. This stops the function from sending messages\.

### Example<a name="modbus-protocol-adapter-connector-usage-example"></a>

The following example Lambda function sends an input message to the connector\.

```
import greengrasssdk
import json

TOPIC_REQUEST = 'modbus/adapter/request'

# Creating a greengrass core sdk client
iot_client = greengrasssdk.client('iot-data')

def create_read_coils_request():
	request = {
		"request": {
			"operation": "ReadCoilsRequest",
			"device": 1,
			"address": 1,
			"count": 1
		},
		"id": "TestRequest"
	}
	return request

def publish_basic_request():
	iot_client.publish(payload=json.dumps(create_read_coils_request()), topic=TOPIC_REQUEST)

publish_basic_request()

def lambda_handler(event, context):
	return
```

## Licenses<a name="modbus-protocol-adapter-connector-license"></a>

The Modbus\-RTU Protocol Adapter connector includes the following third\-party software/licensing:
+ [pymodbus](https://github.com/riptideio/pymodbus/blob/master/README.rst)/BSD
+ [pyserial](https://github.com/pyserial/pyserial)/BSD

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="modbus-protocol-adapter-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 3 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 2 | Updated connector ARN for AWS Region support\. Improved error logging\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="modbus-protocol-adapter-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)