--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# AWS\-provided Greengrass connectors<a name="connectors-list"></a>

AWS provides the following connectors that support common AWS IoT Greengrass scenarios\. For more information about how connectors work, see the following documentation:
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Get started with connectors \(console\)](connectors-console.md) or [Get started with connectors \(CLI\)](connectors-cli.md)


| Connector | Description | Supported Lambda runtimes | Supports **No container** mode | 
| --- | --- | --- | --- | 
| [CloudWatch Metrics](cloudwatch-metrics-connector.md) | Publishes custom metrics to Amazon CloudWatch\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [Device Defender](device-defender-connector.md) | Sends system metrics to AWS IoT Device Defender\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | No | 
| [Docker Application Deployment](docker-app-connector.md) | Runs a Docker Compose file to start a Docker application on the core device\. | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [IoT Analytics](iot-analytics-connector.md) | Sends data from devices and sensors to AWS IoT Analytics\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [IoT Ethernet IP Protocol Adapter](ethernet-ip-connector.md) | Collects data from Ethernet/IP devices\. | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [IoT SiteWise](iot-sitewise-connector.md) | Sends data from devices and sensors to asset properties in AWS IoT SiteWise\. | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [Kinesis Firehose](kinesis-firehose-connector.md) | Sends data to Amazon Kinesis Data Firehose delivery streams\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [ML Feedback](ml-feedback-connector.md) | Publishes machine learning model input to the cloud and output to an MQTT topic\. | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | No | 
| [ML Image Classification](image-classification-connector.md) | Runs a local image classification inference service\. This connector provides versions for several platforms\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | No | 
| [ML Object Detection](obj-detection-connector.md) | Runs a local object detection inference service\. This connector provides versions for several platforms\. | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | No | 
| [Modbus\-RTU Protocol Adapter](modbus-protocol-adapter-connector.md) | Sends requests to Modbus RTU devices\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | No | 
| [Modbus\-TCP Protocol Adapter](modbus-tcp-connector.md) | Collects data from ModbusTCP devices\. | [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [Raspberry Pi GPIO](raspberrypi-gpio-connector.md) | Controls GPIO pins on a Raspberry Pi core device\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | No | 
| [Serial Stream](serial-stream-connector.md) | Reads and writes to a serial port on the core device\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | No | 
| [ServiceNow MetricBase Integration](servicenow-connector.md) | Publishes time series metrics to ServiceNow MetricBase\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [SNS](sns-connector.md) | Sends messages to an Amazon SNS topic\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [Splunk Integration](splunk-connector.md) | Publishes data to Splunk HEC\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 
| [Twilio Notifications](twilio-notifications-connector.md) | Initiates a Twilio text or voice message\. | <a name="python-connectors-runtime"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/connectors-list.html) | Yes | 

\* To use the Python 3\.8 runtimes, you must create a symbolic link from the default Python 3\.7 installation folder to the installed Python 3\.8 binaries\. For more information, see the connector\-specific requirements\.

**Note**  
We recommend that you [upgrade connector versions](connectors.md#upgrade-connector-versions) from Python 2\.7 to Python 3\.7\. Continued support for Python 2\.7 connectors depends on AWS Lambda runtime support\. For more information, see [Runtime support policy](https://docs.aws.amazon.com/lambda/latest/dg/runtime-support-policy.html) in the *AWS Lambda Developer Guide*\.