# Kinesis Firehose<a name="kinesis-firehose-connector"></a>

The Kinesis Firehose [connector](connectors.md) publishes data through an Amazon Kinesis Data Firehose delivery stream to destinations such as Amazon S3, Amazon Redshift, or Amazon Elasticsearch Service\.

This connector is a data producer for a Kinesis delivery stream\. It receives input data on an MQTT topic, and sends the data to a specified delivery stream\. The delivery stream then sends the data record to the configured destination \(for example, an S3 bucket\)\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 2 | arn:aws:greengrass:*region*::/connectors/KinesisFirehose/versions/2 | 
| 1 | arn:aws:greengrass:*region*::/connectors/KinesisFirehose/versions/1 | 

For information about version changes, see the [Changelog](#kinesis-firehose-connector-changelog)\.

## Requirements<a name="kinesis-firehose-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 2 ]
+ <a name="req-ggc-v170"></a>AWS IoT Greengrass Core Software v1\.7 or later\.
+ <a name="req-python-runtime"></a>[Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="req-kinesis-firehose-stream"></a>A configured Kinesis delivery stream\. For more information, see [Creating an Amazon Kinesis Data Firehose Delivery Stream](https://docs.aws.amazon.com/firehose/latest/dev/basic-create.html) in the *Amazon Kinesis Firehose Developer Guide*\.
+ An IAM policy added to the Greengrass group role that allows the `firehose:PutRecords` and `firehose:PutRecord` actions on the target delivery stream, as shown in the following example:

  ```
  {
      "Version":"2012-10-17",
      "Statement":[
          {
              "Sid":"Stmt1528133056761",
              "Action":[
                  "firehose:PutRecord",
                  "firehose:PutRecords"
              ],
              "Effect":"Allow",
              "Resource":[
                  "arn:aws:firehose:region:account-id:deliverystream/stream-name"
              ]
          }
      ]
   }
  ```

  <a name="role-resources"></a>This connector allows you to dynamically override the default delivery stream in the input message payload\. If your implementation uses this feature, the IAM policy should include all target streams as resources\. You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\. For more information, see [Adding and Removing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

------
#### [ Version 1 ]
+ <a name="req-ggc-v170"></a>AWS IoT Greengrass Core Software v1\.7 or later\.
+ <a name="req-python-runtime"></a>[Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="req-kinesis-firehose-stream"></a>A configured Kinesis delivery stream\. For more information, see [Creating an Amazon Kinesis Data Firehose Delivery Stream](https://docs.aws.amazon.com/firehose/latest/dev/basic-create.html) in the *Amazon Kinesis Firehose Developer Guide*\.
+ An IAM policy added to the Greengrass group role that allows the `firehose:PutRecord` action on the target delivery stream, as shown in the following example:

  ```
  {
      "Version":"2012-10-17",
      "Statement":[
          {
              "Sid":"Stmt1528133056761",
              "Action":[
                  "firehose:PutRecord"
              ],
              "Effect":"Allow",
              "Resource":[
                  "arn:aws:firehose:region:account-id:deliverystream/stream-name"
              ]
          }
      ]
   }
  ```

  <a name="role-resources"></a>This connector allows you to dynamically override the default delivery stream in the input message payload\. If your implementation uses this feature, the IAM policy should include all target streams as resources\. You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\. For more information, see [Adding and Removing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

------

## Connector Parameters<a name="kinesis-firehose-connector-param"></a>

This connector provides the following parameters:

------
#### [ Version 2 ]

`DefaultDeliveryStreamArn`  <a name="kinesis-firehose-DefaultDeliveryStreamArn"></a>
The ARN of the default Kinesis Data Firehose delivery stream to send data to\. The destination stream can be overridden by the `delivery_stream_arn` property in the input message payload\.  
The group role must allow the `firehose:PutRecord` action on all target delivery streams\. For more information, see [Requirements](#kinesis-firehose-connector-req)\.
Display name in console: **Default delivery stream ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:firehose:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):deliverystream/([a-zA-Z0-9_\-.]+)$`

`DeliveryStreamQueueSize`  
The maximum number of records to retain in memory before new records for the same delivery stream are rejected\. The minimum value is 2000\.  
Display name in console: **Maximum number of records to buffer \(per stream\)**  
Required: `true`  
Type: `string`  
Valid pattern: `^([2-9]\\d{3}|[1-9]\\d{4,})$`

`MemorySize`  
The amount of memory \(in KB\) to allocate to this connector\.  
Display name in console: **Memory size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`PublishInterval`  
The interval \(in seconds\) for publishing records to Kinesis Data Firehose\. To disable batching, set this value to 0\.  
Display name in console: **Publish interval**  
Required: `true`  
Type: `string`  
Valid values: `0 - 900`  
Valid pattern: `[0-9]|[1-9]\\d|[1-9]\\d\\d|900`

**Example**  <a name="kinesis-firehose-connector-create"></a>
**Create Connector Example \(CLI\)**  
The following CLI command creates a `ConnectorDefinition` with an initial version that contains the connector\.  

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyKinesisFirehoseConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/KinesisFirehose/versions/2",
            "Parameters": {
                "DefaultDeliveryStreamArn": "arn:aws:firehose:region:account-id:deliverystream/stream-name",
                "DeliveryStreamQueueSize": "5000",
                "MemorySize": "65535",
                "PublishInterval": "10"
            }
        }
    ]
}'
```

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

------
#### [ Version 1 ]

`DefaultDeliveryStreamArn`  <a name="kinesis-firehose-DefaultDeliveryStreamArn"></a>
The ARN of the default Kinesis Data Firehose delivery stream to send data to\. The destination stream can be overridden by the `delivery_stream_arn` property in the input message payload\.  
The group role must allow the `firehose:PutRecord` action on all target delivery streams\. For more information, see [Requirements](#kinesis-firehose-connector-req)\.
Display name in console: **Default delivery stream ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:firehose:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):deliverystream/([a-zA-Z0-9_\-.]+)$`

**Example**  <a name="kinesis-firehose-connector-create"></a>
**Create Connector Example \(CLI\)**  
The following CLI command creates a `ConnectorDefinition` with an initial version that contains the connector\.  

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyKinesisFirehoseConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/KinesisFirehose/versions/1",
            "Parameters": {
                "DefaultDeliveryStreamArn": "arn:aws:firehose:region:account-id:deliverystream/stream-name"
            }
        }
    ]
}'
```

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

------

## Input Data<a name="kinesis-firehose-connector-data-input"></a>

This connector accepts stream content on MQTT topics, and then sends the content to the target delivery stream\. It accepts two types of input data:
+ JSON data on the `kinesisfirehose/message` topic\.
+ Binary data on the `kinesisfirehose/message/binary/#` topic\.

------
#### [ Version 2 ]<a name="kinesis-firehose-input-data"></a>

**Topic filter**: `kinesisfirehose/message`  
Use this topic to send a message that contains JSON data\.    
**Message properties**    
`request`  
The data to send to the delivery stream and the target delivery stream, if different from the default stream\.  
Required: `true`  
Type: `object` that includes the following properties:    
`data`  
The data to send to the delivery stream\.  
Required: `true`  
Type: `bytes`  
`delivery_stream_arn`  
The ARN of the target Kinesis delivery stream\. Include this property to override the default delivery stream\.  
Required: `false`  
Type: `string`  
Valid pattern: `arn:aws:firehose:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):deliverystream/([a-zA-Z0-9_\-.]+)$`  
`id`  
An arbitrary ID for the request\. This property is used to map an input request to an output response\. When specified, the `id` property in the response object is set to this value\. If you don't use this feature, you can omit this property or specify an empty string\.  
Required: `false`  
Type: `string`  
Valid pattern: `.*`  
**Example input**  

```
{
     "request": {
        "delivery_stream_arn": "arn:aws:firehose:region:account-id:deliverystream/stream2-name",
        "data": "Data to send to the delivery stream."
     },
     "id": "request123"
}
```
 

**Topic filter**: `kinesisfirehose/message/binary/#`  
Use this topic to send a message that contains binary data\. The connector doesn't parse binary data\. The data is streamed as is\.  
To map the input request to an output response, replace the `#` wildcard in the message topic with an arbitrary request ID\. For example, if you publish a message to `kinesisfirehose/message/binary/request123`, the `id` property in the response object is set to `request123`\.  
If you don't want to map a request to a response, you can publish your messages to `kinesisfirehose/message/binary/`\. Be sure to include the trailing slash\.

------
#### [ Version 1 ]<a name="kinesis-firehose-input-data"></a>

**Topic filter**: `kinesisfirehose/message`  
Use this topic to send a message that contains JSON data\.    
**Message properties**    
`request`  
The data to send to the delivery stream and the target delivery stream, if different from the default stream\.  
Required: `true`  
Type: `object` that includes the following properties:    
`data`  
The data to send to the delivery stream\.  
Required: `true`  
Type: `bytes`  
`delivery_stream_arn`  
The ARN of the target Kinesis delivery stream\. Include this property to override the default delivery stream\.  
Required: `false`  
Type: `string`  
Valid pattern: `arn:aws:firehose:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):deliverystream/([a-zA-Z0-9_\-.]+)$`  
`id`  
An arbitrary ID for the request\. This property is used to map an input request to an output response\. When specified, the `id` property in the response object is set to this value\. If you don't use this feature, you can omit this property or specify an empty string\.  
Required: `false`  
Type: `string`  
Valid pattern: `.*`  
**Example input**  

```
{
     "request": {
        "delivery_stream_arn": "arn:aws:firehose:region:account-id:deliverystream/stream2-name",
        "data": "Data to send to the delivery stream."
     },
     "id": "request123"
}
```
 

**Topic filter**: `kinesisfirehose/message/binary/#`  
Use this topic to send a message that contains binary data\. The connector doesn't parse binary data\. The data is streamed as is\.  
To map the input request to an output response, replace the `#` wildcard in the message topic with an arbitrary request ID\. For example, if you publish a message to `kinesisfirehose/message/binary/request123`, the `id` property in the response object is set to `request123`\.  
If you don't want to map a request to a response, you can publish your messages to `kinesisfirehose/message/binary/`\. Be sure to include the trailing slash\.

------

## Output Data<a name="kinesis-firehose-connector-data-output"></a>

This connector publishes status information as output data\.

------
#### [ Version 2 ]

**Topic filter**  <a name="kinesis-firehose-output-topic-status"></a>
`kinesisfirehose/message/status`

**Example output**  
The response contains the status of each data record sent in the batch\.  

```
{
    "response": [
        {
            "ErrorCode": "error",
            "ErrorMessage": "test error",
            "id": "request123",
            "status": "fail"
        },
        {
            "firehose_record_id": "xyz2",
            "id": "request456",
            "status": "success"
        },
        {
            "firehose_record_id": "xyz3",
            "id": "request890",
            "status": "success"
        }
    ]
}
```
If the connector detects a retryable error \(for example, connection errors\), it retries the publish in the next batch\. Exponential backoff is handled by the AWS SDK\. Requests that fail with retryable errors are added back to the end of the queue for further publishing\.

------
#### [ Version 1 ]

**Topic filter**  <a name="kinesis-firehose-output-topic-status"></a>
`kinesisfirehose/message/status`

**Example output: Success**  

```
{
   "response": {
       "firehose_record_id": "1lxfuuuFomkpJYzt/34ZU/r8JYPf8Wyf7AXqlXm",
       "status": "success"
    },
    "id": "request123"
}
```

**Example output: Failure**  

```
{
   "response" : {
       "error": "ResourceNotFoundException",
       "error_message": "An error occurred (ResourceNotFoundException) when calling the PutRecord operation: Firehose test1 not found under account 123456789012.",
       "status": "fail"
   },
   "id": "request123"
}
```

------

## Usage Example<a name="kinesis-firehose-connector-usage"></a>

The following example Lambda function sends an input message to the connector\. This message contains JSON data\.

**Note**  
This Python function uses the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) to publish an MQTT message\. You can use the following [pip](https://pypi.org/project/pip/) command to install the Python version of the SDK on your core device:   

```
pip install greengrasssdk
```

```
import greengrasssdk
import time
import json

iot_client = greengrasssdk.client('iot-data')
send_topic = 'kinesisfirehose/message'

def create_request_with_all_fields():
    return  {
        "request": {
            "data": "Message from Firehose Connector Test"
        },
        "id" : "req_123"
    }

def publish_basic_message():
    messageToPublish = create_request_with_all_fields()
    print "Message To Publish: ", messageToPublish
    iot_client.publish(topic=send_topic,
        payload=json.dumps(messageToPublish))

publish_basic_message()

def function_handler(event, context):
    return
```

## Licenses<a name="kinesis-firehose-connector-license"></a>

The Kinesis Firehose connector includes the following third\-party software and licensing:
+ [AWS SDK for Python \(Boto 3\)](https://github.com/boto/boto3) / Apache 2\.0

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## Changelog<a name="kinesis-firehose-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 2 | Added support for sending batched data records to Kinesis Data Firehose at a specified interval\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/kinesis-firehose-connector.html)  | 
| 1 | Initial release\.  | 

A Greengrass group can contain only one version of the connector at a time\.

## See Also<a name="kinesis-firehose-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)
+ [What Is Amazon Kinesis Data Firehose?](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html) in the *Amazon Kinesis Developer Guide*