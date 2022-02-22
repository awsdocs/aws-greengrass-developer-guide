--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

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

1. Under **Select type of trusted entity**, choose **AWS service**\.

1. Under **Choose the service that will use this role**, choose **Greengrass**, and then choose **Next: Permissions**\.

1. Under **Attach permissions policies**, select the new **greengrass\_CarStats\_Table** policy\.  
![\[Screenshot of the Attach permissions policies page with the new policy selected.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-gs-mod6-attach-policy.png)

1. Choose **Next: Tags**, and then choose **Next: Review**\.

1. For **Role name**, enter **Greengrass\_Group\_Role**\.

1. For **Role description**, enter **Greengrass group role for connectors and user\-defined Lambda functions**\.  
![\[Screenshot of the Review page displaying the role name, description, and policies.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-gs-mod6-review-group-role.png)

1. Choose **Create role**\.

    

   Now, add the role to your Greengrass group\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, in the navigation pane, choose **Greengrass**, **Classic \(V1\)**, **Groups**\.

1. Under **Greengrass groups**, choose your group\.

1. Choose **Settings**, and then choose **Add Role**\.  
![\[Group settings page with Add Role highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-093.png)

1. Choose **Greengrass\_Group\_Role** from your list of roles, and then choose **Save**\.