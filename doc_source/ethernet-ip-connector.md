--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

--------

# IoT Ethernet IP Protocol Adapter connector<a name="ethernet-ip-connector"></a>

The IoT Ethernet IP Protocol Adapter [connector](connectors.md) collects data from local devices using the Ethernet/IP protocol\. You can use this connector to collect data from multiple devices and publish it to a `StreamManager` message stream\. 

You can also use this connector with the IoT SiteWise connector and your IoT SiteWise gateway\. Your gateway must supply the configuration for the connector\. For more information, see [Configure an Ethernet/IP \(EIP\) source](http://docs.aws.amazon.com/iot-sitewise/latest/userguide/configure-eip-source.html) in the IoT SiteWise user guide\. 

**Note**  
This connector runs in [No container](lambda-group-config.md#no-container-mode) isolation mode, so you can deploy it to a AWS IoT Greengrass group running in a Docker container\. 

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 2 \(recommended\) | `arn:aws:greengrass:region::/connectors/IoTEIPProtocolAdaptor/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/IoTEIPProtocolAdaptor/versions/1` | 

For information about version changes, see the [Changelog](#ethernet-ip-connector-changelog)\.

## Requirements<a name="ethernet-ip-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 1 and 2 ]
+ AWS IoT Greengrass Core software v1\.10\.2 or later\.
+ Stream manager enabled on the AWS IoT Greengrass group\.
+ Java 8 installed on the core device and added to the `PATH` environment variable\.
+ A minimum of 256 MB additional RAM\. This requirement is in addition to AWS IoT Greengrass Core memory requirements\.

**Note**  
 This connector is available only in the following Regions:   
cn\-north\-1
ap\-southeast\-1
ap\-southeast\-2
eu\-central\-1
eu\-west\-1
us\-east\-1
us\-west\-2

------

## Connector Parameters<a name="ethernet-ip-connector-param"></a>

This connector supports the following parameters:

`LocalStoragePath`  
The directory on the AWS IoT Greengrass host that the IoT SiteWise connector can write persistent data to\. The default directory is `/var/sitewise`\.  
Display name in the AWS IoT console: **Local storage path**  
Required: `false`  
Type: `string`  
Valid pattern: `^\s*$|\/.`

`ProtocolAdapterConfiguration`  
The set of Ethernet/IP collector configurations that the connector collect data from or connect to\. This can be an empty list\.  
Display name in the AWS IoT console: **Protocol Adapter Configuration**  
Required: `true`  
Type: A well\-formed JSON string that defines the set of supported feedback configurations\.

 The following is an example of a `ProtocolAdapterConfiguration`: 

```
{
    "sources": [
        {
            "type": "EIPSource",
            "name": "TestSource",
            "endpoint": {
                "ipAddress": "52.89.2.42",
                "port": 44818
            },
            "destination": {
                "type": "StreamManager",
                "streamName": "MyOutput_Stream",
                "streamBufferSize": 10
            },
            "destinationPathPrefix": "EIPSource_Prefix",
            "propertyGroups": [
                {
                    "name": "DriveTemperatures",
                    "scanMode": {
                        "type": "POLL",
                        "rate": 10000
                    },
                    "tagPathDefinitions": [
                        {
                            "type": "EIPTagPath",
                            "path": "arrayREAL[0]",
                            "dstDataType": "double"
                        }
                    ]
                }
            ]
        }
    ]
}
```

### Create Connector Example \(AWS CLI\)<a name="eip-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the IoT Ethernet IP Protocol Adapter connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version 
'{
    "Connectors": [
        {
            "Id": "MyIoTEIPProtocolConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/IoTEIPProtocolAdaptor/versions/2",
            "Parameters": {
                "ProtocolAdaptorConfiguration": "{ \"sources\": [{ \"type\": \"EIPSource\", \"name\": \"Source1\", \"endpoint\": { \"ipAddress\": \"54.245.77.218\", \"port\": 44818 }, \"destinationPathPrefix\": \"EIPConnector_Prefix\", \"propertyGroups\": [{ \"name\": \"Values\", \"scanMode\": { \"type\": \"POLL\", \"rate\": 2000 }, \"tagPathDefinitions\": [{ \"type\": \"EIPTagPath\", \"path\": \"arrayREAL[0]\", \"dstDataType\": \"double\" }]}]}]}",
                "LocalStoragePath": "/var/MyIoTEIPProtocolConnectorState"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

## Input data<a name="ethernet-ip-connector-data-input"></a>

This connector doesn't accept MQTT messages as input data\.

## Output data<a name="ethernet-ip-connector-data-output"></a>

This connector publishes data to `StreamManager`\. You must configure the destination message stream\. The output messages are of the following structure:

```
{
    "alias": "string",
    "messages": [
        {
            "name": "string",
            "value": boolean|double|integer|string,
            "timestamp": number,
            "quality": "string"
        }
    ]
}
```

## Licenses<a name="ethernet-ip-connector-license"></a>

The IoT Ethernet IP Protocol Adapter connector includes the following third\-party software/licensing:
+ [Ethernet/IP client](https://github.com/digitalpetri/ethernet-ip/blob/master/LICENSE)
+ [MapDB](https://github.com/jankotek/mapdb/blob/master/LICENSE.txt)
+ [Elsa](https://github.com/jankotek/elsa/blob/master/LICENSE.txt)

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="ethernet-ip-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | Date | 
| --- | --- | --- | 
| 2 | This version contains bug fixes\. | December 23, 2021 | 
| 1 | Initial release\. | December 15, 2020 | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="ethernet-ip-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)