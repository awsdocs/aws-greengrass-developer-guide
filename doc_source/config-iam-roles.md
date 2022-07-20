--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Configure the group role<a name="config-iam-roles"></a>

The group role is an [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) that you create and attach to your Greengrass group\. This role contains the permissions that deployed Lambda functions \(and other AWS IoT Greengrass features\) use to access AWS services\. For more information, see [Greengrass group role](group-role.md)\.

You use the following high\-level steps to create a group role in the IAM console\.

1. Create a policy that allows or denies actions on one or more resources\.

1. Create a role that uses the Greengrass service as a trusted entity\.

1. Attach your policy to the role\.

Then, in the AWS IoT console, you add the role to the Greengrass group\.

**Note**  
A Greengrass group has one group role\. If you want to add permissions, you can edit attached policies or attach more policies\.

 

For this tutorial, you create a permissions policy that allows describe, create, and update actions on an Amazon DynamoDB table\. Then, you attach the policy to a new role and associate the role with your Greengrass group\.

First, create a customer\-managed policy that grants permissions required by the Lambda function in this module\.

1. In the IAM console, in the navigation pane, choose **Policies**, and then choose **Create policy**\.

1. On the **JSON** tab, replace the placeholder content with the following policy\. The Lambda function in this module uses these permissions to create and update a DynamoDB table named `CarStats`\.

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PermissionsForModule6",
               "Effect": "Allow",
               "Action": [
                   "dynamodb:DescribeTable",
                   "dynamodb:CreateTable",
                   "dynamodb:PutItem"
               ],
               "Resource": "arn:aws:dynamodb:*:*:table/CarStats"
           }
       ]
   }
   ```

1. Choose **Next: Tags**, and then choose **Next: Review**\. Tags aren't used in this tutorial\.

1. For **Name**, enter **greengrass\_CarStats\_Table**, and then choose **Create policy**\.

    

   Next, create a role that uses the new policy\.

1. In the navigation pane, choose **Roles**, and then choose **Create role**\.

1. Under **Trusted entity type**, choose **AWS service**\.

1. Under **Use case**, **Use cases for other AWS services** choose **Greengrass**, select **Greengrass**, and then choose **Next**\.

1. Under **Permissions policies**, select the new **greengrass\_CarStats\_Table** policy, and then choose **Next**\.

1. For **Role name**, enter **Greengrass\_Group\_Role**\.

1. For **Description**, enter **Greengrass group role for connectors and user\-defined Lambda functions**\.

1. Choose **Create role**\.

   Now, add the role to your Greengrass group\.

1. <a name="console-gg-groups"></a>In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

1. Under **Greengrass groups**, choose your group\.

1. Choose **Settings**, and then choose **Associate role**\.

1. Choose **Greengrass\_Group\_Role** from your list of roles, and then choose **Associate role**\.