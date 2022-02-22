--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

--------

# IoT SiteWise connector<a name="iot-sitewise-connector"></a>

The IoT SiteWise connector sends local device and equipment data to asset properties in AWS IoT SiteWise\. You can use this connector to collect data from multiple OPC\-UA servers and publish it to IoT SiteWise\. The connector sends the data to asset properties in the current AWS account and Region\.

**Note**  
IoT SiteWise is a fully managed service that collects, processes, and visualizes data from industrial devices and equipment\. You can configure asset properties that process raw data sent from this connector to your assets' measurement properties\. For example, you can define a transform property that converts a device's Celsius temperature data points to Fahrenheit, or you can define a metric property that calculates the average hourly temperature\. For more information, see [What is AWS IoT SiteWise?](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/) in the *AWS IoT SiteWise User Guide*\.

The connector sends data to IoT SiteWise with the OPC\-UA data stream paths sent from the OPC\-UA servers\. For example, the data stream path `/company/windfarm/3/turbine/7/temperature` might represent the temperature sensor of turbine \#7 at wind farm \#3\. If the AWS IoT Greengrass core loses connection to the internet, the connector caches data until it can successfully connect to the AWS Cloud\. You can configure the maximum disk buffer size used for caching data\. If the cache size exceeds the maximum disk buffer size, the connector discards the oldest data from the queue\.

After you configure and deploy the IoT SiteWise connector, you can add a gateway and OPC\-UA sources in the [IoT SiteWise console](https://console.aws.amazon.com/iotsitewise/)\. When you configure a source in the console, you can filter or prefix the OPC\-UA data stream paths sent by the IoT SiteWise connector\. For instructions to finish setting up your gateway and sources, see [Adding the gateway](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/configure-gateway.html#add-gateway) in the *AWS IoT SiteWise User Guide*\.

IoT SiteWise receives data only from data streams that you have mapped to the measurement properties of IoT SiteWise assets\. To map data streams to asset properties, you can set a property's alias to be equivalent to an OPC\-UA data stream path\. To learn about defining asset models and creating assets, see [Modeling industrial assets](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/industrial-asset-models) in the *AWS IoT SiteWise User Guide*\.

**Notes**  
You can use stream manager to upload data to IoT SiteWise from sources other than OPC\-UA servers\. Stream manager also provides customizable support for persistence and bandwidth management\. For more information, see [Manage data streams on the AWS IoT Greengrass core](stream-manager.md)\.  
This connector runs in [No container](lambda-group-config.md#no-container-mode) isolation mode, so you can deploy it to a Greengrass group running in a Docker container\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 12 \(recommended\) | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/12` | 
| 11 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/11` | 
| 10 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/10` | 
| 9 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/9` | 
| 8 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/8` | 
| 7 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/7` | 
| 6 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/6` | 
| 5 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/5` | 
| 4 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/4` | 
| 3 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/1` | 

For information about version changes, see the [Changelog](#iot-sitewise-connector-changelog)\.

## Requirements<a name="iot-sitewise-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 9, 10, 11, and 12 ]

**Important**  
This version introduces new requirements: AWS IoT Greengrass Core software v1\.10\.2 and [stream manager](stream-manager.md)\.
+ AWS IoT Greengrass Core software v1\.10\.2\.
+ <a name="conn-sitewise-req-stream-manager"></a>[Stream manager](stream-manager.md) enabled on the Greengrass group\.
+ <a name="conn-sitewise-req-java-8"></a>Java 8 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-sitewise-req-regions"></a>This connector can be used only in Amazon Web Services Regions where both [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) and [IoT SiteWise](https://docs.aws.amazon.com/general/latest/gr/iot-sitewise.html) are supported\.
+ <a name="conn-sitewise-req-policy-v3"></a>An IAM policy added to the Greengrass group role\. This role allows the AWS IoT Greengrass group access to the `iotsitewise:BatchPutAssetPropertyValue` action on the target root asset and its children, as shown in the following example\. You can remove the `Condition` from the policy to allow the connector to access all of your IoT SiteWise assets\.

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
                           "/root node asset ID",
                           "/root node asset ID/*"
                       ]
                   }
               }
          }
      ]
  }
  ```

  For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

------
#### [ Versions 6, 7, and 8 ]

**Important**  
This version introduces new requirements: AWS IoT Greengrass Core software v1\.10\.0 and [stream manager](stream-manager.md)\.
+ <a name="conn-sitewise-req-ggc-1010"></a>AWS IoT Greengrass Core software v1\.10\.0\.
+ <a name="conn-sitewise-req-stream-manager"></a>[Stream manager](stream-manager.md) enabled on the Greengrass group\.
+ <a name="conn-sitewise-req-java-8"></a>Java 8 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-sitewise-req-regions"></a>This connector can be used only in Amazon Web Services Regions where both [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) and [IoT SiteWise](https://docs.aws.amazon.com/general/latest/gr/iot-sitewise.html) are supported\.
+ <a name="conn-sitewise-req-policy-v3"></a>An IAM policy added to the Greengrass group role\. This role allows the AWS IoT Greengrass group access to the `iotsitewise:BatchPutAssetPropertyValue` action on the target root asset and its children, as shown in the following example\. You can remove the `Condition` from the policy to allow the connector to access all of your IoT SiteWise assets\.

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
                           "/root node asset ID",
                           "/root node asset ID/*"
                       ]
                   }
               }
          }
      ]
  }
  ```

  For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

------
#### [ Version 5 ]
+ <a name="conn-sitewise-req-ggc-194"></a>AWS IoT Greengrass Core software v1\.9\.4\.
+ <a name="conn-sitewise-req-java-8"></a>Java 8 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-sitewise-req-regions"></a>This connector can be used only in Amazon Web Services Regions where both [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) and [IoT SiteWise](https://docs.aws.amazon.com/general/latest/gr/iot-sitewise.html) are supported\.
+ <a name="conn-sitewise-req-policy-v3"></a>An IAM policy added to the Greengrass group role\. This role allows the AWS IoT Greengrass group access to the `iotsitewise:BatchPutAssetPropertyValue` action on the target root asset and its children, as shown in the following example\. You can remove the `Condition` from the policy to allow the connector to access all of your IoT SiteWise assets\.

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
                           "/root node asset ID",
                           "/root node asset ID/*"
                       ]
                   }
               }
          }
      ]
  }
  ```

  For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

