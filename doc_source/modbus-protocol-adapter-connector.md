# Modbus\-RTU Protocol Adapter<a name="modbus-protocol-adapter-connector"></a>

The Modbus\-RTU Protocol Adapter [connector](connectors.md) polls information from Modbus RTU devices that are in the AWS IoT Greengrass group\.

This connector receives parameters for a Modbus RTU request from a user\-defined Lambda function\. It sends the corresponding request, and then publishes the response from the target device as an MQTT message\.

**ARN**: `arn:aws:greengrass:region::/connectors/ModbusRTUProtocolAdapter/versions/1`

## Requirements<a name="modbus-protocol-adapter-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ A physical connection between the AWS IoT Greengrass core and the Modbus devices\. The core must be physically connected to the Modbus RTU network through a serial port \(for example, a USB port\)\.
+ A [local device resource](access-local-resources.md) in the Greengrass group that points to the physical Modbus serial port\.
+ A user\-defined Lambda function that sends Modbus RTU request parameters to this connector\. The request parameters must conform to expected patterns and include the IDs and addresses of the target devices on the Modbus RTU network\. For more information, see [Input Data](#modbus-protocol-adapter-connector-data-input)\.

## Connector Parameters<a name="modbus-protocol-adapter-connector-param"></a>

This connector supports the following parameters:

`ModbusSerialPort-ResourceId`  
The ID of the local device resource that represents the physical Modbus serial port\.  
This connector is granted read\-write access to the resource\.
Display name in console: **Modbus serial port resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`ModbusSerialPort`  
The absolute path to the physical Modbus serial port on the device\. This is the source path that's specified for the Modbus local device resource\.  
Display name in console: **Source path of Modbus serial port resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

### Create Connector Example \(CLI\)<a name="modbus-protocol-adapter-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the Modbus\-RTU Protocol Adapter connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyModbusRTUProtocolAdapterConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ModbusRTUProtocolAdapter/versions/1",
            "Parameters": {
                "ModbusSerialDevice-ResourceId": "MyLocalModbusSerialPort",
                "ModbusSerialDevice": "/path-to-port"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="modbus-protocol-adapter-connector-data-input"></a>

This connector accepts Modbus RTU request parameters from a user\-defined Lambda function on an MQTT topic\. Input messages must be in JSON format\.

**Topic filter**  
`modbus/adapter/request`

**Message properties**  
The request message varies based on the type of Modbus RTU request that it represents\. The following properties are required for all requests:  
+ In the `request` object:
  + `operation`\. The operation to execute, specified by name or function code\. For example, to read coils, you can specify `ReadCoilsRequest` or `0x01`\. This value must be a Unicode string\.
  + `device`\. The target device of the request\. This value must be between `0 - 247`\.
+ The `id` property\. An ID for the request\. This value is used for data deduplication and is returned as is in the `id` property of all responses, including error responses\. This value must be a Unicode string\.
The other parameters to include in the request depend on the operation\. All request parameters are required except the CRC, which is handled separately\. For examples, see [Example Requests and Responses](#modbus-protocol-adapter-connector-examples)\.

**Example input: Using operation name**  

```
{
    "request": {
        "operation": "ReadCoilsRequest",
    	"device": 1,
    	"address": 0x01,
    	"count": 1
    },
    "id": "TestRequest"
}
```

**Example input: Using function code**  

```
{
    "request": {
        "operation": 0x01,
    	"device": 1,
    	"address": 0x01,
    	"count": 1
    },
    "id": "TestRequest"
}
```
For more examples, see [Example Requests and Responses](#modbus-protocol-adapter-connector-examples)\.

## Output Data<a name="modbus-protocol-adapter-connector-data-output"></a>

This connector publishes responses to incoming Modbus RTU requests\.

**Topic filter**  
`modbus/adapter/response`

**Message properties**  
The format of the response message varies based on the corresponding request and the response status\. For examples, see [Example Requests and Responses](#modbus-protocol-adapter-connector-examples)\.  
A response for a write operation is simply an echo of the request\. Although no meaningful information is returned for write responses, it's a good practice to check the status of the response\.
Every response includes the following properties:  
+ In the `response` object:
  + `status`\. The status of the request\. The status can be one of the following values:
    + `Success`\. The request was valid, sent to the Modbus RTU network, and a response was returned\.
    + `Exception`\. The request was valid, sent to the Modbus RTU network, and an exception response was returned\. For more information, see [Response Status: Exception](#modbus-protocol-adapter-connector-response-exception)\.
    + `No Response`\. The request was invalid, and the connector caught the error before the request was sent over the Modbus RTU network\. For more information, see [Response Status: No Response](#modbus-protocol-adapter-connector-response-noresponse)\.
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
For more examples, see [Example Requests and Responses](#modbus-protocol-adapter-connector-examples)\.

## Modbus RTU Requests and Responses<a name="modbus-protocol-adapter-connector-requests-responses"></a>

This connector accepts Modbus RTU request parameters as [input data](#modbus-protocol-adapter-connector-data-input) and publishes responses as [output data](#modbus-protocol-adapter-connector-data-output)\.

The following common operations are supported\.


| Operation | Function Code | 
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

### Example Requests and Responses<a name="modbus-protocol-adapter-connector-examples"></a>

The following are example requests and responses for supported operations\.

Read Coils  
**Request example:**  

```
{
    "request": {
        "operation": "ReadCoilsRequest",
    	"device": 1,
    	"address": 0x01,
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
        "address": 0x01,
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
    	"address": 0x01,
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
    	"address": 0x01,
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
    	"address": 0x01,
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
    	"address": 0x01,
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
    	"address": 0x01,
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
    	"address": 0x01,
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
    	"address": 0x01,
        "and_mask": 0xaf,
        "or_mask": 0x01
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
    	"read_address": 0x01,
        "read_count": 2,
        "write_address": 0x03,
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

### Response Status: Exception<a name="modbus-protocol-adapter-connector-response-exception"></a>

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

### Response Status: No Response<a name="modbus-protocol-adapter-connector-response-noresponse"></a>

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

The following example Lambda function sends an input message to the connector\.

**Note**  
This Python function uses the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) to publish an MQTT message\. You can use the following [pip](https://pypi.org/project/pip/) command to install the Python version of the SDK on your core device:   

```
pip install greengrasssdk
```

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
			"address": 0x01,
			"count": 1
		},
		"id": "TestRequest"
	}
	return request

def publish_basic_request():
	iot_client.publish(payload=json.dumps(create_read_coils_request()), topic=TOPIC_REQUEST)

publish_basic_request()

def function_handler(event, context):
	return
```

## Licenses<a name="modbus-protocol-adapter-connector-license"></a>

The Modbus\-RTU Protocol Adapter connector includes the following third\-party software/licensing:
+ [pymodbus](https://github.com/riptideio/pymodbus/blob/master/README.rst) / BSD
+ [pyserial](https://github.com/pyserial/pyserial) / BSD

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## See Also<a name="modbus-protocol-adapter-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)