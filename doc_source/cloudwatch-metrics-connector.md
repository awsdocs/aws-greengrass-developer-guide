# CloudWatch Metrics<a name="cloudwatch-metrics-connector"></a>

The CloudWatch Metrics [connector](connectors.md) publishes custom metrics from Greengrass devices to Amazon CloudWatch\. The connector provides a centralized infrastructure for publishing CloudWatch metrics, which you can use to monitor and analyze the Greengrass core environment, and act on local events\. For more information, see [Using Amazon CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) in the *Amazon CloudWatch User Guide*\.

This connector receives metric data as MQTT messages\. The connector batches metrics that are in the same namespace and publishes them to CloudWatch at regular intervals\.

**ARN**: `arn:aws:greengrass:region::/connectors/CloudWatchMetrics/versions/1`

## Requirements<a name="cloudwatch-metrics-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ An IAM policy added to the Greengrass group role that allows the `cloudwatch:PutMetricData` action, as shown in the following example\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "Stmt1528133056761",
              "Action": [
                  "cloudwatch:PutMetricData"
              ],
              "Effect": "Allow",
              "Resource": "*"
          }
      ]
  }
  ```

  For more information, see [Adding and Removing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide* and [ Amazon CloudWatch Permissions Reference](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/permissions-reference-cw.html) in the *IAM User Guide*\.

## Connector Parameters<a name="cloudwatch-metrics-connector-param"></a>

This connector provides the following parameters:

`PublishInterval`  
The maximum number of seconds to wait before publishing batched metrics for a given namespace\. The maximum value is 900\. To configure the connector to publish metrics as they are received \(without batching\), specify 0\.  
The connector publishes to CloudWatch after it receives 20 metrics in the same namespace or after the specified interval\.  
The connector doesn't guarantee the order of publish events\.
Display name in console: **Publish interval**  
Required: `true`  
Type: `string`  
Valid values: `0 - 900`  
Valid pattern: `[0-9]|[1-9]\d|[1-9]\d\d|900`

`PublishRegion`  
The AWS Region to post CloudWatch metrics to\. This value overrides the default Greengrass metrics region\. It is required only when posting cross\-region metrics\.  
Display name in console: **Publish region**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|([a-z]{2}-[a-z]+-\d{1})`

`MemorySize`  
The memory \(in KB\) to allocate to the connector\.  
Display name in console: **Memory size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`MaxMetricsToRetain`  
The maximum number of metrics across all namespaces to save in memory before they are replaced with new metrics\. The minimum value is 2000\.  
This limit applies when there's no connection to the internet and the connector starts to buffer the metrics to publish later\. When the buffer is full, the oldest metrics are replaced by new metrics\. Metrics in a given namespace are replaced only by metrics in the same namespace\.  
Metrics are not saved if the host process for the connector is interrupted\. For example, this can happen during group deployment or when the device restarts\.
Display name in console: **Maximum metrics to retain**  
Required: `true`  
Type: `string`  
Valid pattern: `^([2-9]\d{3}|[1-9]\d{4,})$`

### Create Connector Example \(CLI\)<a name="cloudwatch-metrics-connector-create"></a>

