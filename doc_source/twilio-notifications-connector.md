# Twilio Notifications connector<a name="twilio-notifications-connector"></a>

The Twilio Notifications [connector](connectors.md) makes automated phone calls or sends text messages through Twilio\. You can use this connector to send notifications in response to events in the Greengrass group\. For phone calls, the connector can forward a voice message to the recipient\.

This connector receives Twilio message information on an MQTT topic, and then triggers a Twilio notification\.

**Note**  
For a tutorial that shows how to use the Twilio Notifications connector, see [Getting started with Greengrass connectors \(console\)](connectors-console.md) or [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)\.

This connector has the following versions\.


| Version | ARN | 
| --- | --- | 
| 5 | `arn:aws:greengrass:region::/connectors/TwilioNotifications/versions/5` | 
| 4 | `arn:aws:greengrass:region::/connectors/TwilioNotifications/versions/4` | 
| 3 | `arn:aws:greengrass:region::/connectors/TwilioNotifications/versions/3` | 
| 2 | `arn:aws:greengrass:region::/connectors/TwilioNotifications/versions/2` | 
| 1 | `arn:aws:greengrass:region::/connectors/TwilioNotifications/versions/1` | 

For information about version changes, see the [Changelog](#twilio-notifications-connector-changelog)\.

## Requirements<a name="twilio-notifications-connector-req"></a>

This connector has the following requirements:

------
#### [ Version 4 \- 5 ]
+ <a name="conn-req-ggc-v1.9.3-secrets"></a>AWS IoT Greengrass Core software v1\.9\.3 or later\. AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This requirement includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ A Twilio account SID, auth token, and Twilio\-enabled phone number\. After you create a Twilio project, these values are available on the project dashboard\.
**Note**  
You can use a Twilio trial account\. If you're using a trial account, you must add non\-Twilio recipient phone numbers to a list of verified phone numbers\. For more information, see [ How to Work with your Free Twilio Trial Account](https://www.twilio.com/docs/usage/tutorials/how-to-use-your-free-trial-account)\.
+ <a name="conn-twilio-req-secret"></a>A text type secret in AWS Secrets Manager that stores the Twilio auth token\. For more information, see [Creating a basic secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.
**Note**  
To create the secret in the Secrets Manager console, enter your token on the **Plaintext** tab\. Don't include quotation marks or other formatting\. In the API, specify the token as the value for the `SecretString` property\.
+ A secret resource in the Greengrass group that references the Secrets Manager secret\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

------
#### [ Versions 1 \- 3 ]
+ <a name="conn-req-ggc-v1.7.0-secrets"></a>AWS IoT Greengrass Core software v1\.7 or later\. AWS IoT Greengrass must be configured to support local secrets, as described in [Secrets Requirements](secrets.md#secrets-reqs)\.
**Note**  
This requirement includes allowing access to your Secrets Manager secrets\. If you're using the default Greengrass service role, Greengrass has permission to get the values of secrets with names that start with *greengrass\-*\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ A Twilio account SID, auth token, and Twilio\-enabled phone number\. After you create a Twilio project, these values are available on the project dashboard\.
**Note**  
You can use a Twilio trial account\. If you're using a trial account, you must add non\-Twilio recipient phone numbers to a list of verified phone numbers\. For more information, see [ How to Work with your Free Twilio Trial Account](https://www.twilio.com/docs/usage/tutorials/how-to-use-your-free-trial-account)\.
+ <a name="conn-twilio-req-secret"></a>A text type secret in AWS Secrets Manager that stores the Twilio auth token\. For more information, see [Creating a basic secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_create-basic-secret.html) in the *AWS Secrets Manager User Guide*\.
**Note**  
To create the secret in the Secrets Manager console, enter your token on the **Plaintext** tab\. Don't include quotation marks or other formatting\. In the API, specify the token as the value for the `SecretString` property\.
+ A secret resource in the Greengrass group that references the Secrets Manager secret\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

------

## Connector Parameters<a name="twilio-notifications-connector-param"></a>

This connector provides the following parameters\.

------
#### [ Version 5 ]

`TWILIO_ACCOUNT_SID`  <a name="twilio-TWILIO_ACCOUNT_SID"></a>
The Twilio account SID that's used to invoke the Twilio API\.  
Display name in the AWS IoT console: **Twilio account SID**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`TwilioAuthTokenSecretArn`  <a name="twilio-TwilioAuthTokenSecretArn"></a>
The ARN of the Secrets Manager secret that stores the Twilio auth token\.  
This is used to access the value of the local secret on the core\.
Display name in the AWS IoT console: **ARN of Twilio auth token secret**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:secretsmanager:[a-z0-9\-]+:[0-9]{12}:secret:([a-zA-Z0-9\\]+/)*[a-zA-Z0-9/_+=,.@\-]+-[a-zA-Z0-9]+`

`TwilioAuthTokenSecretArn-ResourceId`  <a name="twilio-TwilioAuthTokenSecretArn-ResourceId"></a>
The ID of the secret resource in the Greengrass group that references the secret for the Twilio auth token\.  
Display name in the AWS IoT console: **Twilio auth token resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`DefaultFromPhoneNumber`  <a name="twilio-DefaultFromPhoneNumber"></a>
The default Twilio\-enabled phone number that Twilio uses to send messages\. Twilio uses this number to initiate the text or call\.  
+ If you don't configure a default phone number, you must specify a phone number in the `from_number` property in the input message body\.
+ If you do configure a default phone number, you can optionally override the default by specifying the `from_number` property in the input message body\.
Display name in the AWS IoT console: **Default from phone number**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|\+[0-9]+`

`IsolationMode`  <a name="IsolationMode"></a>
The [containerization](connectors.md#connector-containerization) mode for this connector\. The default is `GreengrassContainer`, which means that the connector runs in an isolated runtime environment inside the AWS IoT Greengrass container\.  
The default containerization setting for the group does not apply to connectors\.
Display name in the AWS IoT console: **Container isolation mode**  
Required: `false`  
Type: `string`  
Valid values: `GreengrassContainer` or `NoContainer`  
Valid pattern: `^NoContainer$|^GreengrassContainer$`

------
#### [ Version 1 \- 4 ]

`TWILIO_ACCOUNT_SID`  <a name="twilio-TWILIO_ACCOUNT_SID"></a>
The Twilio account SID that's used to invoke the Twilio API\.  
Display name in the AWS IoT console: **Twilio account SID**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`TwilioAuthTokenSecretArn`  <a name="twilio-TwilioAuthTokenSecretArn"></a>
The ARN of the Secrets Manager secret that stores the Twilio auth token\.  
This is used to access the value of the local secret on the core\.
Display name in the AWS IoT console: **ARN of Twilio auth token secret**  
Required: `true`  
Type: `string`  
Valid pattern: `arn:aws:secretsmanager:[a-z0-9\-]+:[0-9]{12}:secret:([a-zA-Z0-9\\]+/)*[a-zA-Z0-9/_+=,.@\-]+-[a-zA-Z0-9]+`

`TwilioAuthTokenSecretArn-ResourceId`  <a name="twilio-TwilioAuthTokenSecretArn-ResourceId"></a>
The ID of the secret resource in the Greengrass group that references the secret for the Twilio auth token\.  
Display name in the AWS IoT console: **Twilio auth token resource**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`DefaultFromPhoneNumber`  <a name="twilio-DefaultFromPhoneNumber"></a>
The default Twilio\-enabled phone number that Twilio uses to send messages\. Twilio uses this number to initiate the text or call\.  
+ If you don't configure a default phone number, you must specify a phone number in the `from_number` property in the input message body\.
+ If you do configure a default phone number, you can optionally override the default by specifying the `from_number` property in the input message body\.
Display name in the AWS IoT console: **Default from phone number**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|\+[0-9]+`

------

### Create Connector Example \(AWS CLI\)<a name="twilio-notifications-connector-create"></a>

The following example CLI command creates a `ConnectorDefinition` with an initial version that contains the Twilio Notifications connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyTwilioNotificationsConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/TwilioNotifications/versions/5",
            "Parameters": {
                "TWILIO_ACCOUNT_SID": "abcd12345xyz",
                "TwilioAuthTokenSecretArn": "arn:aws:secretsmanager:region:account-id:secret:greengrass-secret-hash",
                "TwilioAuthTokenSecretArn-ResourceId": "MyTwilioSecret",
                "DefaultFromPhoneNumber": "+19999999999",
                "IsolationMode" : "GreengrassContainer"
            }
        }
    ]
}'
```

For tutorials that show how add the Twilio Notifications connector to a group, see [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md) and [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="twilio-notifications-connector-data-input"></a>

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

## Output data<a name="twilio-notifications-connector-data-output"></a>

This connector publishes status information as output data on an MQTT topic\.

<a name="topic-filter"></a>**Topic filter in subscription**  
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

<a name="connectors-setup-intro"></a>Use the following high\-level steps to set up an example Python 3\.7 Lambda function that you can use to try out the connector\.

**Note**  <a name="connectors-setup-get-started-topics"></a>
The [Getting started with Greengrass connectors \(console\)](connectors-console.md) and [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md) topics contain end\-to\-end steps that show how to set up, deploy, and test the Twilio Notifications connector\.

1. Make sure you meet the [requirements](#twilio-notifications-connector-req) for the connector\.

1. <a name="connectors-setup-function"></a>Create and publish a Lambda function that sends input data to the connector\.

   Save the [example code](#twilio-notifications-connector-usage-example) as a PY file\. <a name="connectors-setup-function-sdk"></a>Download and unzip the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core)\. Then, create a zip package that contains the PY file and the `greengrasssdk` folder at the root level\. This zip package is the deployment package that you upload to AWS Lambda\.

   <a name="connectors-setup-function-publish"></a>After you create the Python 3\.7 Lambda function, publish a function version and create an alias\.

1. Configure your Greengrass group\.

   1. <a name="connectors-setup-gg-function"></a>Add the Lambda function by its alias \(recommended\)\. Configure the Lambda lifecycle as long\-lived \(or `"Pinned": true` in the CLI\)\.

   1. <a name="connectors-setup-secret-resource"></a>Add the required secret resource and grant read access to the Lambda function\.

   1. Add the connector and configure its [parameters](#twilio-notifications-connector-param)\.

   1. Add subscriptions that allow the connector to receive [input data](#twilio-notifications-connector-data-input) and send [output data](#twilio-notifications-connector-data-output) on supported topic filters\.
      + <a name="connectors-setup-subscription-input-data"></a>Set the Lambda function as the source, the connector as the target, and use a supported input topic filter\.
      + <a name="connectors-setup-subscription-output-data"></a>Set the connector as the source, AWS IoT Core as the target, and use a supported output topic filter\. You use this subscription to view status messages in the AWS IoT console\.

1. <a name="connectors-setup-deploy-group"></a>Deploy the group\.

1. <a name="connectors-setup-test-sub"></a>In the AWS IoT console, on the **Test** page, subscribe to the output data topic to view status messages from the connector\. The example Lambda function is long\-lived and starts sending messages immediately after the group is deployed\.

   When you're finished testing, you can set the Lambda lifecycle to on\-demand \(or `"Pinned": false` in the CLI\) and deploy the group\. This stops the function from sending messages\.

### Example<a name="twilio-notifications-connector-usage-example"></a>

The following example Lambda function sends an input message to the connector\. This example triggers a text message\.

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
    
    print("Message To Publish: ", txt)

    client.publish(topic=TXT_INPUT_TOPIC,
                   payload=json.dumps(txt))

publish_basic_message()

def lambda_handler(event, context):
    return
```

## Licenses<a name="twilio-notifications-connector-license"></a>

The Twilio Notifications connector includes the following third\-party software/licensing:
+ [twilio\-python](https://github.com/twilio/twilio-python)/MIT

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="twilio-notifications-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 5 | <a name="isolation-mode-changelog"></a>Added the `IsolationMode` parameter to configure the containerization mode for the connector\. | 
| 4 | <a name="upgrade-runtime-py3.7"></a>Upgraded the Lambda runtime to Python 3\.7, which changes the runtime requirement\. | 
| 3 | Fix to reduce excessive logging\. | 
| 2 | Minor bug fixes and improvements\. | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="twilio-notifications-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)
+ [Twilio API Reference](https://www.twilio.com/docs/api)