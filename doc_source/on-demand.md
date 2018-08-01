# Test On\-Demand Lambda Functions<a name="on-demand"></a>

An *on\-demand Lambda function* is similar in functionality to an AWS cloud Lambda function\. Multiple invocations of an on\-demand Lambda function can run in parallel\. An invocation of the Lambda function creates a separate container to process invocations or reuses an existing container if resources permit\. Any variables or preprocessing that are defined outside of the function handler are not retained when new containers are created\. For more information, see [Lifecycle Configuration for Greengrass Lambda Functions](lambda-functions.md#lambda-lifecycle)\.

1. On the group configuration page, choose **Lambdas**\. For the **Greengrass\_HelloWorld\_Counter** Lambda function, choose **Edit Configuration**\.   
![\[Screenshot with Lambdas and Edit Configuration highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-059.png)

1. Under **Lambda lifecycle**, select **On\-demand function**\.  
![\[Screenshot with the On-demand function radio button selected.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-060.png)

   Next, choose **Update**\.

1. On the group configuration page, from the **Actions** menu, choose **Deploy** to deploy the updated group configuration to your AWS Greengrass core device\.  
![\[Screenshot of Deploy highlighted under the Actions drop-down menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-061.png)

1. After your deployment is complete, in the AWS IoT console, choose **Test**\. For **Subscription topic**, type **hello/world/counter**\. For **Quality of Service**, select **0**\. For **MQTT payload display**, select **Display payloads as strings** and then choose **Subscribe to topic**\.  
![\[Screenshot of Subscriptions page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-062.png)

   Again, you should not be able to see any messages after you subscribe\. Trigger the function to the **hello/world/counter/trigger** topic by sending any message \(the default message is fine\), then choose **Publish to topic** three times, *within five seconds* of each press of the button\.  
![\[Screenshot showing the Publish to topic button, which must be clicked rapidly three times.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-063.png)

   Each publish is triggering the function handler and creating a new container for each invocation\. The invocation count is not incremented for each of the three times you triggered the function because each on\-demand Lambda function has its own container/sandbox\.  
![\[Screenshot showing Invocation Count fixed at 1.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-064.png)

   Wait approximately thirty seconds or more, and then choose **Publish to topic**\. This time you should see an incremented invocation count\.  
![\[Screenshot showing Invocation Count now at 2.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-065.png)

    This shows that a container, first created from a prior invocation, is being reused, and preprocessing variables outside of the function handler have been stored\.

   You should now understand the two types of Lambda functions that can run on the AWS Greengrass core\. The next module, [Module 4](module4.md), shows you how devices can interact within an AWS Greengrass group\.