The following CLI command creates an `ConnectorDefinition` with an initial version that contains the CloudWatch Metrics connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyCloudWatchMetricsConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/CloudWatchMetrics/versions/1",
            "Parameters": {
                "PublishInterval" : "600",
                "PublishRegion" : "us-west-2",
                "MemorySize" : "16",
                "MaxMetricsToRetain" : "2500"
            }
        }
    ]
}'
```

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="cloudwatch-metrics-connector-data-input"></a>

This connector accepts metrics on an MQTT topic and publishes the metrics to CloudWatch\. Input messages must be in JSON format\.

**Topic filter**  
`cloudwatch/metric/put`

**Message properties**    
`request`  
Information about the metric in this message\.  
The request object contains the metric data to publish to CloudWatch\. The metric values must meet the specifications of the [ `PutMetricData`](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html) API\. Only the `namespace`, `metricData.metricName`, and `metricData.value` properties are required\.  
Required: `true`  
Type: `object` that includes the following properties:    
`namespace`  
The user\-defined namespace for the metric data in this request\. CloudWatch uses namespaces as containers for metric data points\.  
You can't specify a namespace that begins with the reserved string "AWS/"\.
Required: `true`  
Type: `string`  
Valid pattern: `[^:].*`  
`metricData`  
The data for the metric\.  
Required: `true`  
Type: `object` that includes the following properties:    
`metricName`  
The name of the metric\.  
Required: `true`  
Type: `string`  
`dimensions`  
The dimensions that are associated with the metric\. Dimensions provide more information about the metric and its data\. A metric can define up to 10 dimensions\.  
Required: `false`  
Type: `array` of dimension objects that include the following properties:    
`name`  
The dimension name\.  
Required: `false`  
Type: `string`  
`value`  
The dimension value\.  
Required: `false`  
Type: `string`  
`timestamp`  
The time that the metric data was received, expressed as the number of milliseconds since `Jan 1, 1970 00:00:00 UTC`\. If this value is omitted, the connector uses the time that it received the message\.  
Required: `false`  
Type: `timestamp`  
`value`  
The value for the metric\.  
CloudWatch rejects values that are too small or too large\. Values must be in the range of `8.515920e-109` to `1.174271e+108` \(Base 10\) or `2e-360` to `2e360` \(Base 2\)\. Special values \(for example, `NaN`, `+Infinity`, `-Infinity`\) are not supported\.
Required: `true`  
Type: `double`  
`unit`  
The unit of the metric\.  
Required: `false`  
Type: `string`  
Valid values: `Seconds, Microseconds, Milliseconds, Bytes, Kilobytes, Megabytes, Gigabytes, Terabytes, Bits, Kilobits, Megabits, Gigabits, Terabits, Percent, Count, Bytes/Second, Kilobytes/Second, Megabytes/Second, Gigabytes/Second, Terabytes/Second, Bits/Second, Kilobits/Second, Megabits/Second, Gigabits/Second, Terabits/Second, Count/Second, None`

Limits  
All limits that are imposed by the CloudWatch [ `PutMetricData`](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html) API apply to metrics when using this connector\. The following limits are especially important:  
+ 40 KB limit on API payload
+ 20 metrics per API request
+ 150 transactions per second \(TPS\) for the `PutMetricData` API
For more information, see [CloudWatch Limits](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_limits.html) in the *Amazon CloudWatch User Guide*\.

**Example input**  

```
{
   "request": {
       "namespace": "Greengrass",
       "metricData":
           {
               "metricName": "latency",
               "dimensions": [
                   {
                       "name": "hostname",
                       "value": "test_hostname"
                   }
               ],
               "timestamp": 1539027324,
               "value": 123.0,
               "unit": "Seconds"
            }
    }
}
```

## Output Data<a name="cloudwatch-metrics-connector-data-output"></a>

This connector publishes status information as output data\.

**Topic filter**  
`cloudwatch/metric/put/status`

**Example output: Success**  
The response includes the namespace of the metric data and the `RequestId` field from the CloudWatch response\.  

```
{
   "response": {
        "cloudwatch_rid":"70573243-d723-11e8-b095-75ff2EXAMPLE",
        "namespace": "Greengrass",
        "status":"success"
    }
}
```

**Example output: Failure**  

```
{
   "response" : {
        "namespace": "Greengrass",
        "error": "InvalidInputException",
        "error_message":"cw metric is invalid",
        "status":"fail"
   }
}
```
If the connector detects a retryable error \(for example, connection errors\), it retries the publish in the next batch\.

## Usage Example<a name="cloudwatch-metrics-connector-usage"></a>

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
send_topic = 'cloudwatch/metric/put'

def create_request_with_all_fields():
    return  {
        "request": {
            "namespace": "Greengrass_CW_Connector",
            "metricData": {
                "metricName": "Count1",
                "dimensions": [
                    {
                        "name": "test",
                        "value": "test"
                    }
                ],
                "value": 1,
                "unit": "Seconds",
                "timestamp": time.time()
            }
        }
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

## Licenses<a name="cloudwatch-metrics-connector-license"></a>

The CloudWatch Metrics connector includes the following third\-party software/licensing:
+ [AWS SDK for Python \(Boto 3\)](https://github.com/boto/boto3) / Apache 2\.0

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## See Also<a name="cloudwatch-metrics-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)
+ [ Using Amazon CloudWatch Metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) in the *Amazon CloudWatch User Guide*
+ [ PutMetricData](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html) in the *Amazon CloudWatch API Reference*