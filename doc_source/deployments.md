# Deploy AWS IoT Greengrass groups to an AWS IoT Greengrass core<a name="deployments"></a>

AWS IoT Greengrass groups are used to organize entities in your edge environment\. Groups also control how the entities in the group can interact with each other and with the AWS Cloud\. For example, only the Lambda functions in the group are deployed for local execution and only the devices in the group can communicate using the local MQTT server\.

A group must include a [core](gg-core.md), which is an AWS IoT device that runs the AWS IoT Greengrass Core software\. The core acts as an edge gateway and provides AWS IoT Core capabilities in the edge environment\. Depending on your business need, you can also add the following entities to a group:
+ **Devices**\. Represented as things in the AWS IoT registry\. These devices must run [FreeRTOS](https://docs.aws.amazon.com/freertos/latest/userguide/freertos-lib-gg-connectivity.html) or use the [AWS IoT Device SDK](what-is-gg.md#iot-device-sdk) or [ AWS IoT Greengrass Discovery API](gg-discover-api.md) to get connection information for the core\. Only devices that are members of the group can connect to the core\.
+ **Lambda functions**\. User\-defined serverless applications that execute code on the core\. Lambda functions are authored in AWS Lambda and referenced from a Greengrass group\. For more information, see [Run Lambda functions on the AWS IoT Greengrass core](lambda-functions.md)\.
+ **Connectors**\. Predefined serverless applications that execute code on the core\. Connectors can provide built\-in integration with local infrastructure, device protocols, AWS, and other cloud services\. For more information, see [Integrate with services and protocols using Greengrass connectors](connectors.md)\.
+ **Subscriptions**\. Defines the publishers, subscribers, and MQTT topics \(or subjects\) that are authorized for MQTT communication\.
+ **Resources**\. References to local [devices and volumes](access-local-resources.md), [machine learning models](ml-inference.md), and [secrets](secrets.md), used for access control by Greengrass Lambda functions and connectors\.
+ **Loggers**\. Logging configurations for AWS IoT Greengrass system components and Lambda functions\. For more information, see [Monitoring with AWS IoT Greengrass logs](greengrass-logs-overview.md)\.

You manage your Greengrass group in the AWS Cloud and then deploy it to a core\. The deployment copies the group configuration to the `group.json` file on the core device\. This file is located in `greengrass-root/ggc/deployments/group`\.

![\[Cloud definition of Greengrass group deployed to a core device.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/group-deploy.png)

**Note**  
During a deployment, the Greengrass daemon process on the core device stops and then restarts\.

## Deploying groups from the AWS IoT console<a name="manage-deployments-console"></a>

You can deploy a group and manage its deployments from the group's configuration page in the AWS IoT console\.

![\[The Deployments page for a Greengrass group.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-deployments.png)

**Note**  
To open this page in the console, choose **Greengrass** and **Groups**, and then choose your group\.

**To deploy the current version of the group**  
+ From **Actions**, choose **Deploy**\.

**To view the deployment history of the group**  
A group's deployment history includes the date and time, group version, and status of each deployment attempt\.  

1. From the navigation pane, choose **Deployments**\.

1. To see more information about a deployment, including error messages, choose the row that contains the deployment\.

**To redeploy a group deployment**  
You might want to redeploy a deployment if the current deployment fails or revert to a different group version\.  

1. From the navigation pane, choose **Deployments**\.

1. On the row that contains the deployment, in the **Status** column, choose the ellipsis \(**…**\), and then choose **Re\-deploy**\.  
![\[Deployments page showing the Re-Deploy action for a deployment.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-redeployment.png)

**To reset group deployments**  
You might want to reset group deployments to move or delete a group or to remove deployment information\. For more information, see [Reset deployments](reset-deployments-scenario.md)\.  
+ From **Actions**, choose **Reset Deployments**\.

## Deploying groups with the AWS IoT Greengrass API<a name="manage-deployments-api"></a>

The AWS IoT Greengrass API provides the following actions to deploy AWS IoT Greengrass groups and manage group deployments\. You can call these actions from the AWS CLI, AWS IoT Greengrass API, or AWS SDK\.


| Action | Description | 
| --- | --- | 
| [CreateDeployment](https://docs.aws.amazon.com/greengrass/latest/apireference/createdeployment-post.html) |  Creates a `NewDeployment` or `Redeployment` deployment\. You might want to redeploy a deployment if the current deployment fails\. Or you might want to redeploy to revert to a different group version\. | 
| [GetDeploymentStatus](https://docs.aws.amazon.com/greengrass/latest/apireference/getdeploymentstatus-get.html) |  Returns the status of a deployment: `Building`, `InProgress`, `Success`, or `Failure`\. You can configure Amazon EventBridge events to receive deployment notifications\. For more information, see [Get deployment notifications](deployment-notifications.md)\. | 
| [ListDeployments](https://docs.aws.amazon.com/greengrass/latest/apireference/listdeployments-get.html) | Returns the deployment history for the group\. | 
| [ResetDeployments](https://docs.aws.amazon.com/greengrass/latest/apireference/resetdeployments-post.html) |  Resets the deployments for the group\. You might want to reset group deployments to move or delete a group or to remove deployment information\. For more information, see [Reset deployments](reset-deployments-scenario.md)\. | 

**Note**  
For information about bulk deployment operations, see [Create bulk deployments for groups](bulk-deploy-cli.md)\.

### Getting the group ID<a name="api-get-group-id"></a>

The group ID is commonly used in API actions\. You can use the [ListGroups](https://docs.aws.amazon.com/greengrass/latest/apireference/listgroups-get.html) action to find the ID of the target group from your list of groups\. For example, in the AWS CLI, use the `list-groups` command\.

```
aws greengrass list-groups
```

You can also include the `query` option to filter results\. For example:
+ To get the most recently created group:

  ```
  aws greengrass list-groups --query "reverse(sort_by(Groups, &CreationTimestamp))[0]"
  ```
+ To get a group by name:

  ```
  aws greengrass list-groups --query "Groups[?Name=='MyGroup']"
  ```

  Group names are not required to be unique, so multiple groups might be returned\.

The following is an example `list-groups` response\. The information for each group includes the ID of the group \(in the `Id` property\) and the ID of the most recent group version \(in the `LatestVersion` property\)\. To get other version IDs for a group, use the group ID with [ListGroupVersions](https://docs.aws.amazon.com/greengrass/latest/apireference/listgroupversions-get.html)\.

**Note**  
<a name="find-group-ids-console"></a>You can also find these values in the AWS IoT console\. The group ID is displayed on the group's **Settings** page\. Group version IDs are displayed on the group's **Deployments** page\.

```
{
    "Groups": [
        {
            "LatestVersionArn": "arn:aws:us-west-2:123456789012:/greengrass/groups/00dedaaa-ac16-484d-ad77-c3eedEXAMPLE/versions/4cbc3f07-fc5e-48c4-a50e-7d356EXAMPLE",
            "Name": "MyFirstGroup",
            "LastUpdatedTimestamp": "2019-11-11T05:47:31.435Z",
            "LatestVersion": "4cbc3f07-fc5e-48c4-a50e-7d356EXAMPLE",
            "CreationTimestamp": "2019-11-11T05:47:31.435Z",
            "Id": "00dedaaa-ac16-484d-ad77-c3eedEXAMPLE",
            "Arn": "arn:aws:us-west-2:123456789012:/greengrass/groups/00dedaaa-ac16-484d-ad77-c3eedEXAMPLE"
        },
        {
            "LatestVersionArn": "arn:aws:us-west-2:123456789012:/greengrass/groups/036ceaf9-9319-4716-ba2a-237f9EXAMPLE/versions/8fe9e8ec-64d1-4647-b0b0-01dc8EXAMPLE",
            "Name": "GreenhouseSensors",
            "LastUpdatedTimestamp": "2020-01-07T19:58:36.774Z",
            "LatestVersion": "8fe9e8ec-64d1-4647-b0b0-01dc8EXAMPLE",
            "CreationTimestamp": "2020-01-07T19:58:36.774Z",
            "Id": "036ceaf9-9319-4716-ba2a-237f9EXAMPLE",
            "Arn": "arn:aws:us-west-2:123456789012:/greengrass/groups/036ceaf9-9319-4716-ba2a-237f9EXAMPLE"
        },
        ...
    ]
}
```

If you don't specify an AWS Region, AWS CLI commands use the default Region from your profile\. To return groups in a different Region, include the *region* option\. For example:

```
aws greengrass list-groups --region us-east-1
```

## Overview of the AWS IoT Greengrass group object model<a name="api-overview"></a>

When programming with the AWS IoT Greengrass API, it's helpful to understand the Greengrass group object model\.

### Groups<a name="api-overview-groups"></a>

In the AWS IoT Greengrass API, the top\-level `Group` object consists of metadata and a list of `GroupVersion` objects\. `GroupVersion` objects are associated with a `Group` by ID\.

![\[A diagram of a group, which consists of metadata and a list of group versions.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/om-group.png)

### Group versions<a name="api-overview-versions"></a>

`GroupVersion` objects define group membership\. Each `GroupVersion` references a `CoreDefinitionVersion` and other component versions by ARN\. These references determine which entities to include in the group\.

![\[A diagram of a group version that references other version types by ARN.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/om-groupversion.png)

For example, to include three Lambda functions, one device, and two subscriptions in the group, the `GroupVersion` references:
+ The `CoreDefinitionVersion` that contains the required core\.
+ The `FunctionDefinitionVersion` that contains the three functions\. 
+ The `DeviceDefinitionVersion` that contains the device\.
+ The `SubscriptionDefinitionVersion` that contains the two subscriptions\.

The `GroupVersion` deployed to a core device determines the entities that are available in the local environment and how they can interact\.

### Group components<a name="api-overview-group-components"></a>

Components that you add to groups have a three\-level hierarchy:
+ A *Definition* that references a list of *DefinitionVersion* objects of a given type\. For example, a `DeviceDefinition` references a list of `DeviceDefinitionVersion` objects\.
+ A *DefinitionVersion* that contains a set of entities of a given type\. For example, a `DeviceDefinitionVersion` contains a list of `Device` objects\.
+ Individual entities that define their properties and behavior\. For example, a `Device` defines the ARN of the corresponding device in the AWS IoT registry, the ARN of its device certificate, and whether its local shadow syncs automatically with the cloud\.

  You can add the following types of entities to a group:
  + [Connector](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-connector.html)
  + [Core](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-core.html)
  + [Device](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-device.html)
  + [Function](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-function.html)
  + [Logger](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-logger.html)
  + [Resource](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-resource.html)
  + [Subscription](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-subscription.html)

The following example `DeviceDefinition` references three `DeviceDefinitionVersion` objects that each contain multiple `Device` objects\. Only one `DeviceDefinitionVersion` at a time is used in a group\.

![\[A diagram of a device hierarchy, which consists of DeviceDefinition, DeviceDefinitionVersion, and Device objects.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/om-devicedefinition.png)

### Updating groups<a name="api-update-groups"></a>

In the AWS IoT Greengrass API, you use versions to update a group's configuration\. Versions are immutable, so to add, remove, or change group components, you must create *DefinitionVersion* objects that contain new or updated entities\.

You can associate new *DefinitionVersions* objects with new or existing *Definition* objects\. For example, you can use the `CreateFunctionDefinition` action to create a `FunctionDefinition` that includes the `FunctionDefinitionVersion` as an initial version, or you can use the `CreateFunctionDefinitionVersion` action and reference an existing `FunctionDefinition`\.

After you create your group components, you create a `GroupVersion` that contains all *DefinitionVersion* objects that you want to include in the group\. Then, you deploy the `GroupVersion`\.

To deploy a `GroupVersion`, it must reference a `CoreDefinitionVersion` that contains exactly one `Core`\. All referenced entities must be members of the group\. Also, a [Greengrass service role](service-role.md) must be associated with your AWS account in the AWS Region where you are deploying the `GroupVersion`\.

**Note**  
The `Update` actions in the API are used to change the name of a `Group` or component *Definition* object\.

**Updating entities that reference AWS resources**

Greengrass Lambda functions and [secret resources](secrets.md) define Greengrass\-specific properties and also reference corresponding AWS resources\. To update these entities, you might make changes to the corresponding AWS resource instead of your Greengrass objects\. For example, Lambda functions reference a function in AWS Lambda and also define lifecycle and other properties that are specific to the Greengrass group\.
+ To update Lambda function code or packaged dependencies, make your changes in AWS Lambda\. During the next group deployment, these changes are retrieved from AWS Lambda and copied to your local environment\.
+ To update [Greengrass\-specific properties](lambda-group-config.md), you create a `FunctionDefinitionVersion` that contains the updated `Function` properties\.

**Note**  
Greengrass Lambda functions can reference a Lambda function by alias ARN or version ARN\. If you reference the alias ARN \(recommended\), you don't need to update your `FunctionDefinitionVersion` \(or `SubscriptionDefinitionVersion`\) when you publish a new function version in AWS Lambda\. For more information, see [Reference Lambda functions by alias or version](lambda-functions.md#lambda-versions-aliases)\.

## See also<a name="deployments-see-also"></a>
+ [Get deployment notifications](deployment-notifications.md)
+ [Reset deployments](reset-deployments-scenario.md)
+ [Create bulk deployments for groups](bulk-deploy-cli.md)
+ [Troubleshooting Deployment Issues](gg-troubleshooting.md#gg-troubleshooting-deploymentissues)<a name="see-also-gg-api-cli"></a>
+ [AWS IoT Greengrass API Reference](https://docs.aws.amazon.com/greengrass/latest/apireference/)
+ <a name="see-also-gg-cli"></a>[AWS IoT Greengrass commands](https://docs.aws.amazon.com/cli/latest/reference/greengrass/index.html) in the *AWS CLI Command Reference*