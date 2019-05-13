# Twilio Notifications<a name="twilio-notifications-connector"></a>

The Twilio Notifications [connector](connectors.md) makes automated phone calls or sends text messages through Twilio\. You can use this connector to send notifications in response to events in the Greengrass group\. For phone calls, the connector can forward a voice message to the recipient\.

This connector receives Twilio message information on an MQTT topic, and then triggers a Twilio notification\.

**Note**  
For a tutorial that shows how to use the Twilio Notifications connector, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md) or [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 2 | arn:aws:greengrass:*region*::/connectors/TwilioNotifications/versions/2 | 
| 1 | arn:aws:greengrass:*region*::/connectors/TwilioNotifications/versions/1 | 

For information about version changes, see the [Changelog](#twilio-notifications-connector-changelog)\.

## Requirements<a name="twilio-notifications-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\. AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ A Twilio account SID, auth token, and Twilio\-enabled phone number\. After you create a Twilio project, these values are available on the project dashboard\.
**Note**  
You can use a Twilio trial account\. If you're using a trial account, you must add non\-Twilio recipient phone numbers to a list of verified phone numbers\. For more information, see [ How to Work with your Free Twilio Trial Account](https://www.twilio.com/docs/usage/tutorials/how-to-use-your-free-trial-account)\.
+ A text type secret in AWS Secrets Manager that stores the Twilio auth token\. For more information, see [Creating a Basic Secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.
**Note**  
To create the secret in the Secrets Manager console, enter your token on the **Plaintext** tab\. Don't include quotation marks or other formatting\. In the API, specify the token as the value for the `SecretString` property\.
+ A secret resource in the Greengrass group that references the Secrets Manager secret\. For more information, see [Deploy Secrets to the AWS IoT Greengrass Core](secrets.md)\.

## Connector Parameters<a name="twilio-notifications-connector-param"></a>

This connector provides the following parameters\.

`TWILIO_ACCOUNT_SID`  
The Twilio account SID that's used to invoke the Twilio API\.  
Display name in console: **Twilio account SID**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`TwilioAuthTokenSecretArn`  
The ARN of the Secrets Manager secret that stores the Twilio auth token\.  
This is used to access the value of the local secret on the core\.
Display name in console: **ARN of Twilio auth token secret**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:secretsmanager:[a-z0-9\-]+:[0-9]{12}:secret:([a-zA-Z0-9\\]+/)*[a-zA-Z0-9/_+=,.@\-]+-[a-zA-Z0-9]+`

`TwilioAuthTokenSecretArn-ResourceId`  
The ID of the secret resource in the Greengrass group that references the secret for the Twilio auth token\.  
Display name in console: **Twilio auth token resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`DefaultFromPhoneNumber`  
The default Twilio\-enabled phone number that Twilio uses to send messages\. Twilio uses this number to initiate the text or call\.  
+ If you don't configure a default phone number, you must specify a phone number in the `from_number` property in the input message body\.
+ If you do configure a default phone number, you can optionally override the default by specifying the `from_number` property in the input message body\.
Display name in console: **Default from phone number**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|\+[0-9]+`

### Create Connector Example \(CLI\)<a name="twilio-notifications-connector-create"></a>

The following example CLI command creates a `ConnectorDefinition` with an initial version that contains the Twilio Notifications connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyTwilioNotificationsConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/TwilioNotifications/versions/2",
            "Parameters": {
                "TWILIO_ACCOUNT_SID": "abcd12345xyz",
                "TwilioAuthTokenSecretArn": "arn:aws:secretsmanager:region:account-id:secret:greengrass-secret-hash",
                "TwilioAuthTokenSecretArn-ResourceId": "MyTwilioSecret",
                "DefaultFromPhoneNumber": "+19999999999"
            }
        }
    ]
}'
```

For tutorials that show how add the Twilio Notifications connector to a group, see [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md) and [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="twilio-notifications-connector-data-input"></a>

This connector accepts Twilio message information on two MQTT topics\. Input messages must be in JSON format\.
+ Text message information on the `twilio/txt` topic\.
+ Phone message information on the `twilio/call` topic\.

**Note**  
The input message payload can include a text message \(`message`\) or voice message \(`voice_message_location`\), but not both\.

**Topic filter: `twilio/txt`**    
**Message properties**    
`request`  
Information about the Twilio notification\.  
Required: `true`  
Type: `object` that includes the following properties:    
`recipient`  
The message recipient\. Only one recipient is supported\.  
Required: `true`  
Type: `object` that include the following properties:    
`name`  
The name of the recipient\.  
Required: `true`  
Type: `string`  
Valid pattern: `.*`  
`phone_number`  
The phone number of the recipient\.  
Required: `true`  
Type: `string`  
Valid pattern: `\+[1-9]+`  
`message`  
The text content of the text message\. Only text messages are supported on this topic\. For voice messages, use `twilio/call`\.  
Required: `true`  
Type: `string`  
Valid pattern: `.+`  
`from_number`  
The phone number of the sender\. Twilio uses this phone number to initiate the message\. This property is required if the `DefaultFromPhoneNumber` parameter isn't configured\. If `DefaultFromPhoneNumber` is configured, you can use this property to override the default\.  
Required: `false`  
Type: `string`  
Valid pattern: `\+[1-9]+`  
`retries`  
The number of retries\. The default is 0\.  
Required: `false`  
Type: `integer`  
`id`  
An arbitrary ID for the request\. This property is used to map an input request to an output response\.   
Required: `true`  
Type: `string`  
Valid pattern: `.+`  
**Example input**  

```
{
    "request": {
        "recipient": {
            "name": "Darla",
            "phone_number": "+12345000000",
            "message": "Hello from the edge"
        },
        "from_number": "+19999999999",
        "retries": 3
    },
    "id": "request123"
}
```

**Topic filter: `twilio/call`**    
**Message properties**    
`request`  
Information about the Twilio notification\.  
Required: `true`  
Type: `object` that includes the following properties:    
`recipient`  
The message recipient\. Only one recipient is supported\.  
Required: `true`  
Type: `object` that include the following properties:    
`name`  
The name of the recipient\.  
Required: `true`  
Type: `string`  
Valid pattern: `.+`  
`phone_number`  
The phone number of the recipient\.  
Required: `true`  
Type: `string`  
Valid pattern: `\+[1-9]+`  
`voice_message_location`  
The URL of the audio content for the voice message\. This must be in TwiML format\. Only voice messages are supported on this topic\. For text messages, use `twilio/txt`\.  
Required: `true`  
Type: `string`  
Valid pattern: `.+`  
`from_number`  
The phone number of the sender\. Twilio uses this phone number to initiate the message\. This property is required if the `DefaultFromPhoneNumber` parameter isn't configured\. If `DefaultFromPhoneNumber` is configured, you can use this property to override the default\.  
Required: `false`  
Type: `string`  
Valid pattern: `\+[1-9]+`  
`retries`  
The number of retries\. The default is 0\.  
Required: `false`  
Type: `integer`  
`id`  
An arbitrary ID for the request\. This property is used to map an input request to an output response\.   
Required: `true`  
Type: `string`  
Valid pattern: `.+`  
**Example input**  

```
{
    "request": {
        "recipient": {
            "name": "Darla",
            "phone_number": "+12345000000",
            "voice_message_location": "https://some-public-TwiML"
        },
        "from_number": "+19999999999",
        "retries": 3
    },
    "id": "request123"
}
```

## Output Data<a name="twilio-notifications-connector-data-output"></a>

This connector publishes status information as output data\.

**Topic filter**  
`twilio/message/status`

**Example output: Success**  

```
{
    "response": {
        "status": "success",
        "payload": {
            "from_number": "+19999999999",
            "messages": {
                "message_status": "queued",
                "to_number": "+12345000000",
                "name": "Darla"
            }
        }
    },
    "id": "request123"
}
```

**Example output: Failure**  

```
{
    "response": {
        "status": "fail",
        "error_message": "Recipient name cannot be None",
        "error": "InvalidParameter",
        "payload": None
        }
    },
    "id": "request123"
}
```
The `payload` property in the output is the response from the Twilio API when the message is sent\. If the connector detects that the input data is invalid \(for example, it doesn't specify a required input field\), the connector returns an error and sets the value to `None`\. The following are example payloads:  

```
{
    'from_number':'+19999999999',
    'messages': {
        'name':'Darla',
        'to_number':'+12345000000',
        'message_status':'undelivered'
    }
}
```

```
{
    'from_number':'+19999999999',
    'messages': {
        'name':'Darla',
        'to_number':'+12345000000',
        'message_status':'queued'
    }
}
```

## Usage Example<a name="twilio-notifications-connector-usage"></a>

The following example Lambda function sends an input message to the connector\. This example triggers a text message\.

**Note**  
This Python function uses the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) to publish an MQTT message\. You can use the following [pip](https://pypi.org/project/pip/) command to install the Python version of the SDK on your core device:   

```
pip install greengrasssdk
```

```
import greengrasssdk
import json

iot_client = greengrasssdk.client('iot-data')
TXT_INPUT_TOPIC = 'twilio/txt'
CALL_INPUT_TOPIC = 'twilio/call'

def publish_basic_message():

    txt = {
        "request": {
            "recipient" : {
                "name": "Darla",
                "phone_number": "+12345000000",
                "message": 'Hello from the edge'
            },
            "from_number" : "+19999999999"
        },
        "id" : "request123"
    }
    
    print "Message To Publish: ", txt

    client.publish(topic=TXT_INPUT_TOPIC,
                   payload=json.dumps(txt))

publish_basic_message()

def function_handler(event, context):
    return
```

## Licenses<a name="twilio-notifications-connector-license"></a>

The Twilio Notifications connector includes the following third\-party software/licensing:
+ [twilio\-python](https://github.com/twilio/twilio-python) / MIT

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## Changelog<a name="twilio-notifications-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 2 | Minor bug fixes and improvements\. | 
| 1 | Initial release\.  | 

A Greengrass group can contain only one version of the connector at a time\.

## See Also<a name="twilio-notifications-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)
+ [Twilio API Reference](https://www.twilio.com/docs/api)