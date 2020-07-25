# Test long\-lived Lambda functions<a name="long-testing"></a>

A *[long\-lived](lambda-functions.md#lambda-lifecycle)* Lambda function starts automatically when the AWS IoT Greengrass core starts and runs in a single container \(or sandbox\)\. Any variables and preprocessing logic defined outside of the function handler are retained for every invocation of the function handler\. Multiple invocations of the function handler are queued until earlier invocations have been executed\.

 The `greengrassHelloWorldCounter.py` code used in this module defines a `my_counter` variable outside of the function handler\.

**Note**  
You can view the code in the AWS Lambda console or in the [AWS IoT Greengrass Core SDK for Python](https://github.com/aws/aws-greengrass-core-sdk-python/blob/master/examples/HelloWorldCounter/greengrassHelloWorldCounter.py) on GitHub\.

 

In this step, you create subscriptions that allow the Lambda function and AWS IoT to exchange MQTT messages\. Then you deploy the group and test the function\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.

1. Under **Select a source**, choose **Lambdas**, and then choose **Greengrass\_HelloWorld\_Counter**\.

1. Under **Select a target**, choose **Services**, choose **IoT Cloud**, and then choose **Next**\.  
![\[Select your source and target screenshot with Greengrass_HelloWorld_Counter, IoT Cloud, and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-052.png)

1. For **Topic filter**, enter **hello/world/counter**\. Choose **Next**, and then choose **Finish**\.  
![\[Screenshot with hello/world/counter and the Next button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-053.png)

   This single subscription goes in one direction only: from the `Greengrass_HelloWorld_Counter` Lambda function to AWS IoT\. To invoke \(or trigger\) this Lambda function from the cloud, you must create a subscription in the opposite direction\.

1. Follow steps 1 \- 4 to add another subscription that uses the following values\. This subscription allows the Lambda function to receive messages from AWS IoT\. You use this subscription when you send a message from the AWS IoT console that invokes the function\.
   + For the source, choose **Services**, and then choose **IoT Cloud**\.
   + For the target, choose **Lambdas**, and then choose **Greengrass\_HelloWorld\_Counter**\.
   + For the topic filter, enter **hello/world/counter/trigger**\.

      
![\[Screenshot of the Confirm and save your Subscription page with IoT Cloud, hello/world/counter/trigger, Greengrass_HelloWorld_Counter, and Finish highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-054.png)

   The `/trigger` extension is used in this topic filter because you created two subscriptions and don't want them to interfere with each other\.

1. Make sure that the Greengrass daemon is running, as described in [Deploy cloud configurations to a core device](configs-core.md)\.

1. <a name="console-actions-deploy"></a>On the group configuration page, from **Actions**, choose **Deploy**\.  
![\[Screenshot of the Group page with the Deploy action highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-040.png)

   This deploys the group configuration to your AWS IoT Greengrass core device\. For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

1. <a name="console-test-after-deploy"></a>After your deployment is complete, return to the AWS IoT console home page and choose **Test**\.

1. Configure the following fields:
   + For **Subscription topic**, enter **hello/world/counter**\.
   + For **Quality of Service**, choose **0**\.
   + For **MQTT payload display**, choose **Display payloads as strings**\.

      
![\[Subscriptions screenshot with hello/world/counter, 0, Display payloads as strings, and the Subscribe to topic button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-056.png)

1. Choose **Subscribe to topic**\.

   Unlike [Part 1](module3-I.md) of this module, you shouldn't see any messages after you subscribe to `hello/world/counter`\. This is because the `greengrassHelloWorldCounter.py` code that publishes to the `hello/world/counter` topic is inside the function handler, which runs only when the function is invoked\.

   In this module, you configured the `Greengrass_HelloWorld_Counter` Lambda function to be invoked when it receives an MQTT message on the `hello/world/counter/trigger` topic\. You can see this by examining the related subscriptions:  
![\[Subscriptions webpage showing greengrass_HelloWorld_Counter related subscriptions.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-056.5.png)

   The **Greengrass\_HelloWorld\_Counter** to **IoT Cloud** subscription allows the function to send messages to AWS IoT on the `hello/world/counter` topic\. The **IoT Cloud** to **Greengrass\_HelloWorld\_Counter** subscription allows AWS IoT to send messages to the function on the `hello/world/counter/trigger` topic\.

1. To test the long\-lived lifecycle, invoke the Lambda function by publishing a message to the `hello/world/counter/trigger` topic\. You can use the default message\.  
![\[Default Hello from AWS IoT console message sent to hello/world/counter/trigger with the Publish to topic button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-057.png)
**Note**  
 The `Greengrass_HelloWorld_Counter` function ignores the content of received messages\. It just runs the code in `function_handler`, which sends a message to the `hello/world/counter` topic\. You can review this code from the [AWS IoT Greengrass Core SDK for Python](https://github.com/aws/aws-greengrass-core-sdk-python/blob/master/examples/HelloWorldCounter/greengrassHelloWorldCounter.py) on GitHub\.

Every time a message is published to the `hello/world/counter/trigger` topic, the `my_counter` variable is incremented\. This invocation count is shown in the messages sent from the Lambda function\. Because the function handler includes a 20\-second sleep cycle \(`time.sleep(20)`\), repeatedly triggering the handler queues up responses from the AWS IoT Greengrass core\.

![\[Screenshot showing the incrementing of Invocation Count from 1, 2, and 3.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-058.png)