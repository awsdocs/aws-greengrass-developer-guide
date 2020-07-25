# Greengrass service role<a name="service-role"></a>

The Greengrass service role is an AWS Identity and Access Management \(IAM\) service role that authorizes AWS IoT Greengrass to access resources from AWS services on your behalf\. This makes it possible for AWS IoT Greengrass to perform essential tasks, such as retrieving your AWS Lambda functions and managing AWS IoT shadows\.

To allow AWS IoT Greengrass to access your resources, the Greengrass service role must be associated with your AWS account and specify AWS IoT Greengrass as a trusted entity\. The role must include the [ AWSGreengrassResourceAccessRolePolicy](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/service-role/AWSGreengrassResourceAccessRolePolicy) managed policy or a custom policy that defines equivalent permissions for the AWS IoT Greengrass features that you use\. This policy is maintained by AWS and defines the set of permissions that AWS IoT Greengrass uses to access your AWS resources\.

You can reuse the same Greengrass service role across AWS Regions, but you must associate it with your account in every AWS Region where you use AWS IoT Greengrass\. Group deployment fails if the service role doesn't exist in the current AWS account and Region\.

The following sections describe how to create and manage the Greengrass service role in the AWS Management Console or AWS CLI\.
+ [Manage the service role \(console\)](#manage-service-role-console)
+ [Manage the service role \(CLI\)](#manage-service-role-cli)

**Note**  
In addition to the service role that authorizes service\-level access, you can assign a *group role* to an AWS IoT Greengrass group\. The group role is a separate IAM role that controls how Greengrass Lambda functions and connectors in the group can access AWS services\.

## Managing the Greengrass service role \(console\)<a name="manage-service-role-console"></a>

The AWS IoT console makes it easy to manage your Greengrass service role\. For example, when you create or deploy a Greengrass group, the console checks whether your AWS account is attached to a Greengrass service role in the AWS Region that's currently selected in the console\. If not, the console can create and configure a service role for you\. For more information, see [Create the Greengrass service role \(console\)](#create-service-role-console)\.

You can use the AWS IoT console for the following role management tasks:
+ [Find your Greengrass service role](#get-service-role-console)
+ [Create the Greengrass service role](#create-service-role-console)
+ [Change the Greengrass service role](#update-service-role-console)
+ [Detach the Greengrass service role](#remove-service-role-console)

**Note**  
The user who is signed in to the console must have permissions to view, create, or change the service role\.

 

### Find your Greengrass service role \(console\)<a name="get-service-role-console"></a>

Use the following steps to find the service role that AWS IoT Greengrass is using in the current AWS Region\.

1. <a name="iot-settings"></a>In the [AWS IoT console](https://console.aws.amazon.com/iot/), in the navigation pane, choose **Settings**\.

1. Scroll to the **Greengrass service role** section to see your service role and its policies\.  
![\[The Greengrass service role displayed on the Settings page of the AWS IoT console.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-iot-settings-service-role.png)

   If you don't see a service role, you can let the console create or configure one for you\. For more information, see [Create the Greengrass service role](#create-service-role-console)\.

 

### Create the Greengrass service role \(console\)<a name="create-service-role-console"></a>

The console can create and configure a default Greengrass service role for you\. This role has the following properties\.


| Property | Value | 
| --- | --- | 
| Name | Greengrass\_ServiceRole | 
| Trusted entity | AWS service: greengrass | 
| Policy | [ AWSGreengrassResourceAccessRolePolicy](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/service-role/AWSGreengrassResourceAccessRolePolicy) | 

**Note**  
If [Greengrass device setup](quick-start.md) creates the service role, the role name is `GreengrassServiceRole_random-string`\.

When you create or deploy a Greengrass group from the AWS IoT console, the console checks whether a Greengrass service role is associated with your AWS account in the AWS Region that's currently selected in the console\. If not, the console prompts you to allow AWS IoT Greengrass to read and write to AWS services on your behalf\.

If you grant permission, the console checks whether a role named `Greengrass_ServiceRole` exists in your AWS account\.
+ If the role exists, the console attaches the service role to your AWS account in the current AWS Region\.
+ If the role doesn't exist, the console creates a default Greengrass service role and attaches it to your AWS account in the current AWS Region\.

**Note**  
If you want to create a service role with custom role policies, use the IAM console to create or modify the role\. For more information, see [Creating a role to delegate permissions to an AWS service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) or [Modifying a role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_modify.html) in the *IAM User Guide*\. Make sure that the role grants permissions that are equivalent to the `AWSGreengrassResourceAccessRolePolicy` managed policy for the features and resources that you use\.  
If you create a service role, return to the AWS IoT console and attach the role to the group\. You can do this under **Greengrass service role** on the group's **Settings** page\.

 

### Change the Greengrass service role \(console\)<a name="update-service-role-console"></a>

Use the following procedure to choose a different Greengrass service role to attach to your AWS account in the AWS Region currently selected in the console\.

1. <a name="iot-settings"></a>In the [AWS IoT console](https://console.aws.amazon.com/iot/), in the navigation pane, choose **Settings**\.

1. Under **Greengrass service role**, choose **Choose different role**\.

   The IAM roles in your AWS account that define AWS IoT Greengrass as a trusted entity are displayed in the **Choose the Greengrass service role** dialog box\. 

1. Choose your Greengrass service role\.

1. Choose **Save**\.

**Note**  
To allow the console to create a default Greengrass service role for you, choose **Create role for me** instead of choosing a role from the list\. The **Create role for me** link does not appear if a role named `Greengrass_ServiceRole` is in your AWS account\.

 

### Detach the Greengrass service role \(console\)<a name="remove-service-role-console"></a>

Use the following procedure to detach the Greengrass service role from your AWS account in the AWS Region currently selected in the console\. This revokes permissions for AWS IoT Greengrass to access AWS services in the current AWS Region\.

**Important**  
Detaching the service role might interrupt active operations\.

1. <a name="iot-settings"></a>In the [AWS IoT console](https://console.aws.amazon.com/iot/), in the navigation pane, choose **Settings**\.

1. Under **Greengrass service role**, choose **Detach**\.

1. In the confirmation dialog box, choose **Detach role**\.

**Note**  
If you no longer need the role, you can delete it in the IAM console\. For more information, see [Deleting roles or instance profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_delete.html) in the *IAM User Guide*\.  
Other roles might allow AWS IoT Greengrass to access your resources\. To find all roles that allow AWS IoT Greengrass to assume permissions on your behalf, in the IAM console, on the **Roles** page, look for roles that include **AWS service: greengrass** in the **Trusted entities** column\.

## Managing the Greengrass service role \(CLI\)<a name="manage-service-role-cli"></a>

In the following procedures, we assume that the AWS CLI is installed and configured to use your AWS account ID\. For more information, see [Installing the AWS command line interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) in the *AWS Command Line Interface User Guide*\.

You can use the AWS CLI for the following role management tasks:
+ [Get your Greengrass service role](#get-service-role)
+ [Create the Greengrass service role](#create-service-role)
+ [Remove the Greengrass service role](#remove-service-role)

 

### Get the Greengrass service role \(CLI\)<a name="get-service-role"></a>

Use the following procedure to find out if a Greengrass service role is associated with your AWS account in an AWS Region\.
+ Get the service role\. Replace *region* with your AWS Region \(for example, `us-west-2`\)\.

  ```
  aws greengrass get-service-role-for-account --region region
  ```

  If a Greengrass service role is already associated with your account, the following role metadata is returned\.

  ```
  {
    "AssociatedAt": "timestamp",
    "RoleArn": "arn:aws:iam::account-id:role/path/role-name"
  }
  ```

  If no role metadata is returned, then you must create the service role \(if it doesn't exist\) and associate it with your account in the AWS Region\.

 

### Create the Greengrass service role \(CLI\)<a name="create-service-role"></a>

Use the following steps to create a role and associate it with your AWS account\.

**To create the service role using IAM**

1. Create the role with a trust policy that allows AWS IoT Greengrass to assume the role\. This example creates a role named `Greengrass_ServiceRole`, but you can use a different name\.

------
#### [ Linux, macOS, or Unix ]

   ```
   aws iam create-role --role-name Greengrass_ServiceRole --assume-role-policy-document '{
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
   aws iam create-role --role-name Greengrass_ServiceRole --assume-role-policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"greengrass.amazonaws.com\"},\"Action\":\"sts:AssumeRole\"}]}"
   ```

------

1. Copy the role ARN from the role metadata in the output\. You use the ARN to associate the role with your account\.

1. Attach the `AWSGreengrassResourceAccessRolePolicy` policy to the role\.

   ```
   aws iam attach-role-policy --role-name Greengrass_ServiceRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSGreengrassResourceAccessRolePolicy
   ```

**To associate the service role with your AWS account**
+ Associate the role with your account\. Replace *role\-arn* with the service role ARN and *region* with your AWS Region \(for example, `us-west-2`\)\.

  ```
  aws greengrass associate-service-role-to-account --role-arn role-arn --region region
  ```

  If successful, the following response is returned\.

  ```
  {
    "AssociatedAt": "timestamp"
  }
  ```

 

### Remove the Greengrass service role \(CLI\)<a name="remove-service-role"></a>

Use the following steps to disassociate the Greengrass service role from your AWS account\.
+ Disassociate the service role from your account\. Replace *region* with your AWS Region \(for example, `us-west-2`\)\.

  ```
  aws greengrass disassociate-service-role-from-account --region region
  ```

  If successful, the following response is returned\.

  ```
  {
    "DisassociatedAt": "timestamp"
  }
  ```
**Note**  
You should delete the service role if you're not using it in any AWS Region\. First use [https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role-policy.html](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role-policy.html) to detach the `AWSGreengrassResourceAccessRolePolicy` managed policy from the role, and then use [https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role.html](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role.html) to delete the role\. For more information, see [Deleting roles or instance profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_delete.html) in the *IAM User Guide*\.

## See also<a name="service-role-see-also"></a>
+ [Creating a role to delegate permissions to an AWS service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*
+ [Modifying a role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_modify.html) in the *IAM User Guide*
+ [Deleting roles or instance profiles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_manage_delete.html) in the *IAM User Guide*
+ AWS IoT Greengrass commands in the *AWS CLI Command Reference*
  + [associate\-service\-role\-to\-account](https://docs.aws.amazon.com/cli/latest/reference/greengrass/associate-service-role-to-account.html)
  + [disassociate\-service\-role\-from\-account](https://docs.aws.amazon.com/cli/latest/reference/greengrass/disassociate-service-role-from-account.html)
  + [get\-service\-role\-for\-account](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-service-role-for-account.html)
+ IAM commands in the *AWS CLI Command Reference*
  + [attach\-role\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/attach-role-policy.html)
  + [create\-role](https://docs.aws.amazon.com/cli/latest/reference/iam/create-role.html)
  + [delete\-role](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role.html)
  + [delete\-role\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role-policy.html)