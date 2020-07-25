# Greengrass group role<a name="group-role"></a>

The Greengrass group role is an IAM role that authorizes code running on a Greengrass core to access your AWS resources\. You create the role and manage permissions in AWS Identity and Access Management \(IAM\) and attach the role to your Greengrass group\. A Greengrass group has one group role\. To add or change permissions, you can attach a different role or change the IAM policies that are attached to the role\.

The role must define AWS IoT Greengrass as a trusted entity\. Depending on your business case, the group role might contain IAM policies that define:
+ Permissions for user\-defined [Lambda functions](lambda-functions.md) to access AWS services\.
+ Permissions for [connectors](connectors.md) to access AWS services\.
+ Permissions for [stream manager](stream-manager.md) to export streams to AWS IoT Analytics and Kinesis Data Streams\.
+ Permissions to allow [CloudWatch logging](greengrass-logs-overview.md)\.

The following sections describe how to attach or detach a Greengrass group role in the AWS Management Console or AWS CLI\.
+ [Manage the group role \(console\)](#manage-group-role-console)
+ [Manage the group role \(CLI\)](#manage-group-role-cli)

**Note**  
In addition to the group role that authorizes access from the Greengrass core, you can assign a [Greengrass service role](service-role.md) that allows AWS IoT Greengrass to access AWS resources on your behalf\.

## Managing the Greengrass group role \(console\)<a name="manage-group-role-console"></a>

You can use the AWS IoT console for the following role management tasks:
+ [Find your Greengrass group role](#get-group-role-console)
+ [Add or change the Greengrass group role](#add-or-change-group-role-console)
+ [Remove the Greengrass group role](#remove-group-role-console)

**Note**  
The user who is signed in to the console must have permissions to manage the role\.

 

### Find your Greengrass group role \(console\)<a name="get-group-role-console"></a>

Follow these steps to find the role that is attched to a Greengrass group\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="shared-group-settings"></a>On the group configuration page, choose **Settings**\.  
![\[Group settings page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-settings.png)

If a role is attached to the group, it appears under **Group Role**\.

 

### Add or change the Greengrass group role \(console\)<a name="add-or-change-group-role-console"></a>

Follow these steps to choose an IAM role from your AWS account to add to a Greengrass group\.

A group role has the following requirements:
+ AWS IoT Greengrass defined as a trusted entity\.
+ The permission policies attached to the role must grant the permissions to your AWS resources that are required by the Lambda functions and connectors in the group, and by Greengrass system components\.

Use the IAM console to create and configure the role and its permissions\. For steps that create an example role that allows access to an Amazon DynamoDB table, see [Configure the group role](config-iam-roles.md)\. For general steps, see [Creating a role for an AWS service \(console\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html#roles-creatingrole-service-console) in the *IAM User Guide*\.

 

After the role is configured, use the AWS IoT console to add the role to the group\.

**Note**  
This procedure is required only to choose a role for the group\. It's not required after changing the permissions of the currently selected group role\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="shared-group-settings"></a>On the group configuration page, choose **Settings**\.  
![\[Group settings page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-settings.png)

1. Under **Group Role**, choose to add or change the role:
   + To add the role, choose **Add Role**\.
   + To choose a different role, choose the ellipses \(**…**\) for the role, and then choose **Edit IAM Role**\.

1. On the **Your Group's IAM Role** page, choose the target role from your list of roles, and then choose **Save**\. These are the roles in your AWS account that define AWS IoT Greengrass as a trusted entity\.

 

### Remove the Greengrass group role \(console\)<a name="remove-group-role-console"></a>

Follow these steps to detach the role from a Greengrass group\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="shared-group-settings"></a>On the group configuration page, choose **Settings**\.  
![\[Group settings page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-settings.png)

1. Under **Group Role**, choose the ellipses \(**…**\) for the role, and then choose **Remove IAM Role**\.

   This step removes the role from the group but doesn't delete the role\. If you want to delete the role, use the IAM console\.

## Managing the Greengrass group role \(CLI\)<a name="manage-group-role-cli"></a>

You can use the AWS CLI for the following role management tasks:
+ [Get your Greengrass group role](#get-group-role)
+ [Create the Greengrass group role](#create-group-role)
+ [Remove the Greengrass group role](#remove-group-role)

 

### Get the Greengrass group role \(CLI\)<a name="get-group-role"></a>

Follow these steps to find out if a Greengrass group has an associated role\.

1. <a name="cli-list-groups"></a>Get the ID of the target group from the list of your groups\.

   ```
   aws greengrass list-groups
   ```

   The following is an example `list-groups` response\. Each group in the response includes an `Id` property that contains the group ID\.

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

   For more information, including examples that use the `query` option to filter results, see [Getting the group ID](deployments.md#api-get-group-id)\.

1. <a name="cli-list-groups-copy-id"></a>Copy the `Id` of the target group from the output\.

1. Get the group role\. Replace *group\-id* with the ID of the target group\.

   ```
   aws greengrass get-associated-role --group-id group-id
   ```

   If a role is associated with your Greengrass group, the following role metadata is returned\.

   ```
   {
     "AssociatedAt": "timestamp",
     "RoleArn": "arn:aws:iam::account-id:role/path/role-name"
   }
   ```

   If your group doesn't have an associated role, the following error is returned\.

   ```
   An error occurred (404) when calling the GetAssociatedRole operation: You need to attach an IAM role to this deployment group.
   ```

 

### Create the Greengrass group role \(CLI\)<a name="create-group-role"></a>

Follow these steps to create a role and associate it with a Greengrass group\.

**To create the group role using IAM**

1. Create the role with a trust policy that allows AWS IoT Greengrass to assume the role\. This example creates a role named `MyGreengrassGroupRole`, but you can use a different name\.

------
#### [ Linux, macOS, or Unix ]

   ```
   aws iam create-role --role-name MyGreengrassGroupRole --assume-role-policy-document '{
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "greengrass.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }'
   ```

------
#### [ Windows command prompt ]

   ```
   aws iam create-role --role-name MyGreengrassGroupRole --assume-role-policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"greengrass.amazonaws.com\"},\"Action\":\"sts:AssumeRole\"}]}"
   ```

------

1. Copy the role ARN from the role metadata in the output\. You use the ARN to associate the role with your group\.

1. Attach managed or inline policies to the role to support your business case\. For example, if a user\-defined Lambda function reads from Amazon S3, you might attach the `AmazonS3ReadOnlyAccess` managed policy to the role\.

   ```
   aws iam attach-role-policy --role-name MyGreengrassGroupRole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
   ```

   If successful, no response is returned\.

 

**To associate the role with your Greengrass group**

1. <a name="cli-list-groups"></a>Get the ID of the target group from the list of your groups\.

   ```
   aws greengrass list-groups
   ```

   The following is an example `list-groups` response\. Each group in the response includes an `Id` property that contains the group ID\.

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

   For more information, including examples that use the `query` option to filter results, see [Getting the group ID](deployments.md#api-get-group-id)\.

1. <a name="cli-list-groups-copy-id"></a>Copy the `Id` of the target group from the output\.

1. Associate the role with your group\. Replace *group\-id* with the ID of the target group and *role\-arn* with the ARN of the group role\.

   ```
   aws greengrass associate-role-to-group --group-id group-id --role-arn role-arn
   ```

   If successful, the following response is returned\.

   ```
   {
     "AssociatedAt": "timestamp"
   }
   ```

 

### Remove the Greengrass group role \(CLI\)<a name="remove-group-role"></a>

Follow these steps to disassociate the group role from your Greengrass group\.

1. <a name="cli-list-groups"></a>Get the ID of the target group from the list of your groups\.

   ```
   aws greengrass list-groups
   ```

   The following is an example `list-groups` response\. Each group in the response includes an `Id` property that contains the group ID\.

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

   For more information, including examples that use the `query` option to filter results, see [Getting the group ID](deployments.md#api-get-group-id)\.

1. <a name="cli-list-groups-copy-id"></a>Copy the `Id` of the target group from the output\.

1. Disassociate the role from your group\. Replace *group\-id* with the ID of the target group\.

   ```
   aws greengrass disassociate-role-from-group --group-id group-id
   ```

   If successful, the following response is returned\.

   ```
   {
     "DisassociatedAt": "timestamp"
   }
   ```
**Note**  
You can delete the group role if you're not using it\. First use [https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role-policy.html](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role-policy.html) to detach each managed policy from the role, and then use [https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role.html](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role.html) to delete the role\. For more information, see [Deleting roles or instance profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_delete.html) in the *IAM User Guide*\.

## See also<a name="group-role-see-also"></a>
+ Related topics in the *IAM User Guide*
  + [Creating a role to delegate permissions to an AWS service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html)
  + [Modifying a role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_modify.html)
  + [Adding and removing IAM identity permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html)
  + [Deleting roles or instance profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_delete.html)
+ AWS IoT Greengrass commands in the *AWS CLI Command Reference*
  + [list\-groups](https://docs.aws.amazon.com/cli/latest/reference/greengrass/list-groups.html)
  + [associate\-role\-to\-group](https://docs.aws.amazon.com/cli/latest/reference/greengrass/associate-role-to-group.html)
  + [disassociate\-role\-from\-group](https://docs.aws.amazon.com/cli/latest/reference/greengrass/disassociate-role-from-group.html)
  + [get\-associated\-role](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-associated-role.html)
+ IAM commands in the *AWS CLI Command Reference*
  + [attach\-role\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/attach-role-policy.html)
  + [create\-role](https://docs.aws.amazon.com/cli/latest/reference/iam/create-role.html)
  + [delete\-role](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role.html)
  + [delete\-role\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role-policy.html)