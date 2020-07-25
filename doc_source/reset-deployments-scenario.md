# Reset deployments<a name="reset-deployments-scenario"></a>

This feature is available for AWS IoT Greengrass Core v1\.1 and later\.

You might want to reset a group's deployments to:
+ Delete the group \(for example, when the group's core has been reimaged\.\)
+ Move the group's core to a different group\.
+ Revert the group to its state before any deployments\.
+ Remove the deployment configuration from the core device\.
+ Delete sensitive data from the core device or from the cloud\.
+ Deploy a new group configuration to a core without having to replace the core with another in the current group\.

**Note**  
Reset deployments functionality is not available in AWS IoT Greengrass Core Software v1\.0\.0\. You cannot delete a group that has been deployed using v1\.0\.0\.

The reset deployments operation first cleans up all deployment information stored in the cloud for a given group\. It then instructs the group's core device to clean up all of its deployment related information as well \(Lambda functions, user logs, shadow database and server certificate, but not the user\-defined `config.json` or the Greengrass core certificates\)\. You cannot initiate a reset of deployments for a group if the group currently has a deployment with status of `In Progress` or `Building`\.

## Reset deployments from the AWS IoT console<a name="reset-deployments-console"></a>

You can reset group deployments from group configuration page in the AWS IoT console\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. Choose the target group\.

1. From **Actions**, choose **Reset Deployments**\.  
![\[The Deployments page for a Greengrass group.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-deployments.png)

## Reset deployments with the AWS IoT Greengrass API<a name="reset-deployments-api"></a>

You can use the `ResetDeployments` action in the AWS CLI, AWS IoT Greengrass API, or AWS SDK to reset deployments\. The examples in this topic use the CLI\.

```
aws greengrass reset-deployments --group-id GroupId [--force]
```Arguments for the `reset-deployments` CLI command:

`--group-id`  
The group ID\. Use the `list-groups` command to get this value\.

`--force`  
Optional\. Use this parameter if the group's core device has been lost, stolen, or destroyed\. This option causes the reset deployment process to report success after all deployment information in the cloud has been cleaned up, without waiting for a core device to respond\. However, if the core device is or becomes active, it also performs cleanup operations\.

The output of the `reset-deployments` CLI command looks like this:

```
{
    "DeploymentId": "4db95ef8-9309-4774-95a4-eea580b6ceef",
    "DeploymentArn": "arn:aws:greengrass:us-west-2:106511594199:/greengrass/groups/b744ed45-a7df-4227-860a-8d4492caa412/deployments/4db95ef8-9309-4774-95a4-eea580b6ceef"
}
```

You can check the status of the reset deployment with the `get-deployment-status` CLI command:

```
aws greengrass get-deployment-status --deployment-id DeploymentId --group-id GroupId
```Arguments for the `get-deployment-status` CLI command:

`--deployment-id`  
The deployment ID\.

`--group-id`  
The group ID\.

The output of the `get-deployment-status` CLI command looks like this:

```
{
    "DeploymentStatus": "Success",
    "UpdatedAt": "2017-04-04T00:00:00.000Z"
}
```

The `DeploymentStatus` is set to `Building` when the reset deployment is being prepared\. When the reset deployment is ready but the AWS IoT Greengrass core has not picked up the reset deployment, the `DeploymentStatus` is `InProgress`\.

If the reset operation fails, error information is returned in the response\.

## See also<a name="reset-deployments-see-also"></a>
+ [Deploy AWS IoT Greengrass groups to an AWS IoT Greengrass core](deployments.md)
+ [ResetDeployments ](https://docs.aws.amazon.com/greengrass/latest/apireference/resetdeployments-post.html) in the *AWS IoT Greengrass API Reference*
+ [GetDeploymentStatus ](https://docs.aws.amazon.com/greengrass/latest/apireference/getdeploymentstatus-get.html) in the *AWS IoT Greengrass API Reference*