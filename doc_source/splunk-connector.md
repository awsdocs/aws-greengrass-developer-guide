# Splunk Integration connector<a name="splunk-connector"></a>

The Splunk Integration [connector](connectors.md) publishes data from Greengrass devices to Splunk\. This allows you to use Splunk to monitor and analyze the Greengrass core environment, and act on local events\. The connector integrates with HTTP Event Collector \(HEC\)\. For more information, see [Introduction to Splunk HTTP Event Collector](https://dev.splunk.com/view/event-collector/SP-CAAAE6M) in the Splunk documentation\.

This connector receives logging and event data on an MQTT topic and publishes the data as is to the Splunk API\.

You can use this connector to support industrial scenarios, such as:
+ Operators can use periodic data from actuators and sensors \(for example, temperature, pressure, and water readings\) to trigger alarms when values exceed certain thresholds\.
+ Developers use data collected from industrial machinery to build ML models that can monitor the equipment for potential issues\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 4 | `arn:aws:greengrass:region::/connectors/SplunkIntegration/versions/4` | 
| 3 | `arn:aws:greengrass:region::/connectors/SplunkIntegration/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/SplunkIntegration/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/SplunkIntegration/versions/1` | 

For information about version changes, see the [Changelog](#splunk-connector-changelog)\.

## Requirements<a name="splunk-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 3 \- 4 ]
+ <a name="conn-req-ggc-v1.9.3-secrets"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\. AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This requirement includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-splunk-req-http-event-collector"></a>The HTTP Event Collector functionality must be enabled in Splunk\. For more information, see [Set up and use HTTP eEvent Collector in Splunk Web](https://docs.splunk.com/Documentation/Splunk/7.2.0/Data/UsetheHTTPEventCollector) in the Splunk documentation\.
+ <a name="conn-splunk-req-secret"></a>A text type secret in AWS Secrets Manager that stores your Splunk HTTP Event Collector token\. For more information, see [About event collector tokens](https://docs.splunk.com/Documentation/Splunk/7.2.0/Data/UsetheHTTPEventCollector#About_Event_Collector_tokens) in the Splunk documentation and [Creating a basic secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.
**Note**  
To create the secret in the Secrets Manager console, enter your token on the **Plaintext** tab\. Don't include quotation marks or other formatting\. In the API, specify the token as the value for the `SecretString` property\.
+ A secret resource in the Greengrass group that references the Secrets Manager secret\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

------
#### [ Versions 1 \- 2 ]
+ <a name="conn-req-ggc-v1.7.0-secrets"></a>AWS IoT Greengrass Core software v1\.7 or later\. AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This requirement includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-splunk-req-http-event-collector"></a>The HTTP Event Collector functionality must be enabled in Splunk\. For more information, see [Set up and use HTTP eEvent Collector in Splunk Web](https://docs.splunk.com/Documentation/Splunk/7.2.0/Data/UsetheHTTPEventCollector) in the Splunk documentation\.
+ <a name="conn-splunk-req-secret"></a>A text type secret in AWS Secrets Manager that stores your Splunk HTTP Event Collector token\. For more information, see [About event collector tokens](https://docs.splunk.com/Documentation/Splunk/7.2.0/Data/UsetheHTTPEventCollector#About_Event_Collector_tokens) in the Splunk documentation and [Creating a basic secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.
**Note**  
To create the secret in the Secrets Manager console, enter your token on the **Plaintext** tab\. Don't include quotation marks or other formatting\. In the API, specify the token as the value for the `SecretString` property\.
+ A secret resource in the Greengrass group that references the Secrets Manager secret\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

------

## Connector Parameters<a name="splunk-connector-param"></a>

This connector provides the following parameters:

------
#### [ Version 4 ]

`SplunkEndpoint`  <a name="splunk-SplunkEndpoint"></a>
The endpoint of your Splunk instance\. This value must contain the protocol, hostname, and port\.  
Display name in the AWS IoT console: **Splunk endpoint**  
Required: `true`  
Type: `string`  
Valid pattern: `^(http:\/\/|https:\/\/)?[a-z0-9]+([-.]{1}[a-z0-9]+)*.[a-z]{2,5}(:[0-9]{1,5})?(\/.*)?$`

`MemorySize`  <a name="splunk-MemorySize"></a>
The amount of memory \(in KB\) to allocate to the connector\.  
Display name in the AWS IoT console: **Memory size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`SplunkQueueSize`  <a name="splunk-SplunkQueueSize"></a>
The maximum number of items to save in memory before the items are submitted or discarded\. When this limit is met, the oldest items in the queue are replaced with newer items\. This limit typically applies when there's no connection to the internet\.  
Display name in the AWS IoT console: **Maximum items to retain**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`SplunkFlushIntervalSeconds`  <a name="splunk-SplunkFlushIntervalSeconds"></a>
The interval \(in seconds\) for publishing received data to Splunk HEC\. The maximum value is 900\. To configure the connector to publish items as they are received \(without batching\), specify 0\.  
Display name in the AWS IoT console: **Splunk publish interval**  
Required: `true`  
Type: `string`  
Valid pattern: `[0-9]|[1-9]\d|[1-9]\d\d|900`

`SplunkTokenSecretArn`  <a name="splunk-SplunkTokenSecretArn"></a>
The secret in AWS Secrets Manager that stores the Splunk token\. This must be a text type secret\.  
Display name in the AWS IoT console: **ARN of Splunk auth token secret**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:secretsmanager:[a-z]{2}-[a-z]+-\d{1}:\d{12}?:secret:[a-zA-Z0-9-_]+-[a-zA-Z0-9-_]+`

`SplunkTokenSecretArn-ResourceId`  <a name="splunk-SplunkTokenSecretArn-ResourceId"></a>
The secret resource in the Greengrass group that references the Splunk secret\.  
Display name in the AWS IoT console: **Splunk auth token resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`SplunkCustomCALocation`  <a name="splunk-SplunkCustomCALocation"></a>
The file path of the custom certificate authority \(CA\) for Splunk \(for example, `/etc/ssl/certs/splunk.crt`\)\.  
Display name in the AWS IoT console: **Splunk custom certificate authority location**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|/.*`

`IsolationMode`  <a name="IsolationMode"></a>
The [containerization](connectors.md#connector-containerization) mode for this connector\. The default is `GreengrassContainer`, which means that the connector runs in an isolated runtime environment inside the AWS IoT Greengrass container\.  
The default containerization setting for the group does not apply to connectors\.
Display name in the AWS IoT console: **Container isolation mode**  
Required: `false`  
Type: `string`  
Valid values: `GreengrassContainer` or `NoContainer`  
Valid pattern: `^NoContainer$|^GreengrassContainer$`

------
#### [ Version 1 \- 3 ]

`SplunkEndpoint`  <a name="splunk-SplunkEndpoint"></a>
The endpoint of your Splunk instance\. This value must contain the protocol, hostname, and port\.  
Display name in the AWS IoT console: **Splunk endpoint**  
Required: `true`  
Type: `string`  
Valid pattern: `^(http:\/\/|https:\/\/)?[a-z0-9]+([-.]{1}[a-z0-9]+)*.[a-z]{2,5}(:[0-9]{1,5})?(\/.*)?$`

`MemorySize`  <a name="splunk-MemorySize"></a>
The amount of memory \(in KB\) to allocate to the connector\.  
Display name in the AWS IoT console: **Memory size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`SplunkQueueSize`  <a name="splunk-SplunkQueueSize"></a>
The maximum number of items to save in memory before the items are submitted or discarded\. When this limit is met, the oldest items in the queue are replaced with newer items\. This limit typically applies when there's no connection to the internet\.  
Display name in the AWS IoT console: **Maximum items to retain**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`SplunkFlushIntervalSeconds`  <a name="splunk-SplunkFlushIntervalSeconds"></a>
The interval \(in seconds\) for publishing received data to Splunk HEC\. The maximum value is 900\. To configure the connector to publish items as they are received \(without batching\), specify 0\.  
Display name in the AWS IoT console: **Splunk publish interval**  
Required: `true`  
Type: `string`  
Valid pattern: `[0-9]|[1-9]\d|[1-9]\d\d|900`

`SplunkTokenSecretArn`  <a name="splunk-SplunkTokenSecretArn"></a>
The secret in AWS Secrets Manager that stores the Splunk token\. This must be a text type secret\.  
Display name in the AWS IoT console: **ARN of Splunk auth token secret**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:secretsmanager:[a-z]{2}-[a-z]+-\d{1}:\d{12}?:secret:[a-zA-Z0-9-_]+-[a-zA-Z0-9-_]+`

`SplunkTokenSecretArn-ResourceId`  <a name="splunk-SplunkTokenSecretArn-ResourceId"></a>
The secret resource in the Greengrass group that references the Splunk secret\.  
Display name in the AWS IoT console: **Splunk auth token resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`SplunkCustomCALocation`  <a name="splunk-SplunkCustomCALocation"></a>
The file path of the custom certificate authority \(CA\) for Splunk \(for example, `/etc/ssl/certs/splunk.crt`\)\.  
Display name in the AWS IoT console: **Splunk custom certificate authority location**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|/.*`

------

### Create Connector Example \(AWS CLI\)<a name="splunk-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the Splunk Integration connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MySplunkIntegrationConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/SplunkIntegration/versions/4",
            "Parameters": {
                "SplunkEndpoint": "https://myinstance.cloud.splunk.com:8088",
                "MemorySize": 200000,
                "SplunkQueueSize": 10000,
                "SplunkFlushIntervalSeconds": 5,
                "SplunkTokenSecretArn":"arn:aws:secretsmanager:region:account-id:secret:greengrass-secret-hash",
                "SplunkTokenSecretArn-ResourceId": "MySplunkResource", 
                "IsolationMode" : "GreengrassContainer"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="splunk-connector-data-input"></a>

This connector accepts logging and event data on an MQTT topic and publishes the received data as is to the Splunk API\. Input messages must be in JSON format\.

<a name="topic-filter"></a>**Topic filter in subscription**  
`splunk/logs/put`

**Message properties**    
`request`  
The event data to send to the Splunk API\. Events must meet the specifications of the [services/collector](https://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTinput#services.2Fcollector) API\.  
Required: `true`  
Type: `object`\. Only the `event` property is required\.  
`id`  
An arbitrary ID for the request\. This property is used to map an input request to an output status\.  
Required: `false`  
Type: `string`

**Limits**  
All limits that are imposed by the Splunk API apply when using this connector\. For more information, see [services/collector](https://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTinput#services.2Fcollector)\.

**Example input**  

```
{
    "request": {
        "event": "some event",
        "fields": {
            "severity": "INFO",
            "category": [
                "value1",
                "value2"
            ]
        }
    },
    "id": "request123"
}
```

## Output data<a name="splunk-connector-data-output"></a>

This connector publishes output data on two topics:
+ Status information on the `splunk/logs/put/status` topic\.
+ Errors on the `splunk/logs/put/error` topic\.

**Topic filter:** `splunk/logs/put/status`  
Use this topic to listen for the status of the requests\. Each time that the connector sends a batch of received data to the Splunk API, it publishes a list of the IDs of the requests that succeeded and failed\.    
**Example output**  

```
{
    "response": {
        "succeeded": [
            "request123",
            ...
        ],
        "failed": [
            "request789",
            ...
        ]
    }
}
```

**Topic filter:** `splunk/logs/put/error`  
Use this topic to listen for errors from the connector\. The `error_message` property that describes the error or timeout encountered while processing the request\.    
**Example output**  

```
{
    "response": {
        "error": "UnauthorizedException",
        "error_message": "invalid splunk token",
        "status": "fail"
    }
}
```
If the connector detects a retryable error \(for example, connection errors\), it retries the publish in the next batch\.

## Usage Example<a name="splunk-connector-usage"></a>

<a name="connectors-setup-intro"></a>Use the following high\-level steps to set up an example Python 3\.7 Lambda function that you can use to try out the connector\.

**Note**  <a name="connectors-setup-get-started-topics"></a>
The [Get started with connectors \(console\)](connectors-console.md) and [Get started with connectors \(CLI\)](connectors-cli.md) topics contain detailed steps that show you how to configure and deploy an example Twilio Notifications connector\.

1. Make sure you meet the [requirements](#splunk-connector-req) for the connector\.

1. <a name="connectors-setup-function"></a>Create and publish a Lambda function that sends input data to the connector\.

   Save the [example code](#splunk-connector-usage-example) as a PY file\. <a name="connectors-setup-function-sdk"></a>Download and unzip the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core)\. Then, create a zip package that contains the PY file and the `greengrasssdk` folder at the root level\. This zip package is the deployment package that you upload to AWS Lambda\.

   <a name="connectors-setup-function-publish"></a>After you create the Python 3\.7 Lambda function, publish a function version and create an alias\.

1. Configure your Greengrass group\.

   1. <a name="connectors-setup-gg-function"></a>Add the Lambda function by its alias \(recommended\)\. Configure the Lambda lifecycle as long\-lived \(or `"Pinned": true` in the CLI\)\.

   1. <a name="connectors-setup-secret-resource"></a>Add the required secret resource and grant read access to the Lambda function\.

   1. Add the connector and configure its [parameters](#splunk-connector-param)\.

   1. Add subscriptions that allow the connector to receive [input data](#splunk-connector-data-input) and send [output data](#splunk-connector-data-output) on supported topic filters\.
      + <a name="connectors-setup-subscription-input-data"></a>Set the Lambda function as the source, the connector as the target, and use a supported input topic filter\.
      + <a name="connectors-setup-subscription-output-data"></a>Set the connector as the source, AWS IoT Core as the target, and use a supported output topic filter\. You use this subscription to view status messages in the AWS IoT console\.

1. <a name="connectors-setup-deploy-group"></a>Deploy the group\.

1. <a name="connectors-setup-test-sub"></a>In the AWS IoT console, on the **Test** page, subscribe to the output data topic to view status messages from the connector\. The example Lambda function is long\-lived and starts sending messages immediately after the group is deployed\.

   When you're finished testing, you can set the Lambda lifecycle to on\-demand \(or `"Pinned": false` in the CLI\) and deploy the group\. This stops the function from sending messages\.

### Example<a name="splunk-connector-usage-example"></a>

The following example Lambda function sends an input message to the connector\.

```
import greengrasssdk
import time
import json

iot_client = greengrasssdk.client('iot-data')
send_topic = 'splunk/logs/put'

def create_request_with_all_fields():
    return {
        "request": {
            "event": "Access log test message."
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

## Licenses<a name="splunk-connector-license"></a>

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="splunk-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 4 | <a name="isolation-mode-changelog"></a>Added the `IsolationMode` parameter to configure the containerization mode for the connector\. | 
| 3 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 2 | Fix to reduce excessive logging\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="splunk-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)