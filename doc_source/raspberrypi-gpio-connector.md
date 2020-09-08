# Raspberry Pi GPIO connector<a name="raspberrypi-gpio-connector"></a>

The Raspberry Pi GPIO [connector](connectors.md) controls general\-purpose input/output \(GPIO\) pins on a Raspberry Pi core device\.

This connector polls input pins at a specified interval and publishes state changes to MQTT topics\. It also accepts read and write requests as MQTT messages from user\-defined Lambda functions\. Write requests are used to set the pin to high or low voltage\.

The connector provides parameters that you use to designate input and output pins\. This behavior is configured before group deployment\. It can't be changed at runtime\.
+ Input pins can be used to receive data from peripheral devices\.
+ Output pins can be used to control peripherals or send data to peripherals\.

You can use this connector for many scenarios, such as:
+ Controlling green, yellow, and red LED lights for a traffic light\.
+ Controlling a fan \(attached to an electrical relay\) based on data from a humidity sensor\.
+ Alerting employees in a retail store when customers press a button\.
+ Using a smart light switch to control other IoT devices\.

**Note**  
This connector is not suitable for applications that have real\-time requirements\. Events with short durations might be missed\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 3 | `arn:aws:greengrass:region::/connectors/RaspberryPiGPIO/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/RaspberryPiGPIO/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/RaspberryPiGPIO/versions/1` | 

