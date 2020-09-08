# CloudWatch Metrics connector<a name="cloudwatch-metrics-connector"></a>

The CloudWatch Metrics [connector](connectors.md) publishes custom metrics from Greengrass devices to Amazon CloudWatch\. The connector provides a centralized infrastructure for publishing CloudWatch metrics, which you can use to monitor and analyze the Greengrass core environment, and act on local events\. For more information, see [Using Amazon CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) in the *Amazon CloudWatch User Guide*\.

This connector receives metric data as MQTT messages\. The connector batches metrics that are in the same namespace and publishes them to CloudWatch at regular intervals\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 4 | `arn:aws:greengrass:region::/connectors/CloudWatchMetrics/versions/4` | 
| 3 | `arn:aws:greengrass:region::/connectors/CloudWatchMetrics/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/CloudWatchMetrics/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/CloudWatchMetrics/versions/1` | 

For information about version changes, see the [Changelog](#cloudwatch-metrics-connector-changelog)\.

## Requirements<a name="cloudwatch-metrics-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 3 \- 4 ]
+ <a name="conn-req-ggc-v1.9.3"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-cloudwatch-metrics-req-iam-policy"></a>The [Greengrass group role](group-role.md) configured to allow the `cloudwatch:PutMetricData` action, as shown in the following example IAM policy\.

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

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

  For more information about CloudWatch permissions, see [ Amazon CloudWatch permissions reference](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/permissions-reference-cw.html) in the *IAM User Guide*\.

------
#### [ Versions 1 \- 2 ]
+ <a name="conn-req-ggc-v1.7.0"></a>AWS IoT Greengrass Core software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-cloudwatch-metrics-req-iam-policy"></a>The [Greengrass group role](group-role.md) configured to allow the `cloudwatch:PutMetricData` action, as shown in the following example IAM policy\.

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

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

  For more information about CloudWatch permissions, see [ Amazon CloudWatch permissions reference](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/permissions-reference-cw.html) in the *IAM User Guide*\.

------

## Connector Parameters<a name="cloudwatch-metrics-connector-param"></a>

This connector provides the following parameters:

------
#### [ Version 4 ]

`PublishInterval`  <a name="cw-metrics-PublishInterval"></a>
The maximum number of seconds to wait before publishing batched metrics for a given namespace\. The maximum value is 900\. To configure the connector to publish metrics as they are received \(without batching\), specify 0\.  
The connector publishes to CloudWatch after it receives 20 metrics in the same namespace or after the specified interval\.  
The connector doesn't guarantee the order of publish events\.
Display name in the AWS IoT console: **Publish interval**  
Required: `true`  
Type: `string`  
Valid values: `0 - 900`  
Valid pattern: `[0-9]|[1-9]\d|[1-9]\d\d|900`

`PublishRegion`  <a name="cw-metrics-PublishRegion"></a>
The AWS Region to post CloudWatch metrics to\. This value overrides the default Greengrass metrics Region\. It is required only when posting cross\-Region metrics\.  
Display name in the AWS IoT console: **Publish region**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|([a-z]{2}-[a-z]+-\d{1})`

`MemorySize`  <a name="cw-metrics-MemorySize"></a>
The memory \(in KB\) to allocate to the connector\.  
Display name in the AWS IoT console: **Memory size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`MaxMetricsToRetain`  <a name="cw-metrics-MaxMetricsToRetain"></a>
The maximum number of metrics across all namespaces to save in memory before they are replaced with new metrics\. The minimum value is 2000\.  
This limit applies when there's no connection to the internet and the connector starts to buffer the metrics to publish later\. When the buffer is full, the oldest metrics are replaced by new metrics\. Metrics in a given namespace are replaced only by metrics in the same namespace\.  
Metrics are not saved if the host process for the connector is interrupted\. For example, this interruption can happen during group deployment or when the device restarts\.
Display name in the AWS IoT console: **Maximum metrics to retain**  
Required: `true`  
Type: `string`  
Valid pattern: `^([2-9]\d{3}|[1-9]\d{4,})$`

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

`PublishInterval`  <a name="cw-metrics-PublishInterval"></a>
The maximum number of seconds to wait before publishing batched metrics for a given namespace\. The maximum value is 900\. To configure the connector to publish metrics as they are received \(without batching\), specify 0\.  
The connector publishes to CloudWatch after it receives 20 metrics in the same namespace or after the specified interval\.  
The connector doesn't guarantee the order of publish events\.
Display name in the AWS IoT console: **Publish interval**  
Required: `true`  
Type: `string`  
Valid values: `0 - 900`  
Valid pattern: `[0-9]|[1-9]\d|[1-9]\d\d|900`

`PublishRegion`  <a name="cw-metrics-PublishRegion"></a>
The AWS Region to post CloudWatch metrics to\. This value overrides the default Greengrass metrics Region\. It is required only when posting cross\-Region metrics\.  
Display name in the AWS IoT console: **Publish region**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|([a-z]{2}-[a-z]+-\d{1})`

`MemorySize`  <a name="cw-metrics-MemorySize"></a>
The memory \(in KB\) to allocate to the connector\.  
Display name in the AWS IoT console: **Memory size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`MaxMetricsToRetain`  <a name="cw-metrics-MaxMetricsToRetain"></a>
The maximum number of metrics across all namespaces to save in memory before they are replaced with new metrics\. The minimum value is 2000\.  
This limit applies when there's no connection to the internet and the connector starts to buffer the metrics to publish later\. When the buffer is full, the oldest metrics are replaced by new metrics\. Metrics in a given namespace are replaced only by metrics in the same namespace\.  
Metrics are not saved if the host process for the connector is interrupted\. For example, this interruption can happen during group deployment or when the device restarts\.
Display name in the AWS IoT console: **Maximum metrics to retain**  
Required: `true`  
Type: `string`  
Valid pattern: `^([2-9]\d{3}|[1-9]\d{4,})$`

------

### Create Connector Example \(AWS CLI\)<a name="cloudwatch-metrics-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the CloudWatch Metrics connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyCloudWatchMetricsConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/CloudWatchMetrics/versions/4",
            "Parameters": {
                "PublishInterval" : "600",
                "PublishRegion" : "us-west-2",
                "MemorySize" : "16",
                "MaxMetricsToRetain" : "2500",
                "IsolationMode" : "GreengrassContainer"
            }
        }
    ]
}'
```

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="cloudwatch-metrics-connector-data-input"></a>

This connector accepts metrics on an MQTT topic and publishes the metrics to CloudWatch\. Input messages must be in JSON format\.

<a name="topic-filter"></a>**Topic filter in subscription**  
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
For more information, see [CloudWatch limits](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_limits.html) in the *Amazon CloudWatch User Guide*\.

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

## Output data<a name="cloudwatch-metrics-connector-data-output"></a>

This connector publishes status information as output data on an MQTT topic\.

<a name="topic-filter"></a>**Topic filter in subscription**  
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

<a name="connectors-setup-intro"></a>Use the following high\-level steps to set up an example Python 3\.7 Lambda function that you can use to try out the connector\.

**Note**  <a name="connectors-setup-get-started-topics"></a>
The [Get started with connectors \(console\)](connectors-console.md) and [Get started with connectors \(CLI\)](connectors-cli.md) topics contain detailed steps that show you how to configure and deploy an example Twilio Notifications connector\.

1. Make sure you meet the [requirements](#cloudwatch-metrics-connector-req) for the connector\.

   <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

1. <a name="connectors-setup-function"></a>Create and publish a Lambda function that sends input data to the connector\.

   Save the [example code](#cloudwatch-metrics-connector-usage-example) as a PY file\. <a name="connectors-setup-function-sdk"></a>Download and unzip the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core)\. Then, create a zip package that contains the PY file and the `greengrasssdk` folder at the root level\. This zip package is the deployment package that you upload to AWS Lambda\.

   <a name="connectors-setup-function-publish"></a>After you create the Python 3\.7 Lambda function, publish a function version and create an alias\.

1. Configure your Greengrass group\.

   1. <a name="connectors-setup-gg-function"></a>Add the Lambda function by its alias \(recommended\)\. Configure the Lambda lifecycle as long\-lived \(or `"Pinned": true` in the CLI\)\.

   1. Add the connector and configure its [parameters](#cloudwatch-metrics-connector-param)\.

   1. Add subscriptions that allow the connector to receive [input data](#cloudwatch-metrics-connector-data-input) and send [output data](#cloudwatch-metrics-connector-data-output) on supported topic filters\.
      + <a name="connectors-setup-subscription-input-data"></a>Set the Lambda function as the source, the connector as the target, and use a supported input topic filter\.
      + <a name="connectors-setup-subscription-output-data"></a>Set the connector as the source, AWS IoT Core as the target, and use a supported output topic filter\. You use this subscription to view status messages in the AWS IoT console\.

1. <a name="connectors-setup-deploy-group"></a>Deploy the group\.

1. <a name="connectors-setup-test-sub"></a>In the AWS IoT console, on the **Test** page, subscribe to the output data topic to view status messages from the connector\. The example Lambda function is long\-lived and starts sending messages immediately after the group is deployed\.

   When you're finished testing, you can set the Lambda lifecycle to on\-demand \(or `"Pinned": false` in the CLI\) and deploy the group\. This stops the function from sending messages\.

### Example<a name="cloudwatch-metrics-connector-usage-example"></a>

The following example Lambda function sends an input message to the connector\.

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
    print("Message To Publish: ", messageToPublish)
    iot_client.publish(topic=send_topic,
        payload=json.dumps(messageToPublish))

publish_basic_message()

def lambda_handler(event, context):
    return
```

