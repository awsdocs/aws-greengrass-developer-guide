# Create bulk deployments for groups<a name="bulk-deploy-cli"></a>

 You can use simple API calls to deploy large numbers of Greengrass groups at once\. These deployments are triggered with an adaptive rate that has a fixed upper limit\. 

 This tutorial describes how to use the AWS CLI to create and monitor a bulk group deployment in AWS IoT Greengrass\. The bulk deployment example in this tutorial contains multiple groups\. You can use the example in your implementation to add as many groups as you need\. 

 The tutorial contains the following high\-level steps: 

1. [Create and upload the bulk deployment input file](#bulk-deploy-cli-create-input-file)

1. [Create and configure an IAM execution role](#bulk-deploy-cli-create-role)

1. [Allow your execution role access to your S3 Bucket](#bulk-deploy-cli-modify-bucket)

1. [Deploy the groups](#bulk-deploy-cli-start-bulk-deployments)

1. [Test the deployment](#bulk-deploy-cli-test)

## Prerequisites<a name="bulk-deploy-cli-prerequisites"></a>

 To complete this tutorial, you need: 
+  One or more deployable Greengrass groups\. For more information about creating AWS IoT Greengrass groups and cores, see [Getting started with AWS IoT Greengrass](gg-gs.md)\. 
+  The AWS CLI installed and configured on your machine\. For information, see the [ AWS CLI User Guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)\. 
+ An S3 bucket created in the same AWS Region as AWS IoT Greengrass\. For information, see [ Creating and configuring an S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/create-configure-bucket.html) in the *Amazon Simple Storage Service Console User Guide*\. 
**Note**  
 Currently, SSE KMS enabled buckets are not supported\. 

## Step 1: Create and upload the bulk deployment input file<a name="bulk-deploy-cli-create-input-file"></a>

 In this step, you create a deployment input file and upload it to your Amazon S3 bucket\. This file is a serialized, line\-delimited JSON file that contains information about each group in your bulk deployment\. AWS IoT Greengrass uses this information to deploy each group on your behalf when you initialize your bulk group deployment\. 

1.  Run the following command to get the `groupId` for each group you want to deploy\. You enter the `groupId` into your bulk deployment input file so that AWS IoT Greengrass can identify each group to be deployed\. 
**Note**  
<a name="find-group-ids-console"></a>You can also find these values in the AWS IoT console\. The group ID is displayed on the group's **Settings** page\. Group version IDs are displayed on the group's **Deployments** page\.

   ```
   aws greengrass list-groups
   ```

    The response contains information about each group in your AWS IoT Greengrass account: 

   ```
   {
     "Groups": [
       {
         "Name": "string",
         "Id": "string",
         "Arn": "string",
         "LastUpdatedTimestamp": "string",
         "CreationTimestamp": "string",
         "LatestVersion": "string",
         "LatestVersionArn": "string"
       }
     ],
     "NextToken": "string"
   }
   ```

    Run the following command to get the `groupVersionId` of each group you want to deploy\. 

   ```
   list-group-versions --group-id groupId
   ```

    The response contains information about all of the versions in the group\. Make a note of the ID of the group version you want to use\. 

   ```
   {
     "Versions": [
       {
         "Arn": "string",
         "Id": "string",
         "Version": "string",
         "CreationTimestamp": "string"
       }
     ],
     "NextToken": "string"
   }
   ```

1.  In your computer terminal or editor of choice, create a file, *MyBulkDeploymentInputFile*, from the following example\. This file contains information about each AWS IoT Greengrass group to be included in a bulk deployment\. Although this example defines multiple groups, for this tutorial, your file can contain just one\. 
**Note**  
 The size of this file must be less than 100 MB\. 

   ```
   {"GroupId":"groupId1", "GroupVersionId":"groupVersionId1", "DeploymentType":"NewDeployment"}
   {"GroupId":"groupId2", "GroupVersionId":"groupVersionId2", "DeploymentType":"NewDeployment"}
   {"GroupId":"groupId3", "GroupVersionId":"groupVersionId3", "DeploymentType":"NewDeployment"}
   ...
   ```

    Each record \(or line\) contains a group object\. Each group object contains its corresponding `groupId` and `groupVersionId` and a `DeploymentType`\. Currently, AWS IoT Greengrass supports `NewDeployment` bulk deployment types only\. 

    Save and close your file\. Make a note of the location of the file\. 

1.  Use the following command in your terminal to upload your input file to your Amazon S3 bucket\. Replace the file path with the location and name of your file\. For information, see [Add an object to a bucket](https://docs.aws.amazon.com/AmazonS3/latest/gsg/PuttingAnObjectInABucket.html)\. 

   ```
   aws s3 cp path/MyBulkDeploymentInputFile s3://my-bucket/
   ```

## Step 2: Create and configure an IAM execution role<a name="bulk-deploy-cli-create-role"></a>

 In this step, you use the IAM console to create a standalone execution role\. You then establish a trust relationship between the role and AWS IoT Greengrass and ensure that your IAM user has `PassRole` privileges for your execution role\. This allows AWS IoT Greengrass to assume your execution role and create the deployments on your behalf\. 

1.  Use the following policy to create an execution role\. This policy document allows AWS IoT Greengrass to access your bulk deployment input file when it creates each deployment on your behalf\. 

    For more information about creating an IAM role and delegating permissions, see [Creating IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)\. 

   ```
    {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "VisualEditor0",
               "Effect": "Allow",
               "Action": "greengrass:CreateDeployment",
               "Resource": [
                   "arn:aws:greengrass:region:accountId:/greengrass/groups/groupId1",
                   "arn:aws:greengrass:region:accountId:/greengrass/groups/groupId2",
                   "arn:aws:greengrass:region:accountId:/greengrass/groups/groupId3",
                   ...
               ]
           }
       ]
   }
   ```
**Note**  
 This policy must have a resource for each group or group version in your bulk deployment input file to be deployed by AWS IoT Greengrass\. To allow access to all groups, for `Resource`, specify an asterisk:   

   ```
   "Resource": ["*"]
   ```

1.  Modify the trust relationship for your execution role to include AWS IoT Greengrass\. This allows AWS IoT Greengrass to use your execution role and the permissions attached to it\. For information, see [Editing the trust relationship for an existing role](https://docs.aws.amazon.com/directoryservice/latest/admin-guide/edit_trust.html)\. 

   ```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "",
         "Effect": "Allow",
         "Principal": {
           "Service": "greengrass.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

1.  Give IAM `PassRole` permissions for your execution role to your IAM user\. This IAM user is the one used to initiate the bulk deployment\. `PassRole` permissions allow your IAM user to pass your execution role to AWS IoT Greengrass for use\. For more information, see [Granting a user permissions to pass a role to an AWS service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_passrole.html)\. 

    Use the following example to update your trust policy document\. Modify this example, as necessary\. 

   ```
    {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "Stmt1508193814000",
               "Effect": "Allow",
               "Action": [
                   "iam:PassRole"
               ],
               "Resource": [
                   "arn:aws:iam::123456789012:user/executionRoleArn"
               ]
           }
       ]
   }
   ```

## Step 3: Allow your execution role access to your S3 Bucket<a name="bulk-deploy-cli-modify-bucket"></a>

 To start your bulk deployment, your execution role must be able to read your bulk deployment input file from your Amazon S3 bucket\. Attach the following example policy to your Amazon S3 bucket so its `GetObject` permissions are accessible to your execution role\. 

 For more information, see [ How do I add an S3 bucket policy?](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/add-bucket-policy.html) 

```
{
    "Version": "2008-10-17",
    "Id": "examplePolicy",
    "Statement": [
        {
            "Sid": "Stmt1535408982966",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "executionRoleArn"
                ]
            },
            "Action": "s3:GetObject",
            "Resource":
            "arn:aws:s3:::my-bucket/objectKey"
        }
    ]
}
```

 You can use the following command in your terminal to check your bucket's policy: 

```
aws s3api get-bucket-policy --bucket my-bucket
```

**Note**  
 You can directly modify your execution role to grant it permission to your Amazon S3 bucket's `GetObject` permissions instead\. To do this, attach the following example policy to your execution role\.   

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-bucket/objectKey"
        }
    ]
}
```

## Step 4: Deploy the groups<a name="bulk-deploy-cli-start-bulk-deployments"></a>

 In this step, you start a bulk deployment operation for all group versions configured in your bulk deployment input file\. The deployment action for each of your group versions is of type `NewDeploymentType`\. 

**Note**  
 You cannot call StartBulkDeployment while another bulk deployment from the same account is still running\. The request is rejected\. 

1.  Use the following command to start the bulk deployment\. 

    We recommend that you include an `X-Amzn-Client-Token` token in every StartBulkDeployment request\. These requests are idempotent with respect to the token and the request parameters\. This token can be any unique, case\-sensitive string of up to 64 ASCII characters\. 

   ```
   aws greengrass start-bulk-deployment --cli-input-json "{
             "InputFileUri":"URI of file in S3 bucket", 
             "ExecutionRoleArn":"ARN of execution role",
             "AmznClientToken":"your Amazon client token"
             }"
   ```

    The command should result in a successful status code of `200`, along with the following response: 

   ```
   {
     "bulkDeploymentId": UUID
   }
   ```

    Make a note of the bulk deployment ID\. It can be used to check the status of your bulk deployment\. 
**Note**  
Although bulk deployment operations are not currently supported, you can create Amazon EventBridge event rules to get notifications about deployment status changes for individual groups\. For more information, see [Get deployment notifications](deployment-notifications.md)\.

1.  Use the following command to check the status of your bulk deployment\. 

   ```
   aws greengrass get-bulk-deployment-status --bulk-deployment-id 1234567
   ```

    The command should return a successful status code of `200` in addition to a JSON payload of information: 

   ```
    {
     "BulkDeploymentStatus": Running,
     "Statistics": {
        "RecordsProcessed": integer,
        "InvalidInputRecords": integer,
        "RetryAttempts": integer
     },
     "CreatedAt": "string",
     "ErrorMessage": "string",
     "ErrorDetails": [
       {
         "DetailedErrorCode": "string",
         "DetailedErrorMessage": "string"
       }
     ]
   }
   ```

    `BulkDeploymentStatus` contains the current status of the bulk execution\. The execution can have one of six different statuses: 
   + `Initializing`\. The bulk deployment request has been received, and the execution is preparing to start\.
   + `Running`\. The bulk deployment execution has started\.
   + `Completed`\. The bulk deployment execution has finished processing all records\.
   + `Stopping`\. The bulk deployment execution has received a command to stop and will terminate shortly\. You can't start a new bulk deployment while a previous deployment is in the `Stopping` state\.
   + `Stopped`\. The bulk deployment execution has been manually stopped\.
   + `Failed`\. The bulk deployment execution has encountered an error and terminated\. You can find error details in the `ErrorDetails` field\.

    The JSON payload also includes statistical information about the progress of the bulk deployment\. You can use this information to determine how many groups have been processed and how many have failed\. The statistical information includes: 
   +  `RecordsProcessed`: The number of group records that were attempted\. 
   +  `InvalidInputRecords`: The total number of records that returned a non\-retryable error\. For example, this can occur if a group record from the input file uses an invalid format or specifies a nonexistent group version, or if the execution doesn't grant permission to deploy a group or group version\. 
   +  `RetryAttempts`: The number of deployment attempts that returned a retryable error\. For example, a retry is triggered if the attempt to deploy a group returns a throttling error\. A group deployment can be retried up to five times\. 

    In the case of a bulk deployment execution failure, this payload also includes an `ErrorDetails` section that can be used for troubleshooting\. It contains information about the cause of the execution failure\. 

    You can periodically check the status of the bulk deployment to confirm that it is progressing as expected\. After the deployment is complete, `RecordsProcessed` should be equal to the number of deployment groups in your bulk deployment input file\. This indicates that each record has been processed\. 

## Step 5: Test the deployment<a name="bulk-deploy-cli-test"></a>

 Use the ListBulkDeployments command to find the ID of your bulk deployment\. 

```
aws greengrass list-bulk-deployments
```

 This command returns a list of all of your bulk deployments from most to least recent, including your `BulkDeploymentId`\. 

```
{
  "BulkDeployments": [
    {
      "BulkDeploymentId": 1234567,
      "BulkDeploymentArn": "string",
      "CreatedAt": "string"
    }
  ],
  "NextToken": "string"
}
```

 Now call the ListBulkDeploymentDetailedReports command to gather detailed information about each deployment\. 

```
aws greengrass list-bulk-deployment-detailed-reports --bulk-deployment-id 1234567 
```

 The command should return a successful status code of `200` along with a JSON payload of information: 

```
{ 
  "BulkDeploymentResults": [
    {
      "DeploymentId": "string",
      "GroupVersionedArn": "string",
      "CreatedAt": "string",
      "DeploymentStatus": "string",
      "ErrorMessage": "string",
      "ErrorDetails": [
        {
          "DetailedErrorCode": "string",
          "DetailedErrorMessage": "string"
        }
      ]
    }
  ],
  "NextToken": "string"
}
```

 This payload usually contains a paginated list of each deployment and its deployment status from most to least recent\. It also contains more information in the event of a bulk deployment execution failure\. Again, the total number of deployments listed should be equal to the number of groups you identified in your bulk deployment input file\. 

 The information returned can change until the deployments are in a terminal state \(success or failure\)\. You can call this command periodically until then\. 

## Troubleshooting bulk deployments<a name="bulk-deploy-cli-troubleshooting"></a>

 If the bulk deployment is not successful, you can try the following troubleshooting steps\. Run the commands in your terminal\. 

### Troubleshoot input file errors<a name="w64aac15c28c23b5"></a>

 The bulk deployment can fail in the event of syntax errors in the bulk deployment input file\. This returns a bulk deployment status of `Failed` with an error message indicating the line number of the first validation error\. There are four possible errors: 
+ 

  ```
  InvalidInputFile: Missing GroupId at line number: line number
  ```

   This error indicates that the given input file line is unable to register the specified parameter\. The possible missing parameters are the `GroupId` and the `GroupVersionId`\. 
+ 

  ```
  InvalidInputFile: Invalid deployment type at line number : line number. Only valid type is 'NewDeployment'.
  ```

   This error indicates that the given input file line lists an invalid deployment type\. At this time, the only supported deployment type is a `NewDeployment`\. 
+ 

  ```
  Line %s is too long in S3 File. Valid line is less than 256 chars.
  ```

   This error indicates that the given input file line is too long and must be shortened\. 
+ 

  ```
  Failed to parse input file at line number: line number
  ```

   This error indicates that the given input file line is not considered valid json\. 

### Check for concurrent bulk deployments<a name="w64aac15c28c23b7"></a>

 You cannot start a new bulk deployment while another one is still running or in a non\-terminal state\. This can result in a `Concurrent Deployment Error`\. You can use the ListBulkDeployments command to verify that a bulk deployment is not currently running\. This command lists your bulk deployments from most to least recent\. 

```
{
  "BulkDeployments": [
    {
      "BulkDeploymentId": BulkDeploymentId,
      "BulkDeploymentArn": "string",
      "CreatedAt": "string"
    }
  ],
  "NextToken": "string"
}
```

 Use the `BulkDeploymentId` of the first listed bulk deployment to run the GetBulkDeploymentStatus command\. If your most recent bulk deployment is in a running state \(`Initializing` or `Running`\), use the following command to stop the bulk deployment\. 

```
aws greengrass stop-bulk-deployment --bulk-deployment-id BulkDeploymentId
```

 This action results in a status of `Stopping` until the deployment is `Stopped`\. After the deployment has reached a `Stopped` status, you can start a new bulk deployment\. 

### Check ErrorDetails<a name="w64aac15c28c23b9"></a>

 Run the `GetBulkDeploymentStatus` command to return a JSON payload that contains information about any bulk deployment execution failure\. 

```
  "Message": "string",
  "ErrorDetails": [
    {
      "DetailedErrorCode": "string",
      "DetailedErrorMessage": "string"
    }
  ]
```

 When exiting with an error, the `ErrorDetails` JSON payload that is returned by this call contains more information about the bulk deployment execution failure\. An error status code in the `400` series, for example, indicates an input error, either in the input parameters or the caller dependencies\. 

### Check the AWS IoT Greengrass core log<a name="w64aac15c28c23c11"></a>

 You can troubleshoot issues by viewing the AWS IoT Greengrass core logs\. Use the following commands to view `runtime.log`: 

```
cd /greengrass/ggc/var/log
sudo cat system/runtime.log | more
```

For more information about AWS IoT Greengrass logging, see [Monitoring with AWS IoT Greengrass logs](greengrass-logs-overview.md)\. 

## See also<a name="bulk-deploy-cli-see-also"></a>

For more information, see the following resources:
+ [Deploy AWS IoT Greengrass groups to an AWS IoT Greengrass core](deployments.md)
+ [Amazon S3 API commands](https://docs.aws.amazon.com/cli/latest/reference/s3api) in the *AWS CLI Command Reference*
+ <a name="see-also-gg-cli"></a>[AWS IoT Greengrass commands](https://docs.aws.amazon.com/cli/latest/reference/greengrass/index.html) in the *AWS CLI Command Reference*