--------

You are viewing the documentation for AWS IoT Greengrass Version 1\. AWS IoT Greengrass Version 2 is the latest major version of AWS IoT Greengrass\. For more information about using AWS IoT Greengrass V2, see the [https://docs.aws.amazon.com/greengrass/v2/developerguide](https://docs.aws.amazon.com/greengrass/v2/developerguide)\.

--------

# Modbus\-TCP Protocol Adapter connector<a name="modbus-tcp-connector"></a>

The Modbus\-TCP Protocol Adapter [connector](connectors.md) collects data from local devices through the ModbusTCP protocol and publishes it to the selected `StreamManager` streams\.

You can also use this connector with the IoT SiteWise connector and your IoT SiteWise gateway\. Your gateway must supply the configuration for the connector\. For more information, see [Configure a Modbus TCP source](http://docs.aws.amazon.com/iot-sitewise/latest/userguide/configure-modbus-source.html) in the IoT SiteWise user guide\. 

**Note**  
 This connector runs in [No container](lambda-group-config.md#no-container-mode) isolation mode, so you can deploy it to a AWS IoT Greengrass group running in a Docker container\. 

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 1 | `arn:aws:greengrass:region::/connectors/ModbusTCPConnector/versions/1` | 
| 2 | `arn:aws:greengrass:region::/connectors/ModbusTCPConnector/versions/2` | 

For information about version changes, see the [Changelog](#modbus-tcp-connector-changelog)\.

## Requirements<a name="modbus-tcp-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 1 \- 2 ]
+ AWS IoT Greengrass Core software v1\.10\.2 or later\.
+ Stream manager enabled on the AWS IoT Greengrass group\.
+ Java 8 installed on the core device and added to the `PATH` environment variable\.

**Note**  
 This connector is only available in the following regions:   
ap\-southeast\-1
ap\-southeast\-2
eu\-central\-1
eu\-west\-1
us\-east\-1
us\-west\-2
cn\-north\-1

------

## Connector Parameters<a name="modbus-tcp-connector-param"></a>

This connector supports the following parameters:

`LocalStoragePath`  
The directory on the AWS IoT Greengrass host that the IoT SiteWise connector can write persistent data to\. The default directory is `/var/sitewise`\.  
Display name in the AWS IoT console: **Local storage path**  
Required: `false`  
Type: `string`  
Valid pattern: `^\s*$|\/.`

`MaximumBufferSize`  
The maximum size in GB for IoT SiteWise disk usage\. The default size is 10GB\.  
Display name in the AWS IoT console: **Maximum disk buffer size**  
Required: `false`  
Type: `string`  
Valid pattern: `^\s*$|[0-9]+`

`CapabilityConfiguration`  
The set of Modbus TCP collector configurations that the connector collects data from and connects to\.  
Display name in the AWS IoT console: **CapabilityConfiguration**  
Required: `false`  
Type: A well\-formed JSON string that defines the set of supported feedback configurations\.

The following is an example of a `CapabilityConfiguration`:

```
{
    "sources": [
        {
            "type": "ModBusTCPSource",
            "name": "SourceName1",
            "measurementDataStreamPrefix": "SourceName1_Prefix",
            "destination": {
                "type": "StreamManager",
                "streamName": "SiteWise_Stream_1",
                "streamBufferSize": 8
            },
            "endpoint": {
                "ipAddress": "127.0.0.1",
                "port": 8081,
                "unitId": 1
            },
            "propertyGroups": [
                {
                    "name": "GroupName",
                    "tagPathDefinitions": [
                        {
                            "type": "ModBusTCPAddress",
                            "tag": "TT-001",
                            "address": "30001",
                            "size": 2,
                            "srcDataType": "float",
                            "transformation": "byteWordSwap",
                            "dstDataType": "double"
                        }
                    ],
                    "scanMode": {
                        "type": "POLL",
                        "rate": 100
                    }
                }
            ]
        }
    ]
}
```

### Create Connector Example \(AWS CLI\)<a name="modbus-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the Modbus\-TCP Protocol Adapter connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '
{
    "Connectors": [
        {
            "Id": "MyModbusTCPConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ModbusTCP/versions/2",
            "Parameters": {
                "capability_configuration": "{\"version\":1,\"namespace\":\"iotsitewise:modbuscollector:1\",\"configuration\":\"{\"sources\":[{\"type\":\"ModBusTCPSource\",\"name\":\"SourceName1\",\"measurementDataStreamPrefix\":\"\",\"endpoint\":{\"ipAddress\":\"127.0.0.1\",\"port\":8081,\"unitId\":1},\"propertyGroups\":[{\"name\":\"PropertyGroupName\",\"tagPathDefinitions\":[{\"type\":\"ModBusTCPAddress\",\"tag\":\"TT-001\",\"address\":\"30001\",\"size\":2,\"srcDataType\":\"hexdump\",\"transformation\":\"noSwap\",\"dstDataType\":\"string\"}],\"scanMode\":{\"rate\":200,\"type\":\"POLL\"}}],\"destination\":{\"type\":\"StreamManager\",\"streamName\":\"SiteWise_Stream\",\"streamBufferSize\":10},\"minimumInterRequestDuration\":200}]}\"}"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

## Input data<a name="modbus-tcp-connector-data-input"></a>

This connector doesn't accept MQTT messages as input data\.

## Output data<a name="modbus-tcp-connector-data-output"></a>

This connector publishes data to `StreamManager`\. You must configure the destination message stream\. The output messages are of the following structure:

```
{
 "alias" : "string",
 "messages" : [
 {
 "name": "string",
 "value": boolean|double|integer|string,
 "timestamp": number,
 "quality": "string"
 }
 ]
}
```

## Licenses<a name="modbus-tcp-connector-license"></a>

The Modbus\-TCP Protocol Adapter connector includes the following third\-party software/licensing:
+ [Digital Petri](https://github.com/digitalpetri/modbus) Modbus

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="modbus-tcp-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | Date | 
| --- | --- | --- | 
| 2 | Added support for ASCII, UTF8, and ISO8859 encoded source strings\. | May 24, 2021 | 
| 1 | Initial release\. | December 15, 2020 | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="modbus-tcp-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)