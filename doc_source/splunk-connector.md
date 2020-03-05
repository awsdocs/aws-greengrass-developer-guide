# Splunk Integration Connector<a name="splunk-connector"></a>

The Splunk Integration [connector](connectors.md) publishes data from Greengrass devices to Splunk\. This allows you to use Splunk to monitor and analyze the Greengrass core environment, and act on local events\. The connector integrates with HTTP Event Collector \(HEC\)\. For more information, see [Introduction to Splunk HTTP Event Collector](https://dev.splunk.com/view/event-collector/SP-CAAAE6M) in the Splunk documentation\.

This connector receives logging and event data on an MQTT topic and publishes the data as is to the Splunk API\.

You can use this connector to support industrial scenarios, such as:
+ Operators can use periodic data from actuators and sensors \(for example, temperature, pressure, and water readings\) to trigger alarms when values exceed certain thresholds\.
+ Developers use data collected from industrial machinery to build ML models that can monitor the equipment for potential issues\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 2 | `arn:aws:greengrass:region::/connectors/SplunkIntegration/versions/`2 | 
| 1 | `arn:aws:greengrass:region::/connectors/SplunkIntegration/versions/`1 | 

For information about version changes, see the [Changelog](#splunk-connector-changelog)\.

## Requirements<a name="splunk-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\. AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ The HTTP Event Collector functionality must be enabled in Splunk\. For more information, see [Set up and use HTTP Event Collector in Splunk Web](https://docs.splunk.com/Documentation/Splunk/7.2.0/Data/UsetheHTTPEventCollector) in the Splunk documentation\.
+ A text type secret in AWS Secrets Manager that stores your Splunk HTTP Event Collector token\. For more information, see [About Event Collector tokens](https://docs.splunk.com/Documentation/Splunk/7.2.0/Data/UsetheHTTPEventCollector#About_Event_Collector_tokens) in the Splunk documentation and [Creating a Basic Secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.
**Note**  
To create the secret in the Secrets Manager console, enter your token on the **Plaintext** tab\. Don't include quotation marks or other formatting\. In the API, specify the token as the value for the `SecretString` property\.
+ A secret resource in the Greengrass group that references the Secrets Manager secret\. For more information, see [Deploy Secrets to the AWS IoT Greengrass Core](secrets.md)\.

## Connector Parameters<a name="splunk-connector-param"></a>

This connector provides the following parameters:

`SplunkEndpoint`  
The endpoint of your Splunk instance\. This value must contain the protocol, hostname, and port\.  
Display name in the AWS IoT console: **Splunk endpoint**  
Required: `true`  
Type: `string`  
Valid pattern: `^(http:\/\/|https:\/\/)?[a-z0-9]+([-.]{1}[a-z0-9]+)*.[a-z]{2,5}(:[0-9]{1,5})?(\/.*)?$`

`MemorySize`  
The amount of memory \(in KB\) to allocate to the connector\.  
Display name in the AWS IoT console: **Memory size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`SplunkQueueSize`  
The maximum number of items to save in memory before the items are submitted or discarded\. When this limit is met, the oldest items in the queue are replaced with newer items\. This limit typically applies when there's no connection to the internet\.  
Display name in the AWS IoT console: **Maximum items to retain**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`SplunkFlushIntervalSeconds`  
The interval \(in seconds\) for publishing received data to Splunk HEC\. The maximum value is 900\. To configure the connector to publish items as they are received \(without batching\), specify 0\.  
Display name in the AWS IoT console: **Splunk publish interval**  
Required: `true`  
Type: `string`  
Valid pattern: `[0-9]|[1-9]\d|[1-9]\d\d|900`

`SplunkTokenSecretArn`  
The secret in AWS Secrets Manager that stores the Splunk token\. This must be a text type secret\.  
Display name in the AWS IoT console: **ARN of Splunk auth token secret**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:secretsmanager:[a-z]{2}-[a-z]+-\d{1}:\d{12}?:secret:[a-zA-Z0-9-_]+-[a-zA-Z0-9-_]+`

`SplunkTokenSecretArn-ResourceId`  
The secret resource in the Greengrass group that references the Splunk secret\.  
Display name in the AWS IoT console: **Splunk auth token resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`SplunkCustomCALocation`  
The file path of the custom certificate authority \(CA\) for Splunk \(for example, `/etc/ssl/certs/splunk.crt`\)\.  
Display name in the AWS IoT console: **Splunk custom certificate authority location**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|/.*`

### Create Connector Example \(AWS CLI\)<a name="splunk-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the Splunk Integration connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MySplunkIntegrationConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/SplunkIntegration/versions/2",
            "Parameters": {
                "SplunkEndpoint": "https://myinstance.cloud.splunk.com:8088",
                "MemorySize": 200000,
                "SplunkQueueSize": 10000,
                "SplunkFlushIntervalSeconds": 5,
                "SplunkTokenSecretArn":"arn:aws:secretsmanager:region:account-id:secret:greengrass-secret-hash",
                "SplunkTokenSecretArn-ResourceId": "MySplunkResource"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="splunk-connector-data-input"></a>

This connector accepts logging and event data on an MQTT topic and publishes the received data as is to the Splunk API\. Input messages must be in JSON format\.

**Topic filter**  
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

## Output Data<a name="splunk-connector-data-output"></a>

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

The following example Lambda function sends an input message to the connector\.

**Note**  
This Python function uses the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) to publish an MQTT message\.

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
    print "Message To Publish: ", messageToPublish
    iot_client.publish(topic=send_topic,
        payload=json.dumps(messageToPublish))

publish_basic_message()

def function_handler(event, context):
    return
```

## Licenses<a name="splunk-connector-license"></a>

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## Changelog<a name="splunk-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 2 | Fix to reduce excessive logging\. | 
| 1 | Initial release\.  | 

A Greengrass group can contain only one version of the connector at a time\.

## See Also<a name="splunk-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)