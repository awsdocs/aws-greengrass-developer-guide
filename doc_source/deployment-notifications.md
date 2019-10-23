# Get Deployment Notifications<a name="deployment-notifications"></a>

Using Amazon EventBridge event rules, you can get notifications about state changes for your Greengrass group deployments\. EventBridge delivers a near real\-time stream of system events that describes changes in AWS resources\.

AWS IoT Greengrass emits an event when group deployments change state\. You can create an EventBridge rule that runs for all state transitions or transitions to states you specify\. When a deployment enters a state that triggers a rule, EventBridge invokes the target actions defined in the rule\. This allows you to send notifications, capture event information, take corrective action, or initiate other events in response to a state change\. For example, you can create rules for the following use cases:
+ Trigger post\-deployment operations, such as downloading assets and notifying personnel\.
+ Send notifications when a deployment succeeds or fails\.
+ Publish custom metrics about deployment events\.

AWS IoT Greengrass emits an event when a deployment enters the following states: `Building`, `InProgress`, `Success`, and `Failure`\.

**Note**  
Monitoring the status of a [bulk deployment](bulk-deploy-cli.md) operation is not currently supported\. However, AWS IoT Greengrass emits state\-change events for individual group deployments that are part of a bulk deployment\.

## Group Deployment Status Change Event<a name="events-message-format"></a>

The [event](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/CloudWatchEventsandEventPatterns.html) for a deployment state change uses the following format:

```
{
    "version":"0",
    "id":" cd4d811e-ab12-322b-8255-EXAMPLEb1bc8",
    "detail-type":"Greengrass Deployment Status Change",
    "source":"aws.greengrass",
    "account":"123456789012",
    "time":"2018-03-22T00:38:11Z",
    "region":"us-west-2",
    "resources":[],
    "detail":{    
        "group-id": "284dcd4e-24bc-4c8c-a770-EXAMPLEf03b8",
        "deployment-id": "4f38f1a7-3dd0-42a1-af48-EXAMPLE09681",
        "deployment-type": "NewDeployment|Redeployment|ResetDeployment|ForceResetDeployment",
        "status": "Building|InProgress|Success|Failure"
    }
}
```

You can create rules that apply to one or more groups\. You can filter rules by one or more of the following deployment types and deployment states:

**Deployment types**  
+ `NewDeployment`\. The first deployment of a group version\.
+ `ReDeployment`\. A redeployment of a group version\.
+ `ResetDeployment`\. Deletes deployment information stored in the cloud and on the AWS IoT Greengrass core\. For more information, see [Reset Deployments](reset-deployments-scenario.md)\.
+ `ForceResetDeployment`\. Deletes deployment information stored in the cloud and reports success without waiting for the core to respond\. Also deletes deployment information stored on the core if the core is connected or when it next connects\.

**Deployment states**  
+ `Building`\. The AWS IoT Greengrass cloud service is validating the group configuration and building deployment artifacts\.
+ `InProgress`\. The deployment is in progress on the AWS IoT Greengrass core\.
+ `Success`\. The deployment was successful\.
+ `Failure`\. The deployment failed\.

It's possible that events might be duplicated or out of order\. To determine the order of events, use the `time` property\.

**Note**  
AWS IoT Greengrass doesn't use the `resources` property, so it's always empty\.

## Prerequisites for Creating EventBridge Rules<a name="create-events-rule-prereqs"></a>

Before you create an EventBridge rule for AWS IoT Greengrass, you should do the following:
+ Familiarize yourself with events, rules, and targets in EventBridge\.
+ Create and configure the targets invoked by your EventBridge rules\. Rules can invoke many types of target, including:
  + Amazon SNS topics
  + AWS Lambda functions
  + Kinesis streams
  + Amazon SQS queues

For more information, see [What Is Amazon EventBridge?](https://docs.aws.amazon.com/eventbridge/latest/userguide/what-is-amazon-eventbridge.html) and [Getting Started with Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-getting-set-up.html) in the *Amazon EventBridge User Guide*\.

## Configure Deployment Notifications<a name="create-events-rule-cli"></a>

Use the following steps to create an EventBridge rule that publishes an Amazon SNS topic when the deployment state changes for a group\. This allows web servers, email addresses, and other topic subscribers to respond to the event\.

1. Create the rule\.
   + Replace *group\-id* with the ID of your AWS IoT Greengrass group\.

   ```
   aws events put-rule \
     --name TestRule \
     --event-pattern "{\"source\": [\"aws.greengrass\"], \"detail\": {\"group-id\": [\"group-id\"]}}"
   ```

1. Add the topic as a rule target\.
   + Replace *topic\-arn* with the ARN of your Amazon SNS topic\.

   ```
   aws events put-targets \
     --rule TestRule \
     --targets "Id"="1","Arn"="topic-arn"
   ```
**Note**  
To allow Amazon EventBridge to call your target topic, you must add a resource\-based policy to your topic\. For more information, see [Amazon SNS Permissions](https://docs.aws.amazon.com/eventbridge/latest/userguide/resource-based-policies-eventbridge.html#sns-permissions) in the *Amazon EventBridge User Guide*\.

For more information, see [Events and Event Patterns in EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eventbridge-and-event-patterns.html) in the *Amazon EventBridge User Guide*\.