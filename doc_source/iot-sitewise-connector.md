# IoT SiteWise Connector<a name="iot-sitewise-connector"></a>

The IoT SiteWise connector sends local device and equipment data to asset properties in AWS IoT SiteWise\. You can use this connector to collect data from multiple OPC\-UA servers and publish it to AWS IoT SiteWise\. The connector sends the data to asset properties in the current AWS account and Region\.

**Note**  
AWS IoT SiteWise is a fully managed service that collects, processes, and visualizes data from industrial devices and equipment\. You can configure asset properties that further process raw data sent from this connector to your assets' measurement properties\. For example, you can define a transform property that converts a device's Celsius temperature data points to Faherenheit, or you can define a metric property that calculates the average hourly temperature\. For more information, see [What Is AWS IoT SiteWise?](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/) in the *AWS IoT SiteWise User Guide*\.

The connector sends data to AWS IoT SiteWise with the OPC\-UA data stream paths sent from the OPC\-UA servers\. For example, the data stream path `/company/windfarm/3/turbine/7/temperature` might represent the temperature sensor of turbine \#7 at wind farm \#3\. If the AWS IoT Greengrass core loses connection to the internet, the connector caches data until it can successfully connect to the AWS Cloud\. You can configure the maximum disk buffer size used for caching data\. If the cache size exceeds the maximum disk buffer size, the connector discards the oldest data from the queue\.

After you configure and deploy the IoT SiteWise connector, you can add a gateway and OPC\-UA sources in the [AWS IoT SiteWise console](https://console.aws.amazon.com/iotsitewise/)\. When you configure a source in the console, you can filter or prefix the OPC\-UA data stream paths sent by the IoT SiteWise connector\. To learn how to finish setting up your gateway and sources, see [Add the Gateway and Configure Sources](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/configure-gateway.html#add-gateway) in the *AWS IoT SiteWise User Guide*\.

AWS IoT SiteWise receives data only from data streams that you have mapped to the measurement properties of AWS IoT SiteWise assets\. To map data streams to asset properties, you can set a property's alias to be equivalent to an OPC\-UA data stream path\. To learn about defining asset models and creating assets, see [Modeling Industrial Assets](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/industrial-asset-models) in the *AWS IoT SiteWise User Guide*\.

**Note**  
This connector runs in [No container](lambda-group-config.md#no-container-mode) isolation mode, so you can deploy it to a Greengrass group running in a Docker container\.

**ARN**: `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/1`

## Requirements<a name="iot-sitewise-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core software v1\.9\.4 or later\.
+ Java 8 Runtime installed on the core device\.
+ This connector can be used only in supported AWS Regions\. For more information, see [Limits](#iot-sitewise-connector-limits)\.
+ An IAM policy added to the Greengrass group role that allows access to AWS IoT Core and the `iotsitewise:BatchPutAssetPropertyValue` action on the target root asset, as shown in the following example\. You can remove the `Condition` from the policy to allow the connector to access all of your AWS IoT SiteWise assets\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
               "Effect": "Allow",
               "Action": "iotsitewise:BatchPutAssetPropertyValue",
               "Resource": "*",
               "Condition": {
                   "StringLike": {
                       "iotsitewise:assetHierarchyPath": [
                           "root node asset ID/*"
                       ]
                   }
               }
          },
          {
              "Effect": "Allow",
              "Action": [
                   "iot:Connect",
                   "iot:DescribeEndpoint",
                   "iot:Publish",
                   "iot:Receive",
                   "iot:Subscribe"
              ],
              "Resource": "*"
          }
      ]
  }
  ```

  For more information, see [Adding and Removing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

## Parameters<a name="iot-sitewise-connector-param"></a>

`SiteWiseLocalStoragePath`  
The folder on the AWS IoT Greengrass host that the IoT SiteWise connector can write persistent data to\. Defaults to `/var/sitewise`\.  
Display name in console: **Local storage path**  
Required: `false`  
Type: `string`  
Valid pattern: `^\\s*$|\\/.`

`SiteWiseOpcuaUserIdentityTokenSecretArn`  
The secret in AWS Secrets Manager that contains the OPC\-UA user name and password key\-value pair\. This secret must be a key\-value pair type secret\.  
Display name in console: **ARN of OPC\-UA username/password secret**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|arn:(aws(-[a-z]+)*):secretsmanager:[a-z0-9\\-]+:[0-9]{12}:secret:([a-zA-Z0-9\\\\]+/)*[a-zA-Z0-9/_+=,.@\\-]+-[a-zA-Z0-9]+`

`SiteWiseOpcuaUserIdentityTokenSecretArn-ResourceId`  
The secret resource in the AWS IoT Greengrass group that references an OPC\-UA user name and password secret\.  
Display name in console: **OPC\-UA username/password secret resource**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|.+`

`MaximumBufferSize`  
The maximum size in GB for IoT SiteWise disk usage\. Defaults to 10GB\.  
Display name in console: **Maximum disk buffer size**  
Required: `false`  
Type: `string`  
Valid pattern: `^\\s*$|[0-9]+`

### Create Connector Example \(CLI\)<a name="iot-sitewise-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the IoT SiteWise connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyIoTSiteWiseConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/1"
        }
    ]
}'
```

**Note**  
The Lambda functions in this connector have a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="iot-sitewise-connector-data-input"></a>

This connector doesn't accept MQTT messages as input data\.

## Output Data<a name="iot-sitewise-connector-data-output"></a>

This connector doesn't publish MQTT messages as output data\.

## Limits<a name="iot-sitewise-connector-limits"></a>

This connector is subject to the following limits\.
+ All limits imposed by AWS IoT SiteWise, including the following\. For more information, see [ AWS IoT SiteWise Limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#limits_iot_sitewise) in the *AWS General Reference*\. 
  + Maximum number of gateways per AWS account\.
  + Maximum number of OPC\-UA sources per gateway\.
  + Maximum rate of timestamp\-quality\-value \(TQV\) data points stored per AWS account\.
  + Maximum rate of TQV data points stored per asset property\.
+ This connector can be used only in AWS Regions where both [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/rande.html#greengrass_region) and [AWS IoT SiteWise](https://docs.aws.amazon.com/general/latest/gr/rande.html#iotsitewise_region) are supported\. Currently, this includes the following Regions:
  + US East \(N\. Virginia\) \- `us-east-1`
  + US West \(Oregon\) \- `us-west-2`
  + EU \(Frankfurt\) \- `eu-central-1`

## Licenses<a name="iot-sitewise-connector-license"></a>

The IoT SiteWise connector includes the following third\-party software/licensing:
+ [Milo](https://github.com/eclipse/milo/) / EDL 1\.0
+ [Chronicle](https://github.com/OpenHFT/Chronicle-Queue) / Apache 2\.0

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## See Also<a name="iot-sitewise-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)
+ [What Is AWS IoT SiteWise?](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/) in the *AWS IoT SiteWise User Guide*
+ [Use a Gateway Connector](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/gateway-connector.html) in the *AWS IoT SiteWise User Guide*
+ [Troubleshooting an AWS IoT SiteWise Gateway](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/troubleshooting.html#troubleshooting-gateway) in the *AWS IoT SiteWise User Guide*