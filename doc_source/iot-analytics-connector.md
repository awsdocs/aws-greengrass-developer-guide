# IoT Analytics Connector<a name="iot-analytics-connector"></a>

The IoT Analytics connector sends local device data to AWS IoT Analytics\. You can use this connector as a central hub to collect data from sensors on the Greengrass core device and from [connected Greengrass devices](what-is-gg.md#greengrass-devices)\. The connector sends the data to AWS IoT Analytics channels in the current AWS account and Region\. It can send data to a default destination channel and to dynamically specified channels\.

**Note**  
AWS IoT Analytics is a fully managed service that allows you to collect, store, process, and query IoT data\. In AWS IoT Analytics, the data can be further analyzed and processed\. For example, it can be used to train ML models for monitoring machine health or to test new modeling strategies\. For more information, see [What Is AWS IoT Analytics?](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html) in the *AWS IoT Analytics User Guide*\.

The connector accepts formatted and unformatted data on [input MQTT topics](#iot-analytics-connector-data-input)\. It supports two predefined topics where the destination channel is specified inline\. It can also receive messages on customer\-defined topics that are [configured in subscriptions](connectors.md#connectors-inputs-outputs)\. This can be used to route messages from devices that publish to fixed topics or handle unstructured or stack\-dependent data from resource\-constrained devices\.

This connector uses the [ `BatchPutMessage`](https://docs.aws.amazon.com/iotanalytics/latest/userguide/api.html#cli-iotanalytics-batchputmessage) API to send data \(as a JSON or base64\-encoded string\) to the destination channel\. The connector can process raw data into a format that conforms to API requirements\. The connector buffers input messages in per\-channel queues and asynchronously processes the batches\. It provides parameters that allow you to control queueing and batching behavior and to restrict memory consumption\. For example, you can configure the maximum queue size, batch interval, memory size, and number of active channels\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 2 | `arn:aws:greengrass:region::/connectors/IoTAnalytics/versions/`2 | 
| 1 | `arn:aws:greengrass:region::/connectors/IoTAnalytics/versions/`1 | 

For information about version changes, see the [Changelog](#iot-analytics-connector-changelog)\.

## Requirements<a name="iot-analytics-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ This connector can be used only in supported AWS Regions\. For more information, see [Limits](#iot-analytics-connector-limits)\.
+ All related AWS IoT Analytics entities \(channels, pipeline, datastores, datasets\) and workflows are created and configured\. For more information, see the [AWS CLI](https://docs.aws.amazon.com/iotanalytics/latest/userguide/getting-started.html) or [console](https://docs.aws.amazon.com/iotanalytics/latest/userguide/quickstart.html) procedures in the *AWS IoT Analytics User Guide*\.
**Note**  
Destination AWS IoT Analytics channels must use the same account and be in the same AWS Region as this connector\.
+ An IAM policy added to the Greengrass [group role](config-iam-roles.md) that allows the `iotanalytics:BatchPutMessage` action on destination channels, as shown in the following example\. The channels must be in the current AWS account and Region\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "Stmt1528133056761",
              "Action": [
                  "iotanalytics:BatchPutMessage"
              ],
              "Effect": "Allow",
              "Resource": [
                  "arn:aws:iotanalytics:region:account-id:channel/channel_1_name",
                  "arn:aws:iotanalytics:region:account-id:channel/channel_2_name"
              ]
          }
      ]
  }
  ```

  For more information, see [Adding and Removing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

## Parameters<a name="iot-analytics-connector-param"></a>

`MemorySize`  
The amount of memory \(in KB\) to allocate to this connector\.  
Display name in console: **Memory size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`PublishRegion`  
The AWS Region that your AWS IoT Analytics channels are created in\. Use the same Region as the connector\.  
This must also match the Region for the channels that are specified in the [group role](#iot-analytics-connector-req)\.
Display name in console: **Publish region**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|([a-z]{2}-[a-z]+-\\d{1})`

`PublishInterval`  
The interval \(in seconds\) for publishing a batch of received data to AWS IoT Analytics\.  
Display name in console: **Publish interval**  
Required: `false`  
Type: `string`  
Default value: `1`  
Valid pattern: `$|^[0-9]+$`

`IotAnalyticsMaxActiveChannels`  
The maximum number of AWS IoT Analytics channels that the connector actively watches for\. This must be greater than 0, and at least equal to the number of channels that you expect the connector to publish to at a given time\.  
You can use this parameter to restrict memory consumption by limiting the total number of queues that the connector can manage at a given time\. A queue is deleted when all queued messages are sent\.  
Display name in console: **Maximum number of active channels**  
Required: `false`  
Type: `string`  
Default value: `50`  
Valid pattern: `^$|^[1-9][0-9]*$`

`IotAnalyticsQueueDropBehavior`  
The behavior for dropping messages from a channel queue when the queue is full\.  
Display name in console: **Queue drop behavior**  
Required: `false`  
Type: `string`  
Valid values: `DROP_NEWEST` or `DROP_OLDEST`  
Default value: `DROP_NEWEST`  
Valid pattern: `^DROP_NEWEST|DROP_OLDEST$`

`IotAnalyticsQueueSizePerChannel`  
The maximum number of messages to retain in memory \(per channel\) before the messages are submitted or dropped\. This must be greater than 0\.  
Display name in console: **Maximum queue size per channel**  
Required: `false`  
Type: `string`  
Default value: `2048`  
Valid pattern: `^$|^[1-9][0-9]*$`

`IotAnalyticsBatchSizePerChannel`  
The maximum number of messages to send to an AWS IoT Analytics channel in one batch request\. This must be greater than 0\.  
Display name in console: **Maximum number of messages to batch per channel**  
Required: `false`  
Type: `string`  
Default value: `5`  
Valid pattern: `^$|^[1-9][0-9]*$`

`IotAnalyticsDefaultChannelName`  
The name of the AWS IoT Analytics channel that this connector uses for messages that are sent to a customer\-defined input topic\.  
Display name in console: **Default channel name**  
Required: `false`  
Type: `string`  
Valid pattern: `^[a-zA-Z0-9_]$`

### Create Connector Example \(CLI\)<a name="iot-analytics-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the IoT Analytics connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyIoTAnalyticsApplication",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/IoTAnalytics/versions/2",
            "Parameters": {
                "MemorySize": "65535",
                "PublishRegion": "us-west-1",
                "PublishInterval": "2",
                "IotAnalyticsMaxActiveChannels": "25",
                "IotAnalyticsQueueDropBehavior": "DROP_OLDEST",
                "IotAnalyticsQueueSizePerChannel": "1028",
                "IotAnalyticsBatchSizePerChannel": "5",
                "IotAnalyticsDefaultChannelName": "my_channel"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="iot-analytics-connector-data-input"></a>

This connector accepts data on predefined and customer\-defined MQTT topics\. Publishers can be Greengrass devices, Lambda functions, or other connectors\.

Predefined topics  
The connector supports the following two structured MQTT topics that allow publishers to specify the channel name inline\.  
+ A [formatted message](#iot-analytics-connector-data-input-json) on the `iotanalytics/channels/+/messages/put` topic\. The IoT data in these input messages must be formatted as a JSON or base64\-encoded string\.
+ An unformatted message on the `iotanalytics/channels/+/messages/binary/put` topic\. Input messages received on this topic are treated as binary data and can contain any data type\.

   

  To publish to predefined topics, replace the `+` wildcard with the channel name\. For example:

  ```
  iotanalytics/channels/my_channel/messages/put
  ```

Customer\-defined topics  
The connector supports the `#` topic syntax, which allows it to accept input messages on any MQTT topic that you configure in a subscription\. We recommend that you specify a topic path instead of using only the `#` wildcard in your subscriptions\. These messages are sent to the default channel that you specify for the connector\.  
Input messages on customer\-defined topics are treated as binary data\. They can use any message format and can contain any data type\. You can use customer\-defined topics to route messages from devices that publish to fixed topics\. You can also use them to accept input data from devices that can't process the data into a formatted message to send to the connector\.  
For more information about subscriptions and MQTT topics, see [Inputs and Outputs](connectors.md#connectors-inputs-outputs)\.

The group role must allow the `iotanalytics:BatchPutMessage` action on all destination channels\. For more information, see [Requirements](#iot-analytics-connector-req)\.

**Topic filter:** `iotanalytics/channels/+/messages/put`  <a name="iot-analytics-connector-data-input-json"></a>
Use this topic to send formatted messages to the connector and dynamically specify a destination channel\. This topic also allows you to specify an ID that's returned in the response output\. The connector verifies that IDs are unique for each message in the outbound `BatchPutMessage` request that it sends to AWS IoT Analytics\. A message that has a duplicate ID is dropped\.  
Input data sent to this topic must use the following message format\.    
**Message properties**    
`request`  
The data to send to the specified channel\.  
Required: `true`  
Type: `object` that includes the following properties:    
`message`  
The device or sensor data as a JSON or base64\-encoded string\.  
Required: `true`  
Type: `string`  
`id`  
An arbitrary ID for the request\. This property is used to map an input request to an output response\. When specified, the `id` property in the response object is set to this value\. If you omit this property, the connector generates an ID\.  
Required: `false`  
Type: `string`  
Valid pattern: `.*`  
**Example input**  

```
{
    "request": {
        "message" : "{\"temp\":23.33}"
    },
    "id" : "req123"
}
```

**Topic filter:** `iotanalytics/channels/+/messages/binary/put`  
Use this topic to send unformatted messages to the connector and dynamically specify a destination channel\.  
The connector data doesn't parse the input messages received on this topic\. It treats them as binary data\. Before sending the messages to AWS IoT Analytics, the connector encodes and formats them to conform with `BatchPutMessage` API requirements:  
+ The connector base64\-encodes the raw data and includes the encoded payload in an outbound `BatchPutMessage` request\.
+ The connector generates and assigns an ID to each input message\.
**Note**  
The connector's response output doesn't include an ID correlation for these input messages\.  
**Message properties**  
None\.

**Topic filter:** `#`  
Use this topic to send any message format to the default channel\. This is especially useful when your devices publish to fixed topics or when you want to send data to the default channel from devices that can't process the data into the connector's [supported message format](#iot-analytics-connector-data-input-json)\.  
You define the topic syntax in the subscription that you create to connect this connector to the data source\. We recommend that you specify a topic path instead of using only the `#` wildcard in your subscriptions\.  
The connector data doesn't parse the messages that are published to this input topic\. All input messages are treated as binary data\. Before sending the messages to AWS IoT Analytics, the connector encodes and formats them to conform with `BatchPutMessage` API requirements:  
+ The connector base64\-encodes the raw data and includes the encoded payload in an outbound `BatchPutMessage` request\.
+ The connector generates and assigns an ID to each input message\.
**Note**  
The connector's response output doesn't include an ID correlation for these input messages\.  
**Message properties**  
None\.

## Output Data<a name="iot-analytics-connector-data-output"></a>

This connector publishes status information as output data\. This information contains the response returned by AWS IoT Analytics for each input message that it receives and sends to AWS IoT Analytics\.

**Topic filter**  
`iotanalytics/messages/put/status`

**Example output: Success**  

```
{
    "response" : {
        "status" : "success"
    },
    "id" : "req123"
}
```

**Example output: Failure**  

```
{
    "response" : {
        "status" : "fail",
        "error" : "ResourceNotFoundException",
        "error_message" : "A resource with the specified name could not be found."
    },
    "id" : "req123"
}
```
If the connector detects a retryable error \(for example, connection errors\), it retries the publish in the next batch\. Exponential backoff is handled by the AWS SDK\. Requests with retryable errors are added back to the channel queue for further publishing according to the `IotAnalyticsQueueDropBehavior` parameter\.

## Limits<a name="iot-analytics-connector-limits"></a>

This connector is subject to the following limits\.
+ All limits imposed by the AWS SDK for Python \(boto3\) for the AWS IoT Analytics [ `batch_put_message`](http://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iotanalytics.html#IoTAnalytics.Client.batch_put_message) action\.
+ All quotas imposed by the AWS IoT Analytics [ BatchPutMessage](https://docs.aws.amazon.com/iotanalytics/latest/userguide/api.html#cli-iotanalytics-batchputmessage) API\. For more information, see [ Service Quotas](https://docs.aws.amazon.com/general/latest/gr/iot-analytics.html#limits_iot_analytics) for AWS IoT Analytics in the *AWS General Reference*\.
  + 100,000 messages per second per channel\.
  + 100 messages per batch\.
  + 128 KB per message\.

  This API uses channel names \(not channel ARNs\), so sending data to cross\-region or cross\-account channels is not supported\.
+ All quotas imposed by the AWS IoT Greengrass Core\. For more information, see [ Service Quotas](https://docs.aws.amazon.com/general/latest/gr/greengrass.html#limits_greengrass) for the AWS IoT Greengrass core in the *AWS General Reference*\.

  The following quotas might be especially applicable:
  + Maximum size of messages sent by a device is 128 KB\.
  + Maximum message queue size in the Greengrass core router is 2\.5 MB\.
  + Maximum length of a topic string is 256 bytes of UTF\-8 encoded characters\.
+ This connector can be used only in AWS Regions that are supported by both [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/rande.html#greengrass_region) and [AWS IoT Analytics](https://docs.aws.amazon.com/general/latest/gr/rande.html#iotanalytics_region)\. Currently, this includes the following Regions:
  + US East \(Ohio\) \- us\-east\-2
  + US East \(N\. Virginia\) \- us\-east\-1
  + US West \(Oregon\) \- us\-west\-2
  + Asia Pacific \(Tokyo\) \- ap\-northeast\-1
  + Europe \(Frankfurt\) \- eu\-central\-1
  + Europe \(Ireland\) \- eu\-west\-1

## Licenses<a name="iot-analytics-connector-license"></a>

The IoT Analytics connector includes the following third\-party software/licensing:
+ [AWS SDK for Python \(Boto 3\)](https://github.com/boto/boto3)/Apache 2\.0

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## Changelog<a name="iot-analytics-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 2 | Fix to reduce excessive logging\. | 
| 1 | Initial release\.  | 

A Greengrass group can contain only one version of the connector at a time\.

## See Also<a name="iot-analytics-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)
+  [What Is AWS IoT Analytics?](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html) in the *AWS IoT Analytics User Guide*