------
#### [ Version 4 ]
+ <a name="conn-sitewise-req-ggc-1010"></a>AWS IoT Greengrass Core software v1\.10\.0\.
+ <a name="conn-sitewise-req-java-8"></a>Java 8 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-sitewise-req-regions"></a>This connector can be used only in Amazon Web Services Regions where both [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) and [IoT SiteWise](https://docs.aws.amazon.com/general/latest/gr/iot-sitewise.html) are supported\.
+ <a name="conn-sitewise-req-policy-v3"></a>An IAM policy added to the Greengrass group role\. This role allows the AWS IoT Greengrass group access to the `iotsitewise:BatchPutAssetPropertyValue` action on the target root asset and its children, as shown in the following example\. You can remove the `Condition` from the policy to allow the connector to access all of your IoT SiteWise assets\.

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
                           "/root node asset ID",
                           "/root node asset ID/*"
                       ]
                   }
               }
          }
      ]
  }
  ```

  For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

------
#### [ Version 3 ]
+ <a name="conn-sitewise-req-ggc-194"></a>AWS IoT Greengrass Core software v1\.9\.4\.
+ <a name="conn-sitewise-req-java-8"></a>Java 8 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-sitewise-req-regions"></a>This connector can be used only in Amazon Web Services Regions where both [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) and [IoT SiteWise](https://docs.aws.amazon.com/general/latest/gr/iot-sitewise.html) are supported\.
+ <a name="conn-sitewise-req-policy-v3"></a>An IAM policy added to the Greengrass group role\. This role allows the AWS IoT Greengrass group access to the `iotsitewise:BatchPutAssetPropertyValue` action on the target root asset and its children, as shown in the following example\. You can remove the `Condition` from the policy to allow the connector to access all of your IoT SiteWise assets\.

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
                           "/root node asset ID",
                           "/root node asset ID/*"
                       ]
                   }
               }
          }
      ]
  }
  ```

  For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

