# ServiceNow MetricBase Integration<a name="servicenow-connector"></a>

The ServiceNow MetricBase Integration [connector](connectors.md) publishes time series metrics from Greengrass devices to ServiceNow MetricBase\. This allows you to store, analyze, and visualize time series data from the Greengrass core environment, and act on local events\.

This connector receives time series data on an MQTT topic, and publishes the data to the ServiceNow API at regular intervals\.

You can use this connector to support scenarios such as:
+ Create threshold\-based alerts and alarms based on time series data collected from Greengrass devices\.
+ Use time services data from Greengrass devices with custom applications built on the ServiceNow platform\.

**ARN**: `arn:aws:greengrass:region::/connectors/ServiceNowMetricBaseIntegration/versions/1`

## Requirements<a name="servicenow-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\. AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ A ServiceNow account with an activated subscription to MetricBase\. In addition, a metric and metric table must be created in the account\. For more information, see [MetricBase](https://docs.servicenow.com/bundle/london-servicenow-platform/page/administer/metricbase/concept/metricbase.html) in the ServiceNow documentation\.
+ A text type secret in AWS Secrets Manager that stores the user name and password for your ServiceNow instance \(for basic authentication\)\. The secret must contain "user" and "password" keys with corresponding values\. For more information, see [Creating a Basic Secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.
+ A secret resource in the Greengrass group that references the Secrets Manager secret\. For more information, see [Deploy Secrets to the AWS IoT Greengrass Core](secrets.md)\.

## Connector Parameters<a name="servicenow-connector-param"></a>

This connector provides the following parameters:

`PublishInterval`  
The maximum number of seconds to wait between publish events to ServiceNow\. The maximum value is 900\.  
The connector publishes to ServiceNow when `PublishBatchSize` is reached or `PublishInterval` expires\.  
Display name in console: **Publish interval in seconds**  
Required: `true`  
Type: `string`  
Valid values: `1 - 900`  
Valid pattern: `[1-9]|[1-9]\d|[1-9]\d\d|900`

`PublishBatchSize`  
The maximum number of metric values that can be batched before they are published to ServiceNow\.  
The connector publishes to ServiceNow when `PublishBatchSize` is reached or `PublishInterval` expires\.  
Display name in console: **Publish batch size**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`InstanceName`  
The name of the instance used to connect to ServiceNow\.  
Display name in console: **Name of ServiceNow instance**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`DefaultTableName`  
The name of the table that contains the `GlideRecord` associated with the time series MetricBase database\. The `table` property in the input message payload can be used to override this value\.  
Display name in console: **Name of the table to contain the metric**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`MaxMetricsToRetain`  
The maximum number of metrics to save in memory before they are replaced with new metrics\.  
This limit applies when there's no connection to the internet and the connector starts to buffer the metrics to publish later\. When the buffer is full, the oldest metrics are replaced by new metrics\.  
Metrics are not saved if the host process for the connector is interrupted\. For example, this can happen during group deployment or when the device restarts\.
This value should be greater than the batch size and large enough to hold messages based on the incoming rate of the MQTT messages\.  
Display name in console: **Maximum metrics to retain in memory**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]+$`

`AuthSecretArn`  
The secret in AWS Secrets Manager that stores the ServiceNow user name and password\. This must be a text type secret\. The secret must contain "user" and "password" keys with corresponding values\.  
Display name in console: **ARN of auth secret**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:secretsmanager:[a-z0-9\-]+:[0-9]{12}:secret:([a-zA-Z0-9\\]+/)*[a-zA-Z0-9/_+=,.@\-]+-[a-zA-Z0-9]+`

`AuthSecretArn-ResourceId`  
The secret resource in the group that references the Secrets Manager secret for the ServiceNow credentials\.  
Display name in console: **Auth token resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

### Create Connector Example \(CLI\)<a name="servicenow-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the ServiceNow MetricBase Integration connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyServiceNowMetricBaseIntegrationConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ServiceNowMetricBaseIntegration/versions/1",
            "Parameters": {
                "PublishInterval" : "10",
                "PublishBatchSize" : "50",
                "InstanceName" : "myinstance",
                "DefaultTableName" : "u_greengrass_app",
                "MaxMetricsToRetain" : "20000",
                "AuthSecretArn" : "arn:aws:secretsmanager:region:account-id:secret:greengrass-secret-hash",
                "AuthSecretArn-ResourceId" : "MySecretResource"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="servicenow-connector-data-input"></a>

This connector accepts time series metrics on an MQTT topic and publishes the metrics to ServiceNow\. Input messages must be in JSON format\.

**Topic filter**  
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

## Output Data<a name="servicenow-connector-data-output"></a>

This connector publishes status information as output data\.

**Topic filter**  
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

The following example Lambda function sends an input message to the connector\.

**Note**  
This Python function uses the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) to publish an MQTT message\. You can use the following [pip](https://pypi.org/project/pip/) command to install the Python version of the SDK on your core device:   

```
pip install greengrasssdk
```

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
    print "Message To Publish: ", messageToPublish
    iot_client.publish(topic=SEND_TOPIC,
        payload=json.dumps(messageToPublish))

publish_basic_message()

def function_handler(event, context):
    return
```

## Licenses<a name="servicenow-connector-license"></a>

The ServiceNow MetricBase Integration connector includes the following third\-party software/licensing:
+ [pysnow](https://github.com/rbw/pysnow) / MIT

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## See Also<a name="servicenow-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)