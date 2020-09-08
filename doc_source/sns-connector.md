# SNS connector<a name="sns-connector"></a>

The SNS [connector](connectors.md) publishes messages to an Amazon SNS topic\. This enables web servers, email addresses, and other message subscribers to respond to events in the Greengrass group\.

This connector receives SNS message information on an MQTT topic, and then sends the message to a specified SNS topic\. You can optionally use custom Lambda functions to implement filtering or formatting logic on messages before they are published to this connector\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 4 | `arn:aws:greengrass:region::/connectors/SNS/versions/4` | 
| 3 | `arn:aws:greengrass:region::/connectors/SNS/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/SNS/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/SNS/versions/1` | 

For information about version changes, see the [Changelog](#sns-connector-changelog)\.

## Requirements<a name="sns-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 3 \- 4 ]
+ <a name="conn-req-ggc-v1.9.3"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-sns-req-sns-config"></a>A configured SNS topic\. For more information, see [Creating an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-topic.html) in the *Amazon Simple Notification Service Developer Guide*\.
+ <a name="conn-sns-req-iam-policy"></a>The [Greengrass group role](group-role.md) configured to allow the `sns:Publish` action on the target Amazon SNStopic, as shown in the following example IAM policy\.

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

  This connector allows you to dynamically override the default topic in the input message payload\. If your implementation uses this feature, the IAM policy must allow `sns:Publish` permission on all target topics\. You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\.

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

------
#### [ Versions 1 \- 2 ]
+ <a name="conn-req-ggc-v1.7.0"></a>AWS IoT Greengrass Core software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-sns-req-sns-config"></a>A configured SNS topic\. For more information, see [Creating an Amazon SNS topic](https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-topic.html) in the *Amazon Simple Notification Service Developer Guide*\.
+ <a name="conn-sns-req-iam-policy"></a>The [Greengrass group role](group-role.md) configured to allow the `sns:Publish` action on the target Amazon SNStopic, as shown in the following example IAM policy\.

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

  This connector allows you to dynamically override the default topic in the input message payload\. If your implementation uses this feature, the IAM policy must allow `sns:Publish` permission on all target topics\. You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\.

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

------

## Connector Parameters<a name="sns-connector-param"></a>

This connector provides the following parameters:

------
#### [ Version 4 ]

`DefaultSNSArn`  <a name="sns-DefaultSNSArn"></a>
The ARN of the default SNS topic to publish messages to\. The destination topic can be overridden by the `sns_topic_arn` property in the input message payload\.  
The group role must allow `sns:Publish` permission to all target topics\. For more information, see [Requirements](#sns-connector-req)\.
Display name in the AWS IoT console: **Default SNS topic ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:sns:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):([a-zA-Z0-9-_]+)$`

`IsolationMode`  <a name="IsolationMode"></a>
The [containerization](connectors.md#connector-containerization) mode for this connector\. The default is `GreengrassContainer`, which means that the connector runs in an isolated runtime environment inside the AWS IoT Greengrass container\.  
The default containerization setting for the group does not apply to connectors\.
Display name in the AWS IoT console: **Container isolation mode**  
Required: `false`  
Type: `string`  
Valid values: `GreengrassContainer` or `NoContainer`  
Valid pattern: `^NoContainer$|^GreengrassContainer$`

------
#### [ Versions 1 \- 3 ]

`DefaultSNSArn`  <a name="sns-DefaultSNSArn"></a>
The ARN of the default SNS topic to publish messages to\. The destination topic can be overridden by the `sns_topic_arn` property in the input message payload\.  
The group role must allow `sns:Publish` permission to all target topics\. For more information, see [Requirements](#sns-connector-req)\.
Display name in the AWS IoT console: **Default SNS topic ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:sns:([a-z]{2}-[a-z]+-\d{1}):(\d{12}):([a-zA-Z0-9-_]+)$`

------

### Create Connector Example \(AWS CLI\)<a name="sns-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the SNS connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MySNSConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/SNS/versions/4",
            "Parameters": {
                "DefaultSNSArn": "arn:aws:sns:region:account-id:topic-name",
                "IsolationMode" : "GreengrassContainer"
            }
        }
    ]
}'
```

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="sns-connector-data-input"></a>

This connector accepts SNS message information on an MQTT topic, and then publishes the message as is to the target SNS topic\. Input messages must be in JSON format\.

<a name="topic-filter"></a>**Topic filter in subscription**  
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

## Output data<a name="sns-connector-data-output"></a>

This connector publishes status information as output data on an MQTT topic\.

<a name="topic-filter"></a>**Topic filter in subscription**  
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

<a name="connectors-setup-intro"></a>Use the following high\-level steps to set up an example Python 3\.7 Lambda function that you can use to try out the connector\.

**Note**  <a name="connectors-setup-get-started-topics"></a>
The [Get started with connectors \(console\)](connectors-console.md) and [Get started with connectors \(CLI\)](connectors-cli.md) topics contain detailed steps that show you how to configure and deploy an example Twilio Notifications connector\.

1. Make sure you meet the [requirements](#sns-connector-req) for the connector\.

   <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

1. <a name="connectors-setup-function"></a>Create and publish a Lambda function that sends input data to the connector\.

   Save the [example code](#sns-connector-usage-example) as a PY file\. <a name="connectors-setup-function-sdk"></a>Download and unzip the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core)\. Then, create a zip package that contains the PY file and the `greengrasssdk` folder at the root level\. This zip package is the deployment package that you upload to AWS Lambda\.

   <a name="connectors-setup-function-publish"></a>After you create the Python 3\.7 Lambda function, publish a function version and create an alias\.

1. Configure your Greengrass group\.

   1. <a name="connectors-setup-gg-function"></a>Add the Lambda function by its alias \(recommended\)\. Configure the Lambda lifecycle as long\-lived \(or `"Pinned": true` in the CLI\)\.

   1. Add the connector and configure its [parameters](#sns-connector-param)\.

   1. Add subscriptions that allow the connector to receive [input data](#sns-connector-data-input) and send [output data](#sns-connector-data-output) on supported topic filters\.
      + <a name="connectors-setup-subscription-input-data"></a>Set the Lambda function as the source, the connector as the target, and use a supported input topic filter\.
      + <a name="connectors-setup-subscription-output-data"></a>Set the connector as the source, AWS IoT Core as the target, and use a supported output topic filter\. You use this subscription to view status messages in the AWS IoT console\.

1. <a name="connectors-setup-deploy-group"></a>Deploy the group\.

1. <a name="connectors-setup-test-sub"></a>In the AWS IoT console, on the **Test** page, subscribe to the output data topic to view status messages from the connector\. The example Lambda function is long\-lived and starts sending messages immediately after the group is deployed\.

   When you're finished testing, you can set the Lambda lifecycle to on\-demand \(or `"Pinned": false` in the CLI\) and deploy the group\. This stops the function from sending messages\.

### Example<a name="sns-connector-usage-example"></a>

The following example Lambda function sends an input message to the connector\.

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
    print("Message To Publish: ", messageToPublish)
    iot_client.publish(topic=send_topic,
        payload=json.dumps(messageToPublish))

publish_basic_message()

def lambda_handler(event, context):
    return
```

## Licenses<a name="sns-connector-license"></a>

The SNS connector includes the following third\-party software/licensing:<a name="boto-3-licenses"></a>
+ [AWS SDK for Python \(Boto3\)](https://pypi.org/project/boto3/)/Apache License 2\.0
+ [botocore](https://pypi.org/project/botocore/)/Apache License 2\.0
+ [dateutil](https://pypi.org/project/python-dateutil/1.4/)/PSF License
+ [docutils](https://pypi.org/project/docutils/)/BSD License, GNU General Public License \(GPL\), Python Software Foundation License, Public Domain
+ [jmespath](https://pypi.org/project/jmespath/)/MIT License
+ [s3transfer](https://pypi.org/project/s3transfer/)/Apache License 2\.0
+ [urllib3](https://pypi.org/project/urllib3/)/MIT License

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="sns-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 4 | <a name="isolation-mode-changelog"></a>Added the `IsolationMode` parameter to configure the containerization mode for the connector\. | 
| 3 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 2 | Fix to reduce excessive logging\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="sns-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)
+ [ Publish action](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sns.html#SNS.Client.publish) in the Boto 3 documentation
+ [What is Amazon Simple Notification Service?](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) in the *Amazon Simple Notification Service Developer Guide*