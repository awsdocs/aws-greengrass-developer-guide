# Raspberry Pi GPIO<a name="raspberrypi-gpio-connector"></a>

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

**ARN**: `arn:aws:greengrass:region::/connectors/RaspberryPiGPIO/versions/1`

## Requirements<a name="raspberrypi-gpio-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ Raspberry Pi 3 [Model B\+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) or [Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)\. You must know the pin sequence of your Raspberry Pi\. For more information, see [GPIO Pin Sequence](#raspberrypi-gpio-connector-req-pins)\.
+ A [local device resource](access-local-resources.md) in the Greengrass group that points to `/dev/gpiomem` on the Raspberry Pi\. If you create the resource in the console, you must select the **Automatically add OS group permissions of the Linux group that owns the resource** option\. In the API, set the `GroupOwnerSetting.AutoAddGroupOwner` property to `true`\.
+ The [RPi\.GPIO](https://sourceforge.net/p/raspberry-gpio-python/wiki/Home/) module installed on the Raspberry Pi\. In Raspbian, this module is installed by default\. You can use the following command to reinstall it:

  ```
  sudo pip install RPi.GPIO
  ```

### GPIO Pin Sequence<a name="raspberrypi-gpio-connector-req-pins"></a>

The Raspberry Pi GPIO connector references GPIO pins by the numbering scheme of the underlying System on Chip \(SoC\), not by the physical layout of GPIO pins\. The physical ordering of pins might vary in Raspberry Pi versions\. For more information, see [GPIO](https://www.raspberrypi.org/documentation/usage/gpio/) in the Raspberry Pi documentation\.

The connector can't validate that the input and output pins you configure map correctly to the underlying hardware of your Raspberry Pi\. If the pin configuration is invalid, the connector returns a runtime error when it attempts to start on the device\. To resolve this issue, reconfigure the connector and then redeploy\.

**Note**  
Make sure that peripherals for GPIO pins are properly wired to prevent component damage\.

## Connector Parameters<a name="raspberrypi-gpio-connector-param"></a>

This connector provides the following parameters:

`InputGpios`  
A comma\-separated list of GPIO pin numbers to configure as inputs\. Optionally append `U` to set a pin's pull\-up resistor, or `D` to set the pull\-down resistor\. Example: `"5,6U,7D"`\.  
Display name in console: **Input GPIO pins**  
Required: `false`\. You must specify input pins, output pins, or both\.  
Type: `string`  
Valid pattern: `^$|^[0-9]+[UD]?(,[0-9]+[UD]?)*$`

`InputPollPeriod`  
The interval \(in milliseconds\) between each polling operation, which checks input GPIO pins for state changes\. The minimum value is 1\.  
This value depends on your scenario and the type of devices that are polled\. For example, a value of `50` should be fast enough to detect a button press\.  
Display name in console: **Input GPIO polling period**  
Required: `false`  
Type: `integer`  
Valid pattern: `^$|^[1-9][0-9]*$`

`OutputGpios`  
A comma\-separated list of GPIO pin numbers to configure as outputs\. Optionally append `H` to set a high state \(1\), or `L` to set a low state \(0\)\. Example: `"8H,9,27L"`\.  
Display name in console: **Output GPIO pins**  
Required: `false`\. You must specify input pins, output pins, or both\.  
Type: `string`  
Valid pattern: `^$|^[0-9]+[HL]?(,[0-9]+[HL]?)*$`

`GpioMem-ResourceId`  
The ID of the local device resource that represents `/dev/gpiomem`\.  
This connector is granted read\-write access to the resource\.
Display name in console: **Resource for /dev/gpiomem device**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

### Create Connector Example \(CLI\)<a name="raspberrypi-gpio-connector-create"></a>

The following CLI command creates an `ConnectorDefinition` with an initial version that contains the Raspberry Pi GPIO connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyRaspberryPiGPIOConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/RaspberryPiGPIO/versions/1",
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

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="raspberrypi-gpio-connector-data-input"></a>

This connector accepts read or write requests for GPIO pins on two MQTT topics\. Input messages must be in JSON format\.
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

## Output Data<a name="raspberrypi-gpio-connector-data-output"></a>

This connector publishes data to two topics:
+ High or low state changes on the `gpio/+/+/state` topic\.
+ Errors on the `gpio/+/error` topic\.

**Topic filter: `gpio/+/+/state`**  
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

**Topic filter:** `gpio/+/errors`  
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

The following example Lambda function sends an input message to the connector\. This example sends read requests for a set of input GPIO pins\. It shows how to construct topics using the core thing name and pin number\.

**Note**  
This Python function uses the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) to publish an MQTT message\. You can use the following [pip](https://pypi.org/project/pip/) command to install the Python version of the SDK on your core device:   

```
pip install greengrasssdk
```

```
import greengrasssdk
import json

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

def function_handler(event, context):
    return
```

## Licenses<a name="raspberrypi-gpio-connector-license"></a>

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## See Also<a name="raspberrypi-gpio-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)
+ [GPIO](https://www.raspberrypi.org/documentation/usage/gpio/) in the Raspberry Pi documentation