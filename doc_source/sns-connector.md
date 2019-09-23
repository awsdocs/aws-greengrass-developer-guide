# SNS Connector<a name="sns-connector"></a>

The SNS [connector](connectors.md) publishes messages to an Amazon SNS topic\. This enables web servers, email addresses, and other message subscribers to respond to events in the Greengrass group\.

This connector receives SNS message information on an MQTT topic, and then sends the message to a specified SNS topic\. You can optionally use custom Lambda functions to implement filtering or formatting logic on messages before they are published to this connector\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 2 | arn:aws:greengrass:*region*::/connectors/SNS/versions/2 | 
| 1 | arn:aws:greengrass:*region*::/connectors/SNS/versions/1 | 

For information about version changes, see the [Changelog](#sns-connector-changelog)\.

## Requirements<a name="sns-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ A configured SNS topic\. For more information, see [Create a Topic](https://docs.aws.amazon.com/sns/latest/dg/CreateTopic.html) in the *Amazon Simple Notification Service Developer Guide*\.
+ An IAM policy added to the Greengrass group role that allows the `sns:Publish` action on the target SNS topic, as shown in the following example:

  ```
  {
      "Version":"2012-10-17",
      "Statement":[
          {
              "Sid":"Stmt1528133056761",
              "Action":[
                  "sns:Publish"
              ],
              "Effect":"Allow",
              "Resource":[
                  "arn:aws:sns:region:account-id:topic-name"
              ]
          }
      ]
   }
  ```

  This connector allows you to dynamically override the default topic in the input message payload\. If your implementation uses this feature, the IAM policy must allow `sns:Publish` permission on all target topics\. You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\. For more information, see [ Adding and Removing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

## Connector Parameters<a name="sns-connector-param"></a>

This connector provides the following parameters:

`DefaultSNSArn`  
The ARN of the default SNS topic to publish messages to\. The destination topic can be overridden by the `sns_topic_arn` property in the input message payload\.  
The group role must allow `sns:Publish` permission to all target topics\. For more information, see [Requirements](#sns-connector-req)\.
Display name in console: **Default SNS topic ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:sns:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):([a-zA-Z0-9-_]+)$`

### Create Connector Example \(CLI\)<a name="sns-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the SNS connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MySNSConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/SNS/versions/2",
            "Parameters": {
                "DefaultSNSArn": "arn:aws:sns:region:account-id:topic-name"
            }
        }
    ]
}'
```

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="sns-connector-data-input"></a>

This connector accepts SNS message information on an MQTT topic, and then publishes the message as is to the target SNS topic\. Input messages must be in JSON format\.

**Topic filter**  
`sns/message`

**Message properties**    
`request`  
Information about the message to send to the SNS topic\.  
Required: `true`  
Type: `object` that includes the following properties:    
`message`  
The content of the message as a string or in JSON format\. For examples, see [Example input](#sns-connector-data-input-example)\.  
To send JSON, the `message_structure` property must be set to `json` and the message must be a string\-encoded JSON object that contains a `default` key\.  
Required: `true`  
Type: `string`  
Valid pattern: `.*`  
`subject`  
The subject of the message\.  
Required: `false`  
Type: ASCII text, up to 100 characters\. This must begin with a letter, number, or punctuation mark\. This must not include line breaks or control characters\.  
Valid pattern: `.*`  
`sns_topic_arn`  
The ARN of the SNS topic to publish messages to\. If specified, the connector publishes to this topic instead of the default topic\.  
The group role must allow `sns:Publish` permission to any target topics\. For more information, see [Requirements](#sns-connector-req)\.
Required: `false`  
Type: `string`  
Valid pattern: `arn:aws:sns:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):([a-zA-Z0-9-_]+)$`  
`message_structure`  
The structure of the message\.  
Required: `false`\. This must be specified to send a JSON message\.  
Type: `string`  
Valid values: `json`  
`id`  
An arbitrary ID for the request\. This property is used to map an input request to an output response\. When specified, the `id` property in the response object is set to this value\. If you don't use this feature, you can omit this property or specify an empty string\.  
Required: `false`  
Type: `string`  
Valid pattern: `.*`

**Limits**  
The message size is bounded by a maximum SNS message size of 256 KB\.

**Example input: String message**  <a name="sns-connector-data-input-example"></a>
This example sends a string message\. It specifies the optional `sns_topic_arn` property, which overrides the default destination topic\.  

```
{
    "request": {
        "subject": "Message subject",
        "message": "Message data",
        "sns_topic_arn": "arn:aws:sns:region:account-id:topic2-name"
    },
    "id": "request123"
}
```

**Example input: JSON message**  
This example sends a message as a string encoded JSON object that includes the `default` key\.  

```
{
    "request": {
        "subject": "Message subject",
        "message": "{ \"default\": \"Message data\" }",
        "message_structure": "json"
    },
    "id": "request123"
}
```

## Output Data<a name="sns-connector-data-output"></a>

This connector publishes status information as output data\.

**Topic filter**  
`sns/message/status`

**Example output: Success**  

```
{
    "response": {
        "sns_message_id": "f80a81bc-f44c-56f2-a0f0-d5af6a727c8a",
        "status": "success"
    },
    "id": "request123"
}
```

**Example output: Failure**  

```
{
   "response" : {
        "error": "InvalidInputException",
        "error_message": "SNS Topic Arn is invalid",
        "status": "fail"
   },
   "id": "request123"
}
```

## Usage Example<a name="sns-connector-usage"></a>

The following example Lambda function sends an input message to the connector\.

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
send_topic = 'sns/message'

def create_request_with_all_fields():
    return  {
        "request": {
            "message": "Message from SNS Connector Test"
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

## Licenses<a name="sns-connector-license"></a>

The SNS connector includes the following third\-party software/licensing:
+ [AWS SDK for Python \(Boto 3\)](https://github.com/boto/boto3)/Apache 2\.0

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## Changelog<a name="sns-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 2 | Fix to reduce excessive logging\. | 
| 1 | Initial release\.  | 

A Greengrass group can contain only one version of the connector at a time\.

## See Also<a name="sns-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)
+ [ Publish action](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sns.html#SNS.Client.publish) in the Boto 3 documentation
+ [What Is Amazon Simple Notification Service?](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) in the *Amazon Simple Notification Service Developer Guide*