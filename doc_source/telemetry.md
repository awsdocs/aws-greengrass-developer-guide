# Gathering system health telemetry data from AWS IoT Greengrass core devices<a name="telemetry"></a>

System health telemetry data is diagnostic data that can help you monitor the performance of critical operations on your Greengrass core devices\. The telemetry agent on the Greengrass core collects local telemetry data and publishes it to AWS Cloud without any customer interaction\.

You can create projects and applications to retrieve, analyze, transform, and report telemetry data from your edge devices\. Domain experts, such as process engineers, can use these applications to quickly gain insights into fleet health\.

To ensure that the Greengrass edge components function properly, AWS IoT Greengrass uses the data for development and quality improvement purposes\. This feature also helps inform new and enhanced edge capabilities\. AWS IoT Greengrass only retains telemetry data for up to seven days\.

This feature is available in AWS IoT Greengrass Core software v1\.11\.0 and is enabled by default for all Greengrass cores, including existing cores\. You automatically start receiving data as soon as you upgrade to AWS IoT Greengrass Core software v1\.11\.0 or later\.

For information about how to access or manage published telemetry data, see [Subscribing to receive telemetry data](#subscribe-for-telemetry-data)\.

The telemetry agent collects and publishes the following system metrics\.


**Telemetry metrics**  

| Name | Description | Source | 
| --- | --- | --- | 
|  `SystemMemUsage`  |  The amount of memory currently in use by all applications on the Greengrass core device, including the operating system\.  |  System  | 
|  `CpuUsage`  |  The amount of CPU currently in use by all applications on the Greengrass core device, including the operating system\.  |  System  | 
|  `TotalNumberOfFDs`  |  The number of file descriptors stored by the operating system of the Greengrass core device\. One file descriptor uniquely identifies one open file\.  |  System  | 
|  `LambdaOutOfMemory`  |  The number of executions that result in the Lambda function running out of memory\.  |  System  | 
|  `DroppedMessageCount`  |  The number of dropped messages that are destined for AWS IoT Core\.  |  `GGCloudSpooler` system component  | 
|  `LambdaTimeout`  |  The number of timeouts for running the user\-defined Lambda function\.  |  User\-defined Lambda function, AWS Cloud, and system  | 
|  `LambdaUngracefullyKilled`  |  The number of executions that the user\-defined Lambda function fails to complete\.  |  User\-defined Lambda function, AWS Cloud, and system   | 
|  `LambdaError`  |  The number of executions that result in the user\-defined Lambda function writing error logs\.  |  User\-defined Lambda function, AWS Cloud, and system  | 
|  `BytesAppended`  |  The number of bytes of data appended to stream manager\.  |  `GGStreamManager` system component  | 
|  `BytesUploadtedToIoTAnalytics`  |  The number of bytes of data that stream manager exports to channels in AWS IoT Analytics\.  |  `GGStreamManager` system component  | 
|  `BytesUploadedToKinesis`  |  The number of bytes of data that stream manager exports to streams in Amazon Kinesis Data Streams\.  |  `GGStreamManager` system component  | 
|  `BytesUploadedToIoTSiteWise`  |  The number of bytes of data that stream manager exports to asset properties in AWS IoT SiteWise\.  |  `GGStreamManager` system component  | 
|  `BytesUploadedToS3ExportTaskExecutor`  |  The number of bytes of data that stream manager exports to objects in Amazon S3\.  |  `GGStreamManager` system component  | 
|  `BytesUploadedToHTTP`  |  The number of bytes of data that stream manager exports to HTTP\.  |  `GGStreamManager` system component  | 

## Configuring telemetry settings<a name="configure-telemetry-settings"></a>

Greengrass telemetry uses the following default settings:
+ The telemetry agent aggregates telemetry data every hour\.
+ The telemetry agent publishes a telemetry message every 24 hours\.

You can enable or disable the telemetry feature for a Greengrass core device\. AWS IoT Greengrass uses [shadows](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html) to manage the telemetry configuration\. Your changes take effect immediately when the core has a connection to AWS IoT Core\.

The telemetry agent publishes data using the MQTT protocol with a quality of service \(QoS\) level of 0, which means that it doesn't confirm delivery or retry publishing attempts\. Telemetry messages share an MQTT connection with other messages for subscriptions destined for AWS IoT Core\.

Aside from your data link costs, the data transfer from the core to AWS IoT Core is free because the agent publishes to an AWS reserved topic\. However, depending on your use case, you might incur costs when you receive or process the data\.

### Requirements<a name="configure-telemetry-settings-reqs"></a>

The following requirements apply, when you configure telemetry settings:
+ You must use AWS IoT Greengrass Core software v1\.11\.0 or later\.
**Note**  
If you're running an earlier version and you don't want to use telemetry, you don't have to do anything\.
+ You must provide IAM permissions to update the core \(thing\) shadow and to call the configuration APIs before you update telemetry settings\.

  The following example IAM policy lets you manage the shadow and runtime configuration of a specific core:

  ```
  {
      "Version": "2012-10-17",
      "Statement": [        
          {
              "Sid": "AllowManageShadow",
              "Effect": "Allow",
              "Action": [
                  "iot:GetThingShadow",
                  "iot:UpdateThingShadow",
                  "iot:DeleteThingShadow", 
                  "iot:DescribeThing"
              ],
              "Resource": [
                  "arn:aws:iot:region:account-id:thing/core-name-*"
              ]
          },
          {            
              "Sid": "AllowManageRuntimeConfig",
              "Effect": "Allow",
              "Action": [
                  "greengrass:GetCoreRuntimeConfiguration",
                  "greengrass:UpdateCoreRuntimeConfiguration"
              ],
              "Resource": [
                  "arn:aws:iot:region:account-id:thing/core-name"
              ]
          }
      ]
  }
  ```

  <a name="wildcards-grant-granular-conditional-access"></a>You can grant granular or conditional access to resources, for example, by using a wildcard `*` naming scheme\. For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

### Configure telemetry settings \(console\)<a name="configure-telemetry-settings-console"></a>

The following shows how to update the telemetry settings of a Greengrass core in the AWS IoT Greengrass console\.

1. Sign in to the [AWS IoT Greengrass console](https://console.aws.amazon.com/greengrass/)\.

1. In the navigation pane, under **Greengrass**, choose **Groups**\.

1. Under **Greengrass groups**, choose your target group\.

1. In the navigation pane, choose **Cores**\.

1. Under **Cores**, choose your target core\.

1. In the navigation pane, choose **Telemetry**\.

1. On the **System Health Telemetry** page, choose **Edit**\.

1. Under **Configure telemetry**, choose **Enable** or **Disable**, and then choose **Update**\.
**Important**  
By default, the telemetry feature is enabled for AWS IoT Greengrass Core software v1\.11\.0 or later\.

The changes take effect at runtime\. You don't need to deploy the group\.

### Configure telemetry settings \(CLI\)<a name="configure-telemetry-settings-cli"></a>

In the AWS IoT Greengrass API, the `TelemetryConfiguration` object represents the telemetry settings of a Greengrass core\. This object is part of the `RuntimeConfiguration` object associated with the core\. You can use the AWS IoT Greengrass API, AWS CLI, or AWS SDK to manage Greengrass telemetry\. The examples in this section use the AWS CLI\.

**To check telemetry settings**  
The following command gets the telemetry settings of a Greengrass core\.  
+ Replace *core\-thing\-name* with the name of the target core\.

  To get the thing name, you use the [get\-core\-definition\-version](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-core-definition-version.html) command\. The command returns the ARN of the thing that contains the thing name\.

```
aws greengrass get-thing-runtime-configuration --thing-name core-thing-name
```
The command returns a `GetCoreRuntimeConfigurationResponse` object in the JSON response\. For example:  

```
{
    "RuntimeConfiguration": {
        "TelemetryConfiguration": {
            "ConfigurationSyncStatus": "OutOfSync",
            "Telemetry": "On"
        }
    }
}
```

**To configure telemetry settings**  
The following command updates the telemetry settings for a Greengrass core\.  
+ Replace *core\-thing\-name* with the name of the target core\.

  To get the thing name, you use the [get\-core\-definition\-version](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-core-definition-version.html) command\. The command returns the ARN of the thing that contains the thing name\.

```
aws greengrass update-thing-runtime-configuration --thing-name core-thing-name --telemetry-configuration  '{
    "RuntimeConfiguration": {
        "TelemetryConfiguration": {
            "ConfigurationSyncStatus": "InSync",
            "Telemetry": "Off"
        }
    }
}
```

```
aws greengrass update-thing-runtime-configuration --thing-name core-thing-name --telemetry-configuration "{\"TelemetryConfiguration\":{\"ConfigurationSyncStatus\":\"InSync\",\"Telemetry\":\"Off\"}}"
```
Changes to telemetry settings have been applied if the `ConfigurationSyncStatus` is `InSync`\. The changes take effect at runtime\. You don't need to deploy the group\.

#### TelemetryConfiguration object<a name="TelemetryConfiguration-object"></a>

The `TelemetryConfiguration` object has the following properties:

`ConfigurationSyncStatus`  
Checks if telemetry settings are in sync\. You might not make changes to this property\.  
Type: string  
Valid values: `InSync` or `OutOfSync`

`Telemetry`  
Turns telemetry on or off\. The default is `On`\.  
Type: string  
Valid values: `On` or `Off`

## Subscribing to receive telemetry data<a name="subscribe-for-telemetry-data"></a>

You can create rules in Amazon EventBridge that define how to process telemetry data published from the Greengrass core device\. When EventBridge receives the data, it invokes the target actions defined in your rules\. For example, you can create event rules that send notifications, store event information, take corrective action, or invoke other events\.

### Telemetry event<a name="events-message-format"></a>

The event for a deployment state change including the telemetry data uses the following format:

```
{
    "version": "0",
    "id": "f70f943b-9ae2-e7a5-fec4-4c22178a3e6a",
    "detail-type": "Greengrass Telemetry Data",
    "source": "aws.greengrass",
    "account": "123456789012",
    "time": "2020-07-28T20:45:53Z",
    "region": "us-west-1",
    "resources": [],
    "detail": {
        "ThingName": "CoolThing",
        "Schema": "2020-06-30",
        "ADP": [
            {
                "TS": 123231546,
                "NS": "StreamManager",
                "M": [
                    {
                        "N": "BytesAppended|BytesUploadedToKinesis",
                        "Sum": 11,
                        "U": "Bytes"
                    }
                ]
            },
            {
                "TS": 123231546,
                "NS": "StreamManager",
                "M": [
                    {
                        "N": "BytesAppended|BytesUploadedToS3ExportTaskExecutor",
                        "Sum": 11,
                        "U": "Bytes"
                    }
                ]
            },
            {
                "TS": 123231546,
                "NS": "StreamManager",
                "M": [
                    {
                        "N": "BytesAppended|BytesUploadedToHTTP",
                        "Sum": 11,
                        "U": "Bytes"
                    }
                ]
            },
            {
                "TS": 123231546,
                "NS": "StreamManager",
                "M": [
                    {
                        "N": "BytesAppended|BytesUploadedToIoTAnalytics",
                        "Sum": 11,
                        "U": "Bytes"
                    }
                ]
            },
            {
                "TS": 123231546,
                "NS": "StreamManager",
                "M": [
                    {
                        "N": "BytesAppended|BytesUploadedToIoTSiteWise",
                        "Sum": 11,
                        "U": "Bytes"
                    }
                ]
            },
            {
                "TS": 123231546,
                "NS": "arn:aws:lambda:us-west-1:123456789012:function:my-function",
                "M": [
                    {
                        "N": "LambdaTimeout",
                        "Sum": 15,
                        "U": "Count"
                    }
                ]
            },
            {
                "TS": 123231546,
                "NS": "CloudSpooler",
                "M": [
                    {
                        "N": "DroppedMessageCount",
                        "Sum": 15,
                        "U": "Count"
                    }
                ]
            },
            {
                "TS": 1593727692,
                "NS": "SystemMetrics",
                "M": [
                    {
                        "N": "SystemMemUsage",
                        "Sum": 11.23,
                        "U": "Megabytes"
                    },
                    {
                        "N": "CpuUsage",
                        "Sum": 35.63,
                        "U": "Percent"
                    },
                    {
                        "N": "TotalNumberOfFDs",
                        "Sum": 416,
                        "U": "Count"
                    }
                ]
            },
            {
                "TS": 1593727692,
                "NS": "arn:aws:lambda:us-west-1:123456789012:function:my-function",
                "M": [
                    {
                        "N": "LambdaOutOfMemory",
                        "Sum": 12,
                        "U": "Count"
                    },
                    {
                        "N": "LambdaUngracefullyKilled",
                        "Sum": 100,
                        "U": "Count"
                    },
                    {
                        "N": "LambdaError",
                        "Sum": 7,
                        "U": "Count"
                    }
                ]
            }
        ]
    }
}
```

The `ADP` array contains a list of aggregated data points that have the following properties:

`TS`  
Required\. The timestamp of when the data was aggregated\.

`NS`  
Required\. The namespace of the system\.

`M`  
Required\. The list of metrics\. A metric contains the following properties:    
`N`  
The name of the [metric](#telemetry-metrics)\.  
`Sum`  
The aggregated metric value\. The telemetry agent adds new values to the previous total, so the sum is an ever\-increasing value\. You can use the timestamp to find the value of a specific aggregation\. For example, to find the latest aggregated value, subtract the previous timestamped value from the latest timestamped value\.  
`U`  
The unit of the metric value\.

`ThingName`  
Required\. The name of the thing device that you target\.

### Prerequisites for creating EventBridge rules<a name="create-events-rule-prereqs-telemetry"></a>

Before you create an EventBridge rule for AWS IoT Greengrass, you should do the following:
+ Familiarize yourself with events, rules, and targets in EventBridge\.
+ Create and configure the [targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-targets.html) invoked by your EventBridge rules\. Rules can invoke many types of targets, such as Amazon Kinesis streams, AWS Lambda functions, Amazon SNS topics, and Amazon SQS queues\.

  Your EventBridge rule, and the associated targets must be in the AWS Region where you created your Greengrass resources\. For more information, see [Service endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/aws-service-information.html) in the *AWS General Reference*\.

For more information, see [What is Amazon EventBridge?](https://docs.aws.amazon.com/eventbridge/latest/userguide/what-is-amazon-eventbridge.html) and [Getting started with Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-getting-set-up.html) in the *Amazon EventBridge User Guide*\.

### Create an event rule to get telemetry data \(console\)<a name="create-telemetry-event-rule-console"></a>

Use the following steps to use the AWS Management Console to create an EventBridge rule that receives telemetry data published by the Greengrass core\. This allows web servers, email addresses, and other topic subscribers to respond to the event\. For more information, see [Creating a EventBridge rule that triggers on an event from an AWS resource](https://docs.aws.amazon.com/eventbridge/latest/userguide/create-eventbridge-rule.html) in the *Amazon EventBridge User Guide*\.

1. Open the [Amazon EventBridge console](https://console.aws.amazon.com/events/) and choose **Create rule**\.

1. Under **Name and description**, enter a name and description for the rule\.

1. Under **Define pattern**, configure the rule pattern\.

   1. Choose **Event pattern**\.

   1. Choose **Pre\-defined pattern by service**\.

   1. For **Service provider**, choose **AWS**\.

   1. For **Service name**, choose **Greengrass**\.

   1. For **Event type**, select **Greengrass Telemetry Data**\.

1. Under **Select event bus**, keep the default event bus options\.

1. Under **Select targets**, configure your target\. The following example uses an Amazon SQS queue, but you can configure other target types\.

   1. For **Target**, choose **SQS queue**\.

   1. For **Queue\***, choose your target queue\.

1. Under **Tags \- optional**, define tags for the rule or leave the fields empty\.

1. Choose **Create**\.

### Create an event rule to get telemetry data \(CLI\)<a name="create-telemetry-event-rule-cli"></a>

Use the following steps to use the AWS CLI to create an EventBridge rule that receives telemetry data published by the Greengrss core\. This allows web servers, email addresses, and other topic subscribers to respond to the event\.

1. Create the rule\.
   + Replace *thing\-name* with the ID of your Greengrass group\.

     To get the thing name, you use the [get\-core\-definition\-version](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-core-definition-version.html) command\. The command returns the ARN of the thing that contains the thing name\.

   ```
   aws events put-rule \
     --name TestRule \
     --event-pattern "{\"source\": [\"aws.greengrass\"], \"detail\": {\"thing-name\": [\"thing-name\"]}}"
   ```

   Properties that are omitted from the pattern are ignored\.

1. Add the topic as a rule target\. The following example uses Amazon SQS but you can configure other target types\.
   + Replace *queue\-arn* with the ARN of your Amazon SQS queue\.

   ```
   aws events put-targets \
     --rule TestRule \
     --targets "Id"="1","Arn"="queue-arn"
   ```
**Note**  
To allow Amazon EventBridge to invoke your target queue, you must add a resource\-based policy to your topic\. For more information, see [Amazon SNS permissions](https://docs.aws.amazon.com/eventbridge/latest/userguide/resource-based-policies-eventbridge.html#sqs-permissions) in the *Amazon EventBridge User Guide*\.

For more information, see [Events and event patterns in EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-and-event-patterns.html) in the *Amazon EventBridge User Guide*\.

## Troubleshooting AWS IoT Greengrass telemetry<a name="telemetry-troubleshoot"></a>

Use the following information to help troubleshoot issues with configuring AWS IoT Greengrass telemetry\.

### Error: The response contains "ConfigurationStatus": "OutOfSync" after you run the get\-thing\-runtime\-configuration command<a name="telemetry-troubleshoot-ConfigurationStatus-OutOfSync"></a>

**Solutions:**
+ The AWS IoT Device Shadow service takes time to process runtime configuration updates and to deliver the updates to the Greengrass core device\. You might wait and check if telemetry settings are in sync later\.
+ Make sure that your core device is online\.
+ Enable [Amazon CloudWatch Logs in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/cloud-watch-logs.html#viewing-logs) to monitor the shadow\.
+ Use [AWS IoT metrics](https://docs.aws.amazon.com/iot/latest/developerguide/monitoring-cloudwatch.html) to monitor your thing\.