# Configure IAM Roles<a name="config-iam-roles"></a>

1. Because you are creating a Lambda function that accesses other AWS services, you need to create an IAM role that has access to DynamoDB and AWS Greengrass\. For more information about IAM, see the [AWS Identity and Access Management documentation](https://aws.amazon.com/documentation/iam/)\.

   In the IAM console, choose **Roles**, and then choose **Create Role**:  
![\[Screenshot of Roles page with Create role highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-090.png)

   Choose **AWS service**, and then choose **Greengrass**:  
![\[Screenshot of Choose the service that will use this role with Greengrass highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-091.png)

   Choose **Next: Permissions**\.

   On the **Attach permissions policies** page, select the following policies: **AWSGreengrassResourceAccessRolePolicy**, **AWSGreengrassFullAccess**, and **AmazonDynamoDBFullAccess**\. 

   Next, choose **Next: Review**\. For **Role name**, type **Greengrass\_DynamoDB\_Role**, and then choose **Create role**\.  
![\[Screenshot of the Trusted entities field displaying AWSGreengrassResourceAccessRolePolicy, AWSGreengrassFullAccess, and AmazonDynamoDBFullAccess.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-092.png)

1. Repeat the prior step to create the role for the AWS Lambda service \(instead of the AWS Greengrass service\)\. Give the role the same policies \(**AWSGreengrassResourceAccessRolePolicy**, **AWSGreengrassFullAccess**, and **AmazonDynamoDBFullAccess**\)\. For **Role name**, type **Lambda\_DynamoDB\_Role**\.

1. In the AWS IoT console, under **Greengrass**, choose **Groups**, and choose your AWS Greengrass group\. Choose **Settings**, and then choose **Add Role**:  
![\[My1stGroup screenshot with Settings and Add Role highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-093.png)

   The IAM role you just created should appear in the list\. If it does not appear, search for it, select it, and then choose **Save**:  
![\[Your Group's IAM Role webpage with the Greengrasss_DynamoDB_Role and Save button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-094.png)