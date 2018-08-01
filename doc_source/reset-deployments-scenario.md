# Reset Deployments<a name="reset-deployments-scenario"></a>

This feature is available for AWS Greengrass Core v1\.1\.0 and later\.

You may want to reset a group's deployments in order to:
+ Delete the group \(for example, when the group's core has been reimaged\.\)
+ Move the group's core to a different group\.
+ Revert the group to its state prior to any deployments\.
+ Remove the deployment configuration from the core device\.
+ Delete sensitive data from the core device or from the cloud\.
+ Deploy a new group configuration to a core without having to replace the core with another in the current group\.

**Note**  
The Reset Deployments feature is not available in AWS Greengrass Core Software v1\.0\.0\. Also, note that it's not possible to delete a group that has been deployed using v1\.0\.0\.

The `ResetDeployments` command will clean up all deployment information which is stored in the cloud for a given group\. It will then instruct the group's core device to clean up all of its deployment related information as well \(Lambda functions, user logs, shadow database and server certificate, but not the user defined config\.json or the Greengrass core certificates\.\) You cannot initiate a reset of deployments for a group if the group currently has a deployment with status `Pending` or `Building`\.

```
aws greengrass reset-deployments --group-id <GroupId> [--force]
```Arguments for the `reset-deployments` CLI command:

`--group-id`  
The group ID\.

`--force`  
\[Optional\] Use this parameter if the group's core device has been lost, stolen or destroyed\. This option causes the reset deployment process to report success once all deployment information in the cloud has been cleaned up, without waiting for a core device to respond\. However, if the core device is or becomes active, it will perform its clean up operations as well\.

The output of the `reset-deployments` CLI command will look like this:

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

The output of the `get-deployment-status` CLI command will look like this:

```
{
    "DeploymentStatus": "Success",
    "UpdatedAt": "2017-04-04T00:00:00.000Z"
}
```

The `DeploymentStatus` is set to `Building` when the reset deployment is being prepared\. When the reset deployment is ready but the AWS Greengrass core has not picked up the reset deployment, the `DeploymentStatus` is `InProgress`\.