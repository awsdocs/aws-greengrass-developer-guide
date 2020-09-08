# ServiceNow MetricBase Integration connector<a name="servicenow-connector"></a>

The ServiceNow MetricBase Integration [connector](connectors.md) publishes time series metrics from Greengrass devices to ServiceNow MetricBase\. This allows you to store, analyze, and visualize time series data from the Greengrass core environment, and act on local events\.

This connector receives time series data on an MQTT topic, and publishes the data to the ServiceNow API at regular intervals\.

You can use this connector to support scenarios such as:
+ Create threshold\-based alerts and alarms based on time series data collected from Greengrass devices\.
+ Use time services data from Greengrass devices with custom applications built on the ServiceNow platform\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 4 | `arn:aws:greengrass:region::/connectors/ServiceNowMetricBaseIntegration/versions/4` | 
| 3 | `arn:aws:greengrass:region::/connectors/ServiceNowMetricBaseIntegration/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/ServiceNowMetricBaseIntegration/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/ServiceNowMetricBaseIntegration/versions/1` | 

For information about version changes, see the [Changelog](#servicenow-connector-changelog)\.

## Requirements<a name="servicenow-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 3 \- 4 ]
+ <a name="conn-req-ggc-v1.9.3-secrets"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\. AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This requirement includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-servicenow-req-servicenow-account"></a>A ServiceNow account with an activated subscription to MetricBase\. In addition, a metric and metric table must be created in the account\. For more information, see [MetricBase](https://docs.servicenow.com/bundle/london-servicenow-platform/page/administer/metricbase/concept/metricbase.html) in the ServiceNow documentation\.
+ <a name="conn-servicenow-req-secret"></a>A text type secret in AWS Secrets Manager that stores the user name and password to log in to your ServiceNow instance with basic authentication\. The secret must contain "user" and "password" keys with corresponding values\. For more information, see [Creating a basic secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.
+ A secret resource in the Greengrass group that references the Secrets Manager secret\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

------
#### [ Versions 1 \- 2 ]
+ <a name="conn-req-ggc-v1.7.0-secrets"></a>AWS IoT Greengrass Core software v1\.7 or later\. AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This requirement includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-servicenow-req-servicenow-account"></a>A ServiceNow account with an activated subscription to MetricBase\. In addition, a metric and metric table must be created in the account\. For more information, see [MetricBase](https://docs.servicenow.com/bundle/london-servicenow-platform/page/administer/metricbase/concept/metricbase.html) in the ServiceNow documentation\.
+ <a name="conn-servicenow-req-secret"></a>A text type secret in AWS Secrets Manager that stores the user name and password to log in to your ServiceNow instance with basic authentication\. The secret must contain "user" and "password" keys with corresponding values\. For more information, see [Creating a basic secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.
+ A secret resource in the Greengrass group that references the Secrets Manager secret\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

------

## Connector Parameters<a name="servicenow-connector-param"></a>

This connector provides the following parameters:

------
#### [ Version 4 ]

`PublishInterval`  <a name="service-now-PublishInterval"></a>
The maximum number of seconds to wait between publish events to ServiceNow\. The maximum value is 900\.  
The connector publishes to ServiceNow when `PublishBatchSize` is reached or `PublishInterval` expires\.  
Display name in the AWS IoT console: **Publish interval in seconds**  
Required: `true`  
Type: `string`  
Valid values: `1 - 900`  
Valid pattern: `[1-9]|[1-9]\d|[1-9]\d\d|900`

`PublishBatchSize`  <a name="service-now-PublishBatchSize"></a>
The maximum number of metric values that can be batched before they are published to ServiceNow\.  
The connector publishes to ServiceNow when `PublishBatchSize` is reached or `PublishInterval` expires\.  
Display name in the AWS IoT console: **Publish batch size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`InstanceName`  <a name="service-now-InstanceName"></a>
The name of the instance used to connect to ServiceNow\.  
Display name in the AWS IoT console: **Name of ServiceNow instance**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`DefaultTableName`  <a name="service-now-DefaultTableName"></a>
The name of the table that contains the `GlideRecord` associated with the time series MetricBase database\. The `table` property in the input message payload can be used to override this value\.  
Display name in the AWS IoT console: **Name of the table to contain the metric**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`MaxMetricsToRetain`  <a name="service-now-MaxMetricsToRetain"></a>
The maximum number of metrics to save in memory before they are replaced with new metrics\.  
This limit applies when there's no connection to the internet and the connector starts to buffer the metrics to publish later\. When the buffer is full, the oldest metrics are replaced by new metrics\.  
Metrics are not saved if the host process for the connector is interrupted\. For example, this can happen during group deployment or when the device restarts\.
This value should be greater than the batch size and large enough to hold messages based on the incoming rate of the MQTT messages\.  
Display name in the AWS IoT console: **Maximum metrics to retain in memory**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`AuthSecretArn`  <a name="service-now-AuthSecretArn"></a>
The secret in AWS Secrets Manager that stores the ServiceNow user name and password\. This must be a text type secret\. The secret must contain "user" and "password" keys with corresponding values\.  
Display name in the AWS IoT console: **ARN of auth secret**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:secretsmanager:[a-z0-9\-]+:[0-9]{12}:secret:([a-zA-Z0-9\\]+/)*[a-zA-Z0-9/_+=,.@\-]+-[a-zA-Z0-9]+`

`AuthSecretArn-ResourceId`  <a name="service-now-AuthSecretArn-ResourceId"></a>
The secret resource in the group that references the Secrets Manager secret for the ServiceNow credentials\.  
Display name in the AWS IoT console: **Auth token resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

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

`PublishInterval`  <a name="service-now-PublishInterval"></a>
The maximum number of seconds to wait between publish events to ServiceNow\. The maximum value is 900\.  
The connector publishes to ServiceNow when `PublishBatchSize` is reached or `PublishInterval` expires\.  
Display name in the AWS IoT console: **Publish interval in seconds**  
Required: `true`  
Type: `string`  
Valid values: `1 - 900`  
Valid pattern: `[1-9]|[1-9]\d|[1-9]\d\d|900`

`PublishBatchSize`  <a name="service-now-PublishBatchSize"></a>
The maximum number of metric values that can be batched before they are published to ServiceNow\.  
The connector publishes to ServiceNow when `PublishBatchSize` is reached or `PublishInterval` expires\.  
Display name in the AWS IoT console: **Publish batch size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`InstanceName`  <a name="service-now-InstanceName"></a>
The name of the instance used to connect to ServiceNow\.  
Display name in the AWS IoT console: **Name of ServiceNow instance**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`DefaultTableName`  <a name="service-now-DefaultTableName"></a>
The name of the table that contains the `GlideRecord` associated with the time series MetricBase database\. The `table` property in the input message payload can be used to override this value\.  
Display name in the AWS IoT console: **Name of the table to contain the metric**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`MaxMetricsToRetain`  <a name="service-now-MaxMetricsToRetain"></a>
The maximum number of metrics to save in memory before they are replaced with new metrics\.  
This limit applies when there's no connection to the internet and the connector starts to buffer the metrics to publish later\. When the buffer is full, the oldest metrics are replaced by new metrics\.  
Metrics are not saved if the host process for the connector is interrupted\. For example, this can happen during group deployment or when the device restarts\.
This value should be greater than the batch size and large enough to hold messages based on the incoming rate of the MQTT messages\.  
Display name in the AWS IoT console: **Maximum metrics to retain in memory**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`AuthSecretArn`  <a name="service-now-AuthSecretArn"></a>
The secret in AWS Secrets Manager that stores the ServiceNow user name and password\. This must be a text type secret\. The secret must contain "user" and "password" keys with corresponding values\.  
Display name in the AWS IoT console: **ARN of auth secret**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:secretsmanager:[a-z0-9\-]+:[0-9]{12}:secret:([a-zA-Z0-9\\]+/)*[a-zA-Z0-9/_+=,.@\-]+-[a-zA-Z0-9]+`

`AuthSecretArn-ResourceId`  <a name="service-now-AuthSecretArn-ResourceId"></a>
The secret resource in the group that references the Secrets Manager secret for the ServiceNow credentials\.  
Display name in the AWS IoT console: **Auth token resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

------

### Create Connector Example \(AWS CLI\)<a name="servicenow-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the ServiceNow MetricBase Integration connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyServiceNowMetricBaseIntegrationConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ServiceNowMetricBaseIntegration/versions/4",
            "Parameters": {
                "PublishInterval" : "10",
                "PublishBatchSize" : "50",
                "InstanceName" : "myinstance",
                "DefaultTableName" : "u_greengrass_app",
                "MaxMetricsToRetain" : "20000",
                "AuthSecretArn" : "arn:aws:secretsmanager:region:account-id:secret:greengrass-secret-hash",
                "AuthSecretArn-ResourceId" : "MySecretResource", 
                "IsolationMode" : "GreengrassContainer"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="servicenow-connector-data-input"></a>

This connector accepts time series metrics on an MQTT topic and publishes the metrics to ServiceNow\. Input messages must be in JSON format\.

<a name="topic-filter"></a>**Topic filter in subscription**  
`servicenow/metricbase/metric`

**Message properties**    
`request`  
Information about the table, record, and metric\. This request represents the `seriesRef` object in a time series POST request\. For more information, see [ Clotho Time Series API \- POST](https://docs.servicenow.com/bundle/london-application-development/page/integrate/inbound-rest/concept/Clotho-Time-Series-API.html#clotho-POST-put)\.  
Required: `true`  
Type: `object` that includes the following properties:    
`subject`  
The `sys_id` of the specific record in the table\.  
Required: `true`  
Type: `string`  
`metric_name`  
The metric field name\.  
Required: `true`  
Type: `string`  
`table`  
The name of the table to store the record in\. Specify this value to override the `DefaultTableName` parameter\.  
Required: `false`  
Type: `string`  
`value`  
The value of the individual data point\.  
Required: `true`  
Type: `float`  
`timestamp`  
The timestamp of the individual data point\. The default value is the current time\.  
Required: `false`  
Type: `string`

**Example input**  

```
{
    "request": {
        "subject":"ef43c6d40a0a0b5700c77f9bf387afe3",
        "metric_name":"u_count",
        "table": "u_greengrass_app"
        "value": 1.0,
        "timestamp": "2018-10-14T10:30:00"
    }
}
```

## Output data<a name="servicenow-connector-data-output"></a>

This connector publishes status information as output data on an MQTT topic\.

<a name="topic-filter"></a>**Topic filter in subscription**  
`servicenow/metricbase/metric/status`

**Example output: Success**  

```
{
    "response": {
        "metric_name": "Errors",
        "table_name": "GliderProd",
        "processed_on": "2018-10-14T10:35:00",
        "response_id": "khjKSkj132qwr23fcba",
        "status": "success",
        "values": [
            {
                "timestamp": "2016-10-14T10:30:00",
                "value": 1.0
            },
            {
                "timestamp": "2016-10-14T10:31:00",
                "value": 1.1
            }
        ]
    }
}
```

**Example output: Failure**  

```
{
    "response": {
        "error": "InvalidInputException",
        "error_message": "metric value is invalid",
        "status": "fail"
    }
}
```
If the connector detects a retryable error \(for example, connection errors\), it retries the publish in the next batch\.

## Usage Example<a name="servicenow-connector-usage"></a>

<a name="connectors-setup-intro"></a>Use the following high\-level steps to set up an example Python 3\.7 Lambda function that you can use to try out the connector\.

**Note**  <a name="connectors-setup-get-started-topics"></a>
The [Get started with connectors \(console\)](connectors-console.md) and [Get started with connectors \(CLI\)](connectors-cli.md) topics contain detailed steps that show you how to configure and deploy an example Twilio Notifications connector\.

1. Make sure you meet the [requirements](#servicenow-connector-req) for the connector\.

1. <a name="connectors-setup-function"></a>Create and publish a Lambda function that sends input data to the connector\.

   Save the [example code](#servicenow-connector-usage-example) as a PY file\. <a name="connectors-setup-function-sdk"></a>Download and unzip the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core)\. Then, create a zip package that contains the PY file and the `greengrasssdk` folder at the root level\. This zip package is the deployment package that you upload to AWS Lambda\.

   <a name="connectors-setup-function-publish"></a>After you create the Python 3\.7 Lambda function, publish a function version and create an alias\.

1. Configure your Greengrass group\.

   1. <a name="connectors-setup-gg-function"></a>Add the Lambda function by its alias \(recommended\)\. Configure the Lambda lifecycle as long\-lived \(or `"Pinned": true` in the CLI\)\.

   1. <a name="connectors-setup-secret-resource"></a>Add the required secret resource and grant read access to the Lambda function\.

   1. Add the connector and configure its [parameters](#servicenow-connector-param)\.

   1. Add subscriptions that allow the connector to receive [input data](#servicenow-connector-data-input) and send [output data](#servicenow-connector-data-output) on supported topic filters\.
      + <a name="connectors-setup-subscription-input-data"></a>Set the Lambda function as the source, the connector as the target, and use a supported input topic filter\.
      + <a name="connectors-setup-subscription-output-data"></a>Set the connector as the source, AWS IoT Core as the target, and use a supported output topic filter\. You use this subscription to view status messages in the AWS IoT console\.

1. <a name="connectors-setup-deploy-group"></a>Deploy the group\.

1. <a name="connectors-setup-test-sub"></a>In the AWS IoT console, on the **Test** page, subscribe to the output data topic to view status messages from the connector\. The example Lambda function is long\-lived and starts sending messages immediately after the group is deployed\.

   When you're finished testing, you can set the Lambda lifecycle to on\-demand \(or `"Pinned": false` in the CLI\) and deploy the group\. This stops the function from sending messages\.

### Example<a name="servicenow-connector-usage-example"></a>

The following example Lambda function sends an input message to the connector\.

```
import greengrasssdk
import json

iot_client = greengrasssdk.client('iot-data')
SEND_TOPIC = 'servicenow/metricbase/metric'

def create_request_with_all_fields():
    return {
        "request": {
             "subject": '2efdf6badbd523803acfae441b961961',
             "metric_name": 'u_count',
             "value": 1234,
             "timestamp": '2018-10-20T20:22:20',
             "table": 'u_greengrass_metricbase_test'
        }
    }

def publish_basic_message():
    messageToPublish = create_request_with_all_fields()
    print("Message To Publish: ", messageToPublish)
    iot_client.publish(topic=SEND_TOPIC,
        payload=json.dumps(messageToPublish))

publish_basic_message()

def lambda_handler(event, context):
    return
```

## Licenses<a name="servicenow-connector-license"></a>

The ServiceNow MetricBase Integration connector includes the following third\-party software/licensing:
+ [pysnow](https://github.com/rbw/pysnow)/MIT

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="servicenow-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 4 | <a name="isolation-mode-changelog"></a>Added the `IsolationMode` parameter to configure the containerization mode for the connector\. | 
| 3 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 2 | Fix to reduce excessive logging\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="servicenow-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)