------
#### [ Versions 1 and 2 ]
+ <a name="conn-sitewise-req-ggc-194"></a>AWS IoT Greengrass Core software v1\.9\.4\.
+ <a name="conn-sitewise-req-java-8"></a>Java 8 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-sitewise-req-regions"></a>This connector can be used only in Amazon Web Services Regions where both [AWS IoT Greengrass](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) and [IoT SiteWise](https://docs.aws.amazon.com/general/latest/gr/iot-sitewise.html) are supported\.
+ <a name="conn-sitewise-req-policy-v1"></a>An IAM policy added to the Greengrass group role that allows access to AWS IoT Core and the `iotsitewise:BatchPutAssetPropertyValue` action on the target root asset and its children, as shown in the following example\. You can remove the `Condition` from the policy to allow the connector to access all of your IoT SiteWise assets\.

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
                           "/root node asset ID",
                           "/root node asset ID/*"
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

  For more information, see [Adding and removing IAM identity permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

------

## Parameters<a name="iot-sitewise-connector-param"></a>

------
#### [ Versions 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, and 12 ]<a name="conn-sitewise-params-v2"></a>

`SiteWiseLocalStoragePath`  
The directory on the AWS IoT Greengrass host that the IoT SiteWise connector can write persistent data to\. Defaults to `/var/sitewise`\.  
Display name in the AWS IoT console: **Local storage path**  
Required: `false`  
Type: `string`  
Valid pattern: `^\s*$|\/.`

`AWSSecretsArnList`  
A list of secrets in AWS Secrets Manager that each contain a OPC\-UA user name and password key\-value pair\. Each secret must be a key\-value pair type secret\.  
Display name in the AWS IoT console: **List of ARNs for OPC\-UA username/password secrets**  
Required: `false`  
Type: `JsonArrayOfStrings`  
Valid pattern: `\[( ?,? ?\"(arn:(aws(-[a-z]+)*):secretsmanager:[a-z0-9\\-]+:[0-9]{12}:secret:([a-zA-Z0-9\\\\]+\/)*[a-zA-Z0-9\/_+=,.@\\-]+-[a-zA-Z0-9]+)*\")*\]`

`MaximumBufferSize`  
The maximum size in GB for IoT SiteWise disk usage\. Defaults to 10GB\.  
Display name in the AWS IoT console: **Maximum disk buffer size**  
Required: `false`  
Type: `string`  
Valid pattern: `^\s*$|[0-9]+`

------
#### [ Version 1 ]<a name="conn-sitewise-params-v1"></a>

`SiteWiseLocalStoragePath`  
The directory on the AWS IoT Greengrass host that the IoT SiteWise connector can write persistent data to\. Defaults to `/var/sitewise`\.  
Display name in the AWS IoT console: **Local storage path**  
Required: `false`  
Type: `string`  
Valid pattern: `^\s*$|\/.`

`SiteWiseOpcuaUserIdentityTokenSecretArn`  
The secret in AWS Secrets Manager that contains the OPC\-UA user name and password key\-value pair\. This secret must be a key\-value pair type secret\.  
Display name in the AWS IoT console: **ARN of OPC\-UA username/password secret**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|arn:(aws(-[a-z]+)*):secretsmanager:[a-z0-9\\-]+:[0-9]{12}:secret:([a-zA-Z0-9\\\\]+/)*[a-zA-Z0-9/_+=,.@\\-]+-[a-zA-Z0-9]+`

`SiteWiseOpcuaUserIdentityTokenSecretArn-ResourceId`  
The secret resource in the AWS IoT Greengrass group that references an OPC\-UA user name and password secret\.  
Display name in the AWS IoT console: **OPC\-UA username/password secret resource**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|.+`

`MaximumBufferSize`  
The maximum size in GB for IoT SiteWise disk usage\. Defaults to 10GB\.  
Display name in the AWS IoT console: **Maximum disk buffer size**  
Required: `false`  
Type: `string`  
Valid pattern: `^\s*$|[0-9]+`

------

### Create Connector Example \(AWS CLI\)<a name="iot-sitewise-connector-create"></a>

The following AWS CLI command creates a `ConnectorDefinition` with an initial version that contains the IoT SiteWise connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyIoTSiteWiseConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/IoTSiteWise/versions/11"
        }
    ]
}'
```

**Note**  
The Lambda functions in this connector have a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="iot-sitewise-connector-data-input"></a>

This connector doesn't accept MQTT messages as input data\.

## Output data<a name="iot-sitewise-connector-data-output"></a>

This connector doesn't publish MQTT messages as output data\.

## Limits<a name="iot-sitewise-connector-limits"></a>

This connector is subject to the following all limits imposed by IoT SiteWise, including the following\. For more informatison, see [AWS IoT SiteWise endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/iot-sitewise.html) in the *AWS General Reference*\. 
+ Maximum number of gateways per AWS account\.
+ Maximum number of OPC\-UA sources per gateway\.
+ Maximum rate of timestamp\-quality\-value \(TQV\) data points stored per AWS account\.
+ Maximum rate of TQV data points stored per asset property\.

## Licenses<a name="iot-sitewise-connector-license"></a>

------
#### [ Version 9, 10, 11, and 12 ]

The IoT SiteWise connector includes the following third\-party software/licensing:
+  [MapDB](https://github.com/jankotek/mapdb/blob/master/LICENSE.txt) 
+  [Elsa](https://github.com/jankotek/elsa/blob/master/LICENSE.txt) 
+ [Eclipse Milo](https://github.com/eclipse/milo/blob/maintenance/0.2/LICENSE)

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

------
#### [ Versions 6, 7, and 8 ]

The IoT SiteWise connector includes the following third\-party software/licensing:
+ [Milo](https://github.com/eclipse/milo/) / EDL 1\.0

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

------
#### [ Versions 1, 2, 3, 4, and 5 ]

The IoT SiteWise connector includes the following third\-party software/licensing:
+ [Milo](https://github.com/eclipse/milo/) / EDL 1\.0
+ [Chronicle\-Queue](https://github.com/OpenHFT/Chronicle-Queue) / Apache License 2\.0

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

------

## Changelog<a name="iot-sitewise-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | Date | 
| --- | --- | --- | 
|  12  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/iot-sitewise-connector.html)  |  December 22, 2021  | 
|  11  |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/iot-sitewise-connector.html)  |  March 24, 2021  | 
|  10  |  Configured `StreamManager` to improve handling when the source connection is lost and re\-established\. This version also accepts OPC\-UA values with a `ServerTimestamp` when no `SourceTimestamp` is available\.  |  January 22, 2021  | 
|  9  |  Support launched for custom Greengrass `StreamManager` stream destinations, OPC\-UA deadbanding, custom scan mode and custom scan rate\. Also includes improved performance during configuration updates made from the IoT SiteWise gateway\.  |  December 15, 2020  | 
|  8  |  Improved stability when the connector experiences intermittent network connectivity\.  |  November 19, 2020  | 
|  7  |  Fixed an issue with gateway metrics\.  |  August 14, 2020  | 
|  6  |  Added support for CloudWatch metrics and automatic discovery of new OPC\-UA tags\. This version requires [stream manager](stream-manager.md) and AWS IoT Greengrass Core software v1\.10\.0 or higher\.  |  April 29, 2020  | 
|  5  |  Fixed a compatibility issue with AWS IoT Greengrass Core software v1\.9\.4\.  |  February 12, 2020  | 
|  4  |  Fixed an issue with OPC\-UA server reconnection\.  |  February 7, 2020  | 
|  3  |  Removed `iot:*` permissions requirement\.  |  December 17, 2019  | 
|  2  |  Added support for multiple OPC\-UA secret resources\.  |  December 10, 2019  | 
|  1  |  Initial release\.  |  December 2, 2019  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="iot-sitewise-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)
+ See the following topics in the *AWS IoT SiteWise User Guide*:
  + [What is AWS IoT SiteWise?](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/)
  + [Using a gateway](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/gateway-connector.html)
  + [Gateway CloudWatch metrics](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/monitor-cloudwatch-metrics.html#gateway-metrics)
  + [Troubleshooting an IoT SiteWise gateway](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/troubleshooting.html#troubleshooting-gateway)