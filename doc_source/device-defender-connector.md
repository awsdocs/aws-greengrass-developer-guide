# Device Defender connector<a name="device-defender-connector"></a>

The Device Defender [connector](connectors.md) notifies administrators of changes in the state of a Greengrass core device\. This can help identify unusual behavior that might indicate a compromised device\.

This connector reads system metrics from the `/proc` directory on the core device, and then publishes the metrics to AWS IoT Device Defender\. For metrics reporting details, see [Device metrics document specification](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender-detect.html#DetectMetricsMessagesSpec) in the *AWS IoT Developer Guide*\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 3 | `arn:aws:greengrass:region::/connectors/DeviceDefender/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/DeviceDefender/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/DeviceDefender/versions/1` | 

For information about version changes, see the [Changelog](#device-defender-connector-changelog)\.

## Requirements<a name="device-defender-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 3 ]
+ <a name="conn-req-ggc-v1.9.3"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-device-defender-req-itdd-config"></a>AWS IoT Device Defender configured to use the Detect feature to keep track of violations\. For more information, see [Detect](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender-detect.html) in the *AWS IoT Developer Guide*\.
+ <a name="conn-device-defender-req-proc-dir-resource"></a>A [local volume resource](access-local-resources.md) in the Greengrass group that points to the `/proc` directory\. The resource must use the following properties:
  + Source path: `/proc`
  + Destination path: `/host_proc` \(or a value that matches the [valid pattern](#param-ProcDestinationPath)\)
  + AutoAddGroupOwner: `true`
+ <a name="conn-device-defender-req-psutil-v3"></a>The [psutil](https://pypi.org/project/psutil/) library installed on the Greengrass core\. Version 5\.7\.0 is the latest version that is verified to work with the connector\.
+ <a name="conn-device-defender-req-cbor-v3"></a>The [cbor](https://pypi.org/project/cbor/) library installed on the Greengrass core\. Version 1\.0\.0 is the latest version that is verified to work with the connector\.

------
#### [ Versions 1 \- 2 ]
+ <a name="conn-req-ggc-v1.7.0"></a>AWS IoT Greengrass Core software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-device-defender-req-itdd-config"></a>AWS IoT Device Defender configured to use the Detect feature to keep track of violations\. For more information, see [Detect](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender-detect.html) in the *AWS IoT Developer Guide*\.
+ <a name="conn-device-defender-req-proc-dir-resource"></a>A [local volume resource](access-local-resources.md) in the Greengrass group that points to the `/proc` directory\. The resource must use the following properties:
  + Source path: `/proc`
  + Destination path: `/host_proc` \(or a value that matches the [valid pattern](#param-ProcDestinationPath)\)
  + AutoAddGroupOwner: `true`
+ <a name="conn-device-defender-req-psutil"></a>The [psutil](https://pypi.org/project/psutil/) library installed on the Greengrass core\.
+ <a name="conn-device-defender-req-cbor"></a>The [cbor](https://pypi.org/project/cbor/) library installed on the Greengrass core\.

------

## Connector Parameters<a name="device-defender-connector-param"></a>

This connector provides the following parameters:

`SampleIntervalSeconds`  
The number of seconds between each cycle of gathering and reporting metrics\. The minimum value is 300 seconds \(5 minutes\)\.  
Display name in the AWS IoT console: **Metrics reporting interval**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]*(?:3[0-9][0-9]|[4-9][0-9]{2}|[1-9][0-9]{3,})$`

`ProcDestinationPath-ResourceId`  
The ID of the `/proc` volume resource\.  
This connector is granted read\-only access to the resource\.
Display name in the AWS IoT console: **Resource for /proc directory**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9_-]+`

`ProcDestinationPath`  <a name="param-ProcDestinationPath"></a>
The destination path of the `/proc` volume resource\.  
Display name in the AWS IoT console: **Destination path of /proc resource**  
Required: `true`  
Type: `string`  
Valid pattern: `\/[a-zA-Z0-9_-]+`

### Create Connector Example \(AWS CLI\)<a name="device-defender-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the Device Defender connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyDeviceDefenderConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/DeviceDefender/versions/3",
            "Parameters": {
                "SampleIntervalSeconds": "600",
                "ProcDestinationPath": "/host_proc",
                "ProcDestinationPath-ResourceId": "my-proc-resource"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="device-defender-connector-data-input"></a>

This connector doesn't accept MQTT messages as input data\.

## Output data<a name="device-defender-connector-data-output"></a>

This connector publishes security metrics to AWS IoT Device Defender as output data\.

<a name="topic-filter"></a>**Topic filter in subscription**  
`$aws/things/+/defender/metrics/json`  
This is the topic syntax that AWS IoT Device Defender expects\. The connector replaces the `+` wildcard with the device name \(for example, `$aws/things/thing-name/defender/metrics/json`\)\.

**Example output**  
For metrics reporting details, see [ Device metrics document specification](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender-detect.html#DetectMetricsMessagesSpec) in the *AWS IoT Developer Guide*\.  

```
{
    "header": {
        "report_id": 1529963534,
        "version": "1.0"
    },
    "metrics": {
        "listening_tcp_ports": {
            "ports": [
                {
                    "interface": "eth0",
                    "port": 24800
                },
                {
                    "interface": "eth0",
                    "port": 22
                },
                {
                    "interface": "eth0",
                    "port": 53
                }
            ],
            "total": 3
        },
        "listening_udp_ports": {
            "ports": [
                {
                    "interface": "eth0",
                    "port": 5353
                },
                {
                    "interface": "eth0",
                    "port": 67
                }
            ],
            "total": 2
        },
        "network_stats": {
            "bytes_in": 1157864729406,
            "bytes_out": 1170821865,
            "packets_in": 693092175031,
            "packets_out": 738917180
        },
        "tcp_connections": {
            "established_connections":{
                "connections": [
                    {
                    "local_interface": "eth0",
                    "local_port": 80,
                    "remote_addr": "192.168.0.1:8000"
                    },
                    {
                    "local_interface": "eth0",
                    "local_port": 80,
                    "remote_addr": "192.168.0.1:8000"
                    }
                ],
                "total": 2
            }
        }
    }
}
```

## Licenses<a name="device-defender-connector-license"></a>

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="device-defender-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 3 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 2 | Fix to reduce excessive logging\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="device-defender-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)
+ [Device Defender](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender.html) in the *AWS IoT Developer Guide*