For information about version changes, see the [Changelog](#raspberrypi-gpio-connector-changelog)\.

## Requirements<a name="raspberrypi-gpio-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 3 ]
+ <a name="conn-req-ggc-v1.9.3"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-gpio-req-pin-seq"></a>Raspberry Pi 4 Model B, or Raspberry Pi 3 Model B/B\+\. You must know the pin sequence of your Raspberry Pi\. For more information, see [GPIO Pin sequence](#raspberrypi-gpio-connector-req-pins)\.
+ <a name="conn-gpio-req-dev-gpiomem-resource"></a>A [local device resource](access-local-resources.md) in the Greengrass group that points to `/dev/gpiomem` on the Raspberry Pi\. If you create the resource in the console, you must select the **Automatically add OS group permissions of the Linux group that owns the resource** option\. In the API, set the `GroupOwnerSetting.AutoAddGroupOwner` property to `true`\.
+ <a name="conn-gpio-req-rpi-gpio"></a>The [RPi\.GPIO](https://sourceforge.net/p/raspberry-gpio-python/wiki/Home/) module installed on the Raspberry Pi\. In Raspbian, this module is installed by default\. You can use the following command to reinstall it:

  ```
  sudo pip install RPi.GPIO
  ```

------
#### [ Versions 1 \- 2 ]
+ <a name="conn-req-ggc-v1.7.0"></a>AWS IoT Greengrass Core software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="conn-gpio-req-pin-seq"></a>Raspberry Pi 4 Model B, or Raspberry Pi 3 Model B/B\+\. You must know the pin sequence of your Raspberry Pi\. For more information, see [GPIO Pin sequence](#raspberrypi-gpio-connector-req-pins)\.
+ <a name="conn-gpio-req-dev-gpiomem-resource"></a>A [local device resource](access-local-resources.md) in the Greengrass group that points to `/dev/gpiomem` on the Raspberry Pi\. If you create the resource in the console, you must select the **Automatically add OS group permissions of the Linux group that owns the resource** option\. In the API, set the `GroupOwnerSetting.AutoAddGroupOwner` property to `true`\.
+ <a name="conn-gpio-req-rpi-gpio"></a>The [RPi\.GPIO](https://sourceforge.net/p/raspberry-gpio-python/wiki/Home/) module installed on the Raspberry Pi\. In Raspbian, this module is installed by default\. You can use the following command to reinstall it:

  ```
  sudo pip install RPi.GPIO
  ```

------

### GPIO Pin sequence<a name="raspberrypi-gpio-connector-req-pins"></a>

The Raspberry Pi GPIO connector references GPIO pins by the numbering scheme of the underlying System on Chip \(SoC\), not by the physical layout of GPIO pins\. The physical ordering of pins might vary in Raspberry Pi versions\. For more information, see [GPIO](https://www.raspberrypi.org/documentation/usage/gpio/) in the Raspberry Pi documentation\.

The connector can't validate that the input and output pins you configure map correctly to the underlying hardware of your Raspberry Pi\. If the pin configuration is invalid, the connector returns a runtime error when it attempts to start on the device\. To resolve this issue, reconfigure the connector and then redeploy\.

**Note**  
Make sure that peripherals for GPIO pins are properly wired to prevent component damage\.

## Connector Parameters<a name="raspberrypi-gpio-connector-param"></a>

This connector provides the following parameters:

`InputGpios`  
A comma\-separated list of GPIO pin numbers to configure as inputs\. Optionally append `U` to set a pin's pull\-up resistor, or `D` to set the pull\-down resistor\. Example: `"5,6U,7D"`\.  
Display name in the AWS IoT console: **Input GPIO pins**  
Required: `false`\. You must specify input pins, output pins, or both\.  
Type: `string`  
Valid pattern: `^$|^[0-9]+[UD]?(,[0-9]+[UD]?)*$`

`InputPollPeriod`  
The interval \(in milliseconds\) between each polling operation, which checks input GPIO pins for state changes\. The minimum value is 1\.  
This value depends on your scenario and the type of devices that are polled\. For example, a value of `50` should be fast enough to detect a button press\.  
Display name in the AWS IoT console: **Input GPIO polling period**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|^[1-9][0-9]*$`

`OutputGpios`  
A comma\-separated list of GPIO pin numbers to configure as outputs\. Optionally append `H` to set a high state \(1\), or `L` to set a low state \(0\)\. Example: `"8H,9,27L"`\.  
Display name in the AWS IoT console: **Output GPIO pins**  
Required: `false`\. You must specify input pins, output pins, or both\.  
Type: `string`  
Valid pattern: `^$|^[0-9]+[HL]?(,[0-9]+[HL]?)*$`

`GpioMem-ResourceId`  
The ID of the local device resource that represents `/dev/gpiomem`\.  
This connector is granted read\-write access to the resource\.
Display name in the AWS IoT console: **Resource for /dev/gpiomem device**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

### Create Connector Example \(AWS CLI\)<a name="raspberrypi-gpio-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the Raspberry Pi GPIO connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyRaspberryPiGPIOConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/RaspberryPiGPIO/versions/3",
            "Parameters": {
                "GpioMem-ResourceId": "my-gpio-resource",
                "InputGpios": "5,6U,7D",
                "InputPollPeriod": 50,
                "OutputGpios": "8H,9,27L"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in this connector has a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="raspberrypi-gpio-connector-data-input"></a>

This connector accepts read or write requests for GPIO pins on two MQTT topics\.
+ Read requests on the `gpio/+/+/read` topic\.
+ Write requests on the `gpio/+/+/write` topic\.

To publish to these topics, replace the `+` wildcards with the core thing name and the target pin number, respectively\. For example:

```
gpio/core-thing-name/gpio-number/read
```

**Note**  
Currently, when you create a subscription that uses the Raspberry Pi GPIO connector, you must specify a value for at least one of the \+ wildcards in the topic\.

**Topic filter:** `gpio/+/+/read`  
Use this topic to direct the connector to read the state of the GPIO pin that's specified in the topic\.  
The connector publishes the response to the corresponding output topic \(for example, `gpio/core-thing-name/gpio-number/state`\)\.    
**Message properties**  
None\. Messages that are sent to this topic are ignored\.

**Topic filter:** `gpio/+/+/write`  
Use this topic to send write requests to a GPIO pin\. This directs the connector to set the GPIO pin that's specified in the topic to a low or high voltage\.  
+ `0` sets the pin to low voltage\.
+ `1` sets the pin to high voltage\.
The connector publishes the response to the corresponding output `/state` topic \(for example, `gpio/core-thing-name/gpio-number/state`\)\.    
**Message properties**  
The value `0` or `1`, as an integer or string\.  
**Example input**  

```
0
```

## Output data<a name="raspberrypi-gpio-connector-data-output"></a>

This connector publishes data to two topics:
+ High or low state changes on the `gpio/+/+/state` topic\.
+ Errors on the `gpio/+/error` topic\.

**Topic filter:** `gpio/+/+/state`  
Use this topic to listen for state changes on input pins and responses for read requests\. The connector returns the string `"0"` if the pin is in a low state, or `"1"` if it's in a high state\.  
When publishing to this topic, the connector replaces the `+` wildcards with the core thing name and the target pin, respectively\. For example:  

```
gpio/core-thing-name/gpio-number/state
```
Currently, when you create a subscription that uses the Raspberry Pi GPIO connector, you must specify a value for at least one of the \+ wildcards in the topic\.  
**Example output**  

```
0
```

**Topic filter:** `gpio/+/error`  
Use this topic to listen for errors\. The connector publishes to this topic as a result of an invalid request \(for example, when a state change is requested on an input pin\)\.  
When publishing to this topic, the connector replaces the `+` wildcard with the core thing name\.    
**Example output**  

```
{
   "topic": "gpio/my-core-thing/22/write",
   "error": "Invalid GPIO operation",
   "long_description": "GPIO 22 is configured as an INPUT GPIO. Write operations are not permitted."
 }
```

## Usage Example<a name="raspberrypi-gpio-connector-usage"></a>

<a name="connectors-setup-intro"></a>Use the following high\-level steps to set up an example Python 3\.7 Lambda function that you can use to try out the connector\.

**Note**  <a name="connectors-setup-get-started-topics"></a>
The [Get started with connectors \(console\)](connectors-console.md) and [Get started with connectors \(CLI\)](connectors-cli.md) topics contain detailed steps that show you how to configure and deploy an example Twilio Notifications connector\.

1. Make sure you meet the [requirements](#raspberrypi-gpio-connector-req) for the connector\.

1. <a name="connectors-setup-function"></a>Create and publish a Lambda function that sends input data to the connector\.

   Save the [example code](#raspberrypi-gpio-connector-usage-example) as a PY file\. <a name="connectors-setup-function-sdk"></a>Download and unzip the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core)\. Then, create a zip package that contains the PY file and the `greengrasssdk` folder at the root level\. This zip package is the deployment package that you upload to AWS Lambda\.

   <a name="connectors-setup-function-publish"></a>After you create the Python 3\.7 Lambda function, publish a function version and create an alias\.

1. Configure your Greengrass group\.

   1. <a name="connectors-setup-gg-function"></a>Add the Lambda function by its alias \(recommended\)\. Configure the Lambda lifecycle as long\-lived \(or `"Pinned": true` in the CLI\)\.

   1. <a name="connectors-setup-device-resource"></a>Add the required local device resource and grant read/write access to the Lambda function\.

   1. Add the connector and configure its [parameters](#raspberrypi-gpio-connector-param)\.

   1. Add subscriptions that allow the connector to receive [input data](#raspberrypi-gpio-connector-data-input) and send [output data](#raspberrypi-gpio-connector-data-output) on supported topic filters\.
      + <a name="connectors-setup-subscription-input-data"></a>Set the Lambda function as the source, the connector as the target, and use a supported input topic filter\.
      + <a name="connectors-setup-subscription-output-data"></a>Set the connector as the source, AWS IoT Core as the target, and use a supported output topic filter\. You use this subscription to view status messages in the AWS IoT console\.

1. <a name="connectors-setup-deploy-group"></a>Deploy the group\.

1. <a name="connectors-setup-test-sub"></a>In the AWS IoT console, on the **Test** page, subscribe to the output data topic to view status messages from the connector\. The example Lambda function is long\-lived and starts sending messages immediately after the group is deployed\.

   When you're finished testing, you can set the Lambda lifecycle to on\-demand \(or `"Pinned": false` in the CLI\) and deploy the group\. This stops the function from sending messages\.

### Example<a name="raspberrypi-gpio-connector-usage-example"></a>

The following example Lambda function sends an input message to the connector\. This example sends read requests for a set of input GPIO pins\. It shows how to construct topics using the core thing name and pin number\.

```
import greengrasssdk
import json
import os

iot_client = greengrasssdk.client('iot-data')
INPUT_GPIOS = [6, 17, 22]

thingName = os.environ['AWS_IOT_THING_NAME']

def get_read_topic(gpio_num):
    return '/'.join(['gpio', thingName, str(gpio_num), 'read'])

def get_write_topic(gpio_num):
    return '/'.join(['gpio', thingName, str(gpio_num), 'write'])

def send_message_to_connector(topic, message=''):
    iot_client.publish(topic=topic, payload=str(message))

def set_gpio_state(gpio, state):
    send_message_to_connector(get_write_topic(gpio), str(state))

def read_gpio_state(gpio):
    send_message_to_connector(get_read_topic(gpio))

def publish_basic_message():
    for i in INPUT_GPIOS:
    	read_gpio_state(i)

publish_basic_message()

def lambda_handler(event, context):
    return
```

## Licenses<a name="raspberrypi-gpio-connector-license"></a>

The Raspberry Pi GPIO; connector includes the following third\-party software/licensing:
+ [RPi\.GPIO](https://pypi.org/project/RPi.GPIO/)/MIT

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="raspberrypi-gpio-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 3 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 2 | Updated connector ARN for AWS Region support\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="raspberrypi-gpio-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)
+ [GPIO](https://www.raspberrypi.org/documentation/usage/gpio/) in the Raspberry Pi documentation