## Licenses<a name="cloudwatch-metrics-connector-license"></a>

The CloudWatch Metrics connector includes the following third\-party software/licensing:<a name="boto-3-licenses"></a>
+ [AWS SDK for Python \(Boto3\)](https://pypi.org/project/boto3/)/Apache License 2\.0
+ [botocore](https://pypi.org/project/botocore/)/Apache License 2\.0
+ [dateutil](https://pypi.org/project/python-dateutil/1.4/)/PSF License
+ [docutils](https://pypi.org/project/docutils/)/BSD License, GNU General Public License \(GPL\), Python Software Foundation License, Public Domain
+ [jmespath](https://pypi.org/project/jmespath/)/MIT License
+ [s3transfer](https://pypi.org/project/s3transfer/)/Apache License 2\.0
+ [urllib3](https://pypi.org/project/urllib3/)/MIT License

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="cloudwatch-metrics-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 4 | <a name="isolation-mode-changelog"></a>Added the `IsolationMode` parameter to configure the containerization mode for the connector\. | 
| 3 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 2 | Fix to reduce excessive logging\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="cloudwatch-metrics-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)
+ [ Using Amazon CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) in the *Amazon CloudWatch User Guide*
+ [ PutMetricData](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_PutMetricData.html) in the *Amazon CloudWatch API Reference*