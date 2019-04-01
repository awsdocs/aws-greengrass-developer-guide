# Device Defender<a name="device-defender-connector"></a>

The Device Defender [connector](connectors.md) notifies administrators of changes in the state of a Greengrass core device\. This can help identify unusual behavior that might indicate a compromised device\.

This connector reads system metrics from the `/proc` directory on the core device, and then publishes the metrics to AWS IoT Device Defender\. For metrics reporting details, see [Device Metrics Document Specification](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender-detect.html#DetectMetricsMessagesSpec) in the *AWS IoT Developer Guide*\.

**ARN**: `arn:aws:greengrass:region::/connectors/DeviceDefender/versions/1`

## Requirements<a name="device-defender-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ AWS IoT Device Defender configured to use the Detect feature to keep track of violations\. For more information, see [Detect](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender-detect.html) in the *AWS IoT Developer Guide*\.
+ A [local volume resource](access-local-resources.md) in the Greengrass group that points to the `/proc` directory\. The resource must use the following properties:
  + Source path: `/proc`
  + Destination path: `/host_proc` \(or a value that matches the [valid pattern](#param-ProcDestinationPath)\)
  + AutoAddGroupOwner: `true`
+ The [psutil](https://pypi.org/project/psutil/) library installed on the AWS IoT Greengrass core\. Use the following command to install it:

  ```
  pip install psutil
  ```
+ The [cbor](http://cbor.io/) library installed on the AWS IoT Greengrass core\. Use the following command to install it:

  ```
  pip install cbor
  ```

## Connector Parameters<a name="device-defender-connector-param"></a>

This connector provides the following parameters:

`SampleIntervalSeconds`  
The number of seconds between each cycle of gathering and reporting metrics\. The minimum value is 300 seconds \(5 minutes\)\.  
Display name in console: **Metrics reporting interval**  
Required: `true`  
Type: `string`  
Valid pattern: `^[0-9]*(?:3[0-9][0-9]|[4-9][0-9]{2}|[1-9][0-9]{3,})$`

`ProcDestinationPath-ResourceId`  
The ID of the `/proc` volume resource\.  
This connector is granted read\-only access to the resource\.
Display name in console: **Resource for /proc directory**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9_-]+`

`ProcDestinationPath`  <a name="param-ProcDestinationPath"></a>
The destination path of the `/proc` volume resource\.  
Display name in console: **Destination path of /proc resource**  
Required: `true`  
Type: `string`  
Valid pattern: `\/[a-zA-Z0-9_-]+`

### Create Connector Example \(CLI\)<a name="device-defender-connector-create"></a>

The following CLI command creates an `ConnectorDefinition` with an initial version that contains the Device Defender connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyDeviceDefenderConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/DeviceDefender/versions/1",
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

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="device-defender-connector-data-input"></a>

This connector doesn't accept MQTT messages as input data\.

## Output Data<a name="device-defender-connector-data-output"></a>

This connector publishes security metrics to AWS IoT Device Defender as output data\.

**Topic filter**  
`$aws/things/+/defender/metrics/json`  
This is the topic syntax that AWS IoT Device Defender expects\. The connector replaces the `+` wildcard with the device name \(for example, `$aws/things/thing-name/defender/metrics/json`\)\.

**Example output**  
For metrics reporting details, see [ Device Metrics Document Specification](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender-detect.html#DetectMetricsMessagesSpec) in the *AWS IoT Developer Guide*\.  

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

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## See Also<a name="device-defender-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)
+ [Device Defender](https://docs.aws.amazon.com/iot/latest/developerguide/device-defender.html) in the *AWS IoT Developer Guide*