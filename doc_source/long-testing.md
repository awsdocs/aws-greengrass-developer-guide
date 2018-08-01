# Test Long\-Lived Lambda Functions<a name="long-testing"></a>

A *long\-lived Lambda function* starts automatically when the AWS Greengrass core starts and runs in a single container \(or sandbox\)\. Any variables or preprocessing that are defined outside of the function handler are retained for every invocation of the function handler\. Multiple invocations of the function handler are queued until earlier invocations have been executed\. The `greengrassHelloWorldCounter.py` Lambda function is similar to the `greengrassHelloWorld.py` function except there is a variable, `my_counter`, that is outside of the `function_handler(event, context)` method \(code comments were removed for brevity\):

```
import greengrasssdk
import platform
import time
import json

client = greengrasssdk.client('iot-data')

my_platform = platform.platform()

my_counter = 0

def function_handler(event, context):
    global my_counter
    my_counter = my_counter + 1
    if not my_platform:
        client.publish(
            topic='hello/world/counter',
            payload=json.dumps({'message': 'Hello world! Sent from Greengrass Core.  Invocation Count: {}'.format(my_counter)})
        )
    else:
        client.publish(
            topic='hello/world/counter',
            payload=json.dumps({'message': 'Hello world! Sent from Greengrass Core running on platform: {}.  Invocation Count: {}'
                                .format(my_platform, my_counter)})
        )
    time.sleep(20)
    return
```

1. On the group configuration page, choose **Subscriptions**, then **Add Subscription**\. Under **Select a source**, choose the **Lambdas** tab, then choose **Greengrass\_HelloWorld\_Counter**\. Next, under **Select a target**, choose the **Services** tab, choose **IoT Cloud**, and then choose **Next**\.  
![\[Select your source and target screenshot with Greengrass_HelloWorld_Counter, IoT Cloud, and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-052.png)

   For **Optional topic filter**, type **hello/world/counter**\. Choose **Next** and then choose **Finish**\.  
![\[Screenshot with hello/world/counter and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-053.png)

   This single subscription goes in one direction only: from the **Greengrass\_HelloWorld\_Counter** Lambda function to the AWS IoT cloud\. To trigger this Lambda function from the cloud, you need to create a subscription in the opposite direction\.

1. Add another subscription with **IoT Cloud** as the source and **Greengrass\_HelloWorld\_Counter** as the target\. Use the **hello/world/counter/trigger** topic:  
![\[Screenshot of the Confirm and save your Subscription page with IoT Cloud, hello/world/counter/trigger, Greengrass_HelloWorld_Counter, and Finish highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-054.png)

   Note the **/trigger** extension above – because you have created two subscriptions, you do not want them to interfere with each other\.

1. Make sure that the AWS Greengrass daemon is running, as described in [Deploy Cloud Configurations to a Core Device](configs-core.md)\.

   Note that with the daemon running, the prior `greengrassHelloWorld.py` Lambda function will continue to send messages to the `hello/world` topic \(in the AWS IoT cloud\)\. This does not, however, interfere with the messages sent from the `greengrassHelloWorldCounter.py` Lambda function to the AWS IoT cloud, since they're directed to a different topic, namely `hello/world/counter`\.

1. On the group configuration page, from the **Actions** menu, choose **Deploy** to deploy the updated group configuration to your AWS Greengrass core device\.   
![\[Deployments and Deploy (under the Actions menu) are highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-055.png)

   For help troubleshooting any issues that you encounter, see [Troubleshooting AWS Greengrass Applications](gg-troubleshooting.md)\.

1. After your deployment is complete, in the AWS IoT console, choose **Test**\. In **Subscription topic**, type **hello/world/counter**\. For **Quality of Service**, select **0**\. For **MQTT payload display**, select **Display payloads as strings**, and then choose **Subscribe to topic**\.  
![\[Subscriptions screenshot with hello/world/counter, 0, Display payloads as string, and the Subscribe to topic button all highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-056.png)

   Unlike [Part 1](module3-I.md) of this module, you should *not* be able to see any messages after you subscribe to `hello/world/counter`\. This is because the `greengrassHelloWorldCounter.py` code to publish to the topic `hello/world/counter` is inside the `function_handler(event, context)` function, and `function_handler(event, context)` is triggered only when it receives an MQTT message on the **hello/world/counter/trigger** topic\. To help further explain this, consider the **greengrass\_HelloWorld\_Counter** related subscriptions:  
![\[Subscriptions webpage showing greengrass_HelloWorld_Counter related subscriptions.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-056.5.png)

   In the second row, we see that the **greengrass\_HelloWorld\_Counter** Lambda function can send messages to the **IoT Cloud** on the **hello/world/counter** topic\. In the third row, we see that the **IoT Cloud** will can send messages to the **greengrass\_HelloWorld\_Counter** Lambda function when that message is sent to the **hello/world/counter/trigger** topic \(note that there is nothing special about the word **trigger**\)\. The **greengrass\_HelloWorld\_Counter** Lambda function ignores these sent messages and merely runs the code within `function_handler(event, context)`, which sends a message back to the **hello/world/counter** topic in the AWS IoT cloud \(see the prior `greengrassHelloWorldCounter.py` code listing\)\.

   So, to trigger the `function_handler(event, context)` handler, publish any message \(the default message is fine\) to the **hello/world/counter/trigger** topic, as shown next\.  
![\[Default Hello from AWS IoT console message sent to hello/world/counter/trigger with the Publish to topic button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-057.png)

   Every time a message is published to the `hello/world/counter/trigger` topic, the `my_counter` variable is incremented \(see `Invocation Count` in the following\)\. Because the function handler in the Lambda function includes a 20\-second sleep cycle \(i\.e\., `time.sleep(20)`\), repeatedly triggering the handler queues up responses from the AWS Greengrass core\.  
![\[Screenshot showing the incrementing of Invocation Count from 1, 2, and 3.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-058.png)