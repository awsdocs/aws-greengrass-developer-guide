# Configure the Group Role<a name="config-iam-roles"></a>

The group role is an IAM role that you create and attach to your group\. This role contains the permissions that your deployed Lambda functions \(and [connectors](connectors.md)\) use to access AWS services\. For more information about IAM roles, see the [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)\.

In this step, you create a permissions policy that allows describe, create, and update actions on an Amazon DynamoDB table\. Then, you attach the policy to a new role and associate the role with your Greengrass group\.

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

1. Choose **Review policy**\.

1. For **Name**, enter **greengrass\_CarStats\_Table**, and then choose **Create policy**\.

   Next, create a role that uses the new policy\.

1. In the navigation pane, choose **Roles**, and then choose **Create role**\.

1. Under **Select type of trusted entity**, choose **AWS service**\.

1. Under **Choose the service that will use this role**, choose **Greengrass**, and then choose **Next: Permissions**\.

1. Under **Attach permissions policies**, select the new **greengrass\_CarStats\_Table** policy\.  
![\[Screenshot of the Attach permissions policies page with the new policy selected.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-gs-mod6-attach-policy.png)

1. Choose **Next: Tags**, and then choose **Next: Review**\. Tags aren't used in this tutorial\.

1. For **Role name**, enter **Greengrass\_Group\_Role**\.

1. For **Role description**, enter **Greengrass group role for connectors and user\-defined Lambda functions**\.  
![\[Screenshot of the Review page displaying the role name, description, and policies.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-gs-mod6-review-group-role.png)

1. Choose **Create role**\.

   Now, attach the role to your Greengrass group\.

1. In the AWS IoT Core console, under **Greengrass**, choose **Groups**, and then choose your AWS IoT Greengrass group\.

1. Choose **Settings**, and then choose **Add Role**\.  
![\[Group settings page with Add Role highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-093.png)

1. Choose **Greengrass\_Group\_Role** from your list of roles, and then choose **Save**\.