# Kinesis Firehose<a name="kinesis-firehose-connector"></a>

The Kinesis Firehose [connector](connectors.md) publishes data through an Amazon Kinesis Data Firehose delivery stream to destinations such as Amazon S3, Amazon Redshift, or Amazon Elasticsearch Service\.

This connector is a data producer for a Kinesis delivery stream\. It receives input data on an MQTT topic, and sends the data to a specified delivery stream\. The delivery stream then sends the data record to the configured destination \(for example, an S3 bucket\)\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 4 | `arn:aws:greengrass:region::/connectors/KinesisFirehose/versions/4` | 
| 3 | `arn:aws:greengrass:region::/connectors/KinesisFirehose/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/KinesisFirehose/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/KinesisFirehose/versions/1` | 

For information about version changes, see the [Changelog](#kinesis-firehose-connector-changelog)\.

## Requirements<a name="kinesis-firehose-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 4 ]
+ <a name="conn-req-ggc-v1.9.3"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="req-kinesis-firehose-stream"></a>A configured Kinesis delivery stream\. For more information, see [Creating an Amazon Kinesis Data Firehose delivery stream](https://docs.aws.amazon.com/firehose/latest/dev/basic-create.html) in the *Amazon Kinesis Firehose Developer Guide*\.
+ <a name="req-kinesis-firehose-iam-policy-v2"></a>The [Greengrass group role](group-role.md) configured to allow the `firehose:PutRecord` and `firehose:PutRecordBatch` actions on the target delivery stream, as shown in the following example IAM policy\.

  ```
  {
      "Version":"2012-10-17",
      "Statement":[
          {
              "Sid":"Stmt1528133056761",
              "Action":[
                  "firehose:PutRecord",
                  "firehose:PutRecordBatch"
              ],
              "Effect":"Allow",
              "Resource":[
                  "arn:aws:firehose:region:account-id:deliverystream/stream-name"
              ]
          }
      ]
   }
  ```

  This connector allows you to dynamically override the default delivery stream in the input message payload\. If your implementation uses this feature, the IAM policy should include all target streams as resources\. You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\.

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

------
#### [ Versions 2 \- 3 ]
+ <a name="conn-req-ggc-v1.7.0"></a>AWS IoT Greengrass Core software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="req-kinesis-firehose-stream"></a>A configured Kinesis delivery stream\. For more information, see [Creating an Amazon Kinesis Data Firehose delivery stream](https://docs.aws.amazon.com/firehose/latest/dev/basic-create.html) in the *Amazon Kinesis Firehose Developer Guide*\.
+ <a name="req-kinesis-firehose-iam-policy-v2"></a>The [Greengrass group role](group-role.md) configured to allow the `firehose:PutRecord` and `firehose:PutRecordBatch` actions on the target delivery stream, as shown in the following example IAM policy\.

  ```
  {
      "Version":"2012-10-17",
      "Statement":[
          {
              "Sid":"Stmt1528133056761",
              "Action":[
                  "firehose:PutRecord",
                  "firehose:PutRecordBatch"
              ],
              "Effect":"Allow",
              "Resource":[
                  "arn:aws:firehose:region:account-id:deliverystream/stream-name"
              ]
          }
      ]
   }
  ```

  This connector allows you to dynamically override the default delivery stream in the input message payload\. If your implementation uses this feature, the IAM policy should include all target streams as resources\. You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\.

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

------
#### [ Version 1 ]
+ <a name="conn-req-ggc-v1.7.0"></a>AWS IoT Greengrass Core software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="req-kinesis-firehose-stream"></a>A configured Kinesis delivery stream\. For more information, see [Creating an Amazon Kinesis Data Firehose delivery stream](https://docs.aws.amazon.com/firehose/latest/dev/basic-create.html) in the *Amazon Kinesis Firehose Developer Guide*\.
+ The [Greengrass group role](group-role.md) configured to allow the `firehose:PutRecord` action on the target delivery stream, as shown in the following example IAM policy\.

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

  <a name="role-resources"></a>This connector allows you to dynamically override the default delivery stream in the input message payload\. If your implementation uses this feature, the IAM policy should include all target streams as resources\. You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\.

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

------

## Connector Parameters<a name="kinesis-firehose-connector-param"></a>

This connector provides the following parameters:

------
#### [ Versions 2 \- 4 ]

`DefaultDeliveryStreamArn`  <a name="kinesis-firehose-DefaultDeliveryStreamArn"></a>
The ARN of the default Kinesis Data Firehose delivery stream to send data to\. The destination stream can be overridden by the `delivery_stream_arn` property in the input message payload\.  
The group role must allow the appropriate actions on all target delivery streams\. For more information, see [Requirements](#kinesis-firehose-connector-req)\.
Display name in the AWS IoT console: **Default delivery stream ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:firehose:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):deliverystream/([a-zA-Z0-9_\-.]+)$`

`DeliveryStreamQueueSize`  
The maximum number of records to retain in memory before new records for the same delivery stream are rejected\. The minimum value is 2000\.  
Display name in the AWS IoT console: **Maximum number of records to buffer \(per stream\)**  
Required: `true`  
Type: `string`  
Valid pattern: `^([2-9]\\d{3}|[1-9]\\d{4,})$`

`MemorySize`  
The amount of memory \(in KB\) to allocate to this connector\.  
Display name in the AWS IoT console: **Memory size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`PublishInterval`  
The interval \(in seconds\) for publishing records to Kinesis Data Firehose\. To disable batching, set this value to 0\.  
Display name in the AWS IoT console: **Publish interval**  
Required: `true`  
Type: `string`  
Valid values: `0 - 900`  
Valid pattern: `[0-9]|[1-9]\\d|[1-9]\\d\\d|900`

------
#### [ Version 1 ]

`DefaultDeliveryStreamArn`  <a name="kinesis-firehose-DefaultDeliveryStreamArn"></a>
The ARN of the default Kinesis Data Firehose delivery stream to send data to\. The destination stream can be overridden by the `delivery_stream_arn` property in the input message payload\.  
The group role must allow the appropriate actions on all target delivery streams\. For more information, see [Requirements](#kinesis-firehose-connector-req)\.
Display name in the AWS IoT console: **Default delivery stream ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:firehose:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):deliverystream/([a-zA-Z0-9_\-.]+)$`

------

**Example**  <a name="kinesis-firehose-connector-create"></a>
**Create Connector Example \(AWS CLI\)**  
The following CLI command creates a `ConnectorDefinition` with an initial version that contains the connector\.  

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyKinesisFirehoseConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/KinesisFirehose/versions/4",
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

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="kinesis-firehose-connector-data-input"></a>

This connector accepts stream content on MQTT topics, and then sends the content to the target delivery stream\. It accepts two types of input data:
+ JSON data on the `kinesisfirehose/message` topic\.
+ Binary data on the `kinesisfirehose/message/binary/#` topic\.

------
#### [ Versions 2 \- 4 ]<a name="kinesis-firehose-input-data"></a>

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

## Output data<a name="kinesis-firehose-connector-data-output"></a>

This connector publishes status information as output data on an MQTT topic\.

------
#### [ Versions 2 \- 4 ]

<a name="topic-filter"></a>**Topic filter in subscription**  <a name="kinesis-firehose-output-topic-status"></a>
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

<a name="topic-filter"></a>**Topic filter in subscription**  <a name="kinesis-firehose-output-topic-status"></a>
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

<a name="connectors-setup-intro"></a>Use the following high\-level steps to set up an example Python 3\.7 Lambda function that you can use to try out the connector\.

**Note**  <a name="connectors-setup-get-started-topics"></a>
The [Get started with connectors \(console\)](connectors-console.md) and [Get started with connectors \(CLI\)](connectors-cli.md) topics contain detailed steps that show you how to configure and deploy an example Twilio Notifications connector\.

1. Make sure you meet the [requirements](#kinesis-firehose-connector-req) for the connector\.

   <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

1. <a name="connectors-setup-function"></a>Create and publish a Lambda function that sends input data to the connector\.

   Save the [example code](#kinesis-firehose-connector-usage-example) as a PY file\. <a name="connectors-setup-function-sdk"></a>Download and unzip the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core)\. Then, create a zip package that contains the PY file and the `greengrasssdk` folder at the root level\. This zip package is the deployment package that you upload to AWS Lambda\.

   <a name="connectors-setup-function-publish"></a>After you create the Python 3\.7 Lambda function, publish a function version and create an alias\.

1. Configure your Greengrass group\.

   1. <a name="connectors-setup-gg-function"></a>Add the Lambda function by its alias \(recommended\)\. Configure the Lambda lifecycle as long\-lived \(or `"Pinned": true` in the CLI\)\.

   1. Add the connector and configure its [parameters](#kinesis-firehose-connector-param)\.

   1. Add subscriptions that allow the connector to receive [JSON input data](#kinesis-firehose-connector-data-input) and send [output data](#kinesis-firehose-connector-data-output) on supported topic filters\.
      + <a name="connectors-setup-subscription-input-data"></a>Set the Lambda function as the source, the connector as the target, and use a supported input topic filter\.
      + <a name="connectors-setup-subscription-output-data"></a>Set the connector as the source, AWS IoT Core as the target, and use a supported output topic filter\. You use this subscription to view status messages in the AWS IoT console\.

1. <a name="connectors-setup-deploy-group"></a>Deploy the group\.

1. <a name="connectors-setup-test-sub"></a>In the AWS IoT console, on the **Test** page, subscribe to the output data topic to view status messages from the connector\. The example Lambda function is long\-lived and starts sending messages immediately after the group is deployed\.

   When you're finished testing, you can set the Lambda lifecycle to on\-demand \(or `"Pinned": false` in the CLI\) and deploy the group\. This stops the function from sending messages\.

### Example<a name="kinesis-firehose-connector-usage-example"></a>

The following example Lambda function sends an input message to the connector\. This message contains JSON data\.

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
    print("Message To Publish: ", messageToPublish)
    iot_client.publish(topic=send_topic,
        payload=json.dumps(messageToPublish))

publish_basic_message()

def lambda_handler(event, context):
    return
```

## Licenses<a name="kinesis-firehose-connector-license"></a>

The Kinesis Firehose connector includes the following third\-party software/licensing:<a name="boto-3-licenses"></a>
+ [AWS SDK for Python \(Boto3\)](https://pypi.org/project/boto3/)/Apache License 2\.0
+ [botocore](https://pypi.org/project/botocore/)/Apache License 2\.0
+ [dateutil](https://pypi.org/project/python-dateutil/1.4/)/PSF License
+ [docutils](https://pypi.org/project/docutils/)/BSD License, GNU General Public License \(GPL\), Python Software Foundation License, Public Domain
+ [jmespath](https://pypi.org/project/jmespath/)/MIT License
+ [s3transfer](https://pypi.org/project/s3transfer/)/Apache License 2\.0
+ [urllib3](https://pypi.org/project/urllib3/)/MIT License

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="kinesis-firehose-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 4 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 3 | Fix to reduce excessive logging and other minor bug fixes\.  | 
| 2 | Added support for sending batched data records to Kinesis Data Firehose at a specified interval\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/kinesis-firehose-connector.html)  | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="kinesis-firehose-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)
+ [What is Amazon Kinesis Data Firehose?](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html) in the *Amazon Kinesis Developer Guide*