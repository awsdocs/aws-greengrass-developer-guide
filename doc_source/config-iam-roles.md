# Configure IAM Roles<a name="config-iam-roles"></a>

When you create Lambda functions that access AWS services, you must grant sufficient permissions in the Greengrass group role to allow the functions to access those services\. The group role is an IAM role that you attach to your group\.

In this module, you create a role that allows a Greengrass Lambda function to access DynamoDB\. For more information about IAM, see the [AWS Identity and Access Management documentation](https://aws.amazon.com/documentation/iam/)\.

1. In the IAM console, in the navigation pane, choose **Roles**, and then choose **Create Role**\.

1. Under **Select type of trusted entity**, choose **AWS service**\.

1. Under **Choose the service that will use this role**, choose **Greengrass**, and then choose **Next: Permissions**\.

1. On the **Attach permissions policies** page, select the following policies, and then choose **Next: Tags**\.
   + **AWSGreengrassResourceAccessRolePolicy**
   + **AWSGreengrassFullAccess**
   + **AmazonDynamoDBFullAccess**

1. Choose **Next: Review**\. You don't need to create tags for this tutorial\.

1. Enter the following values, and then choose **Create role**\.
   + For **Role name**, enter **Greengrass\_DynamoDB\_Role**\.
   + For **Role description**, enter **Greengrass group role**\.

      
![\[Screenshot of the Review page displaying the role name, description, and selected policies.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-092.png)

1. Repeat the previous step to create the role for the AWS Lambda service\. Select the same policies \(**AWSGreengrassResourceAccessRolePolicy**, **AWSGreengrassFullAccess**, and **AmazonDynamoDBFullAccess**\)\. For **Role name**, enter **Lambda\_DynamoDB\_Role**\.

1. In the AWS IoT Core console, under **Greengrass**, choose **Groups**, and then choose your AWS IoT Greengrass group\.

1. Choose **Settings**, and then choose **Add Role**\.  
![\[Group settings page with Add Role highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-093.png)

1. Choose **Greengrass\_DynamoDB\_Role** from the list of roles, and then choose **Save**\.