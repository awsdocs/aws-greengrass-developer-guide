# Greengrass Service Role<a name="service-role"></a>

The Greengrass service role is an AWS Identity and Access Management \(IAM\) service role that authorizes AWS IoT Greengrass to access resources from AWS services on your behalf\. This makes it possible for AWS IoT Greengrass to perform essential tasks, such as retrieving your AWS Lambda functions and managing AWS IoT shadows\.

To allow AWS IoT Greengrass to access your resources, the Greengrass service role must be associated with your AWS account and specify AWS IoT Greengrass as a trusted entity\. The role must also include the `AWSGreengrassResourceAccessRolePolicy` managed policy \(which defines the set of permissions required by AWS IoT Greengrass\) or define equivalent permissions\.

You can reuse the same service role in multiple AWS Regions, but you must associate it with your AWS account in every AWS Region where you use AWS IoT Greengrass\. Group deployment fails if the service role doesn't exist in the current AWS account and Region\.

**Note**  
In addition to the service role that authorizes service\-level access, you can assign a *group role* to an AWS IoT Greengrass group\. This is an IAM role that controls how Greengrass Lambda functions and connectors in the group can access AWS services\.

In the following procedures, we assume that the AWS CLI is installed and configured to use your AWS account ID\. For more information, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) in the *AWS Command Line Interface User Guide*\.

## Getting the Greengrass Service Role<a name="get-service-role"></a>

Use the following procedure to find out if a Greengrass service role is associated with your AWS account in an AWS Region\.
+ Get the service role\. Replace *region* with your AWS Region \(for example, `us-west-2`\)\.

  ```
  aws greengrass get-service-role-for-account --region region
  ```

  If a Greengrass service role is already associated with your account, the following role metadata is returned\.

  ```
  {
    "AssociatedAt": "time-stamp",
    "RoleArn": "arn:aws:iam::account-id:role/service-role/role-name"
  }
  ```

  If no role metadata is returned, then you must create the service role \(if it doesn't exist\) and associate it with your account in the AWS Region\.

## Creating the Greengrass Service Role<a name="create-service-role"></a>

Use the following procedures to create a Greengrass service role and associate it with your AWS account\.

You can create one Greengrass service role that applies across AWS Regions, but you must associate it with your account in every AWS Region where you use AWS IoT Greengrass\.

**To create the service role in IAM**

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
#### [ Windows Command Prompt ]

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
    "AssociatedAt": "time-stamp"
  }
  ```

## Removing the Greengrass Service Role<a name="remove-service-role"></a>

Use the following procedure to disassociate the Greengrass service role from your AWS account\.
+ Disassociate the service role from your account\. Replace *region* with your AWS Region \(for example, `us-west-2`\)\.

  ```
  aws greengrass disassociate-service-role-from-account --region region
  ```

  If successful, the following response is returned\.

  ```
  {
    "DisassociatedAt": "time-stamp"
  }
  ```

## See Also<a name="service-role-see-also"></a>
+ [Creating a Role to Delegate Permissions to an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the *IAM User Guide*
+ AWS IoT Greengrass commands in the *AWS CLI Command Reference*
  + [associate\-service\-role\-to\-account](https://docs.aws.amazon.com/cli/latest/reference/greengrass/associate-service-role-to-account.html)
  + [disassociate\-service\-role\-from\-account](https://docs.aws.amazon.com/cli/latest/reference/greengrass/disassociate-service-role-from-account.html)
  + [get\-service\-role\-for\-account](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-service-role-for-account.html)
+ IAM commands in the *AWS CLI Command Reference*
  + [attach\-role\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/attach-role-policy.html)
  + [create\-role](https://docs.aws.amazon.com/cli/latest/reference/iam/create-role.html)
  + [delete\-role](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role.html)
  + [delete\-role\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/delete-role-policy.html)