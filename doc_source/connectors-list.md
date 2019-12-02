# AWS\-Provided Greengrass Connectors<a name="connectors-list"></a>

AWS provides the following connectors that support common AWS IoT Greengrass scenarios\. For more information about how connectors work, see the following documentation:
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Get Started with Connectors \(Console\)](connectors-console.md) or [Get Started with Connectors \(CLI\)](connectors-cli.md)


| Connector | Description | Lambda runtime | 
| --- | --- | --- | 
| [CloudWatch Metrics](cloudwatch-metrics-connector.md) | Publishes custom metrics to Amazon CloudWatch\. | Python 2\.7 | 
| [Device Defender](device-defender-connector.md) | Sends system metrics to AWS IoT Device Defender\. | Python 2\.7 | 
| [Docker Application Deployment](docker-app-connector.md) | Runs a Docker Compose file to start a Docker application on the core device\. | Python 3\.7 | 
| [IoT Analytics](iot-analytics-connector.md) | Sends data from devices and sensors to AWS IoT Analytics\. | Python 2\.7 | 
| [IoT SiteWise](iot-sitewise-connector.md) | Sends data from devices and sensors to asset properties in AWS IoT SiteWise\. | Java 8 | 
| [Kinesis Firehose](kinesis-firehose-connector.md) | Sends data to Amazon Kinesis Data Firehose delivery streams\. | Python 2\.7 | 
| [ML Feedback](ml-feedback-connector.md) | Publishes machine learning model input to the cloud and output to an MQTT topic\. | Python 3\.7 | 
| [ML Image Classification](image-classification-connector.md) | Runs a local image classification inference service\. This connector provides versions for several platforms\. | Python 3\.7 | 
| [ML Object Detection](obj-detection-connector.md) | Runs a local object detection inference service\. This connector provides versions for several platforms\. | Python 3\.7 | 
| [Modbus\-RTU Protocol Adapter](modbus-protocol-adapter-connector.md) | Sends requests to Modbus RTU devices\. | Python 2\.7 | 
| [Raspberry Pi GPIO](raspberrypi-gpio-connector.md) | Controls GPIO pins on a Raspberry Pi core device\. | Python 2\.7 | 
| [Serial Stream](serial-stream-connector.md) | Reads and writes to a serial port on the core device\. | Python 2\.7 | 
| [ServiceNow MetricBase Integration](servicenow-connector.md) | Publishes time series metrics to ServiceNow MetricBase\. | Python 2\.7 | 
| [SNS](sns-connector.md) | Sends messages to an Amazon SNS topic\. | Python 2\.7 | 
| [Splunk Integration](splunk-connector.md) | Publishes data to Splunk HEC\. | Python 2\.7 | 
| [Twilio Notifications](twilio-notifications-connector.md) | Triggers a Twilio text or voice message\. | Python 2\.7 | 