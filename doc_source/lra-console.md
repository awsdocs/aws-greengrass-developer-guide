# How to Configure Local Resource Access Using the AWS Management Console<a name="lra-console"></a>

This feature is available for AWS IoT Greengrass Core v1\.3\.0 and later\.

You can configure Lambda functions to securely access local resources on the host Greengrass core device\. *Local resources* refer to buses and peripherals that are physically on the host, or file system volumes on the host OS\. For more information, including requirements and constraints, see [Access Local Resources with Lambda Functions](access-local-resources.md)\.

This tutorial describes how to use the AWS Management Console to configure access to local resources that are present on an AWS IoT Greengrass core device\. It contains the following high\-level steps:

1. [Create a Lambda Function Deployment Package](#lra-console-create-package)

1. [Create and Publish a Lambda Function](#lra-console-create-function)

1. [Add the Lambda Function to the Group](#lra-console-add-function)

1. [Add a Local Resource to the Group](#lra-console-create-resource)

1. [Add Subscriptions to the Group](#lra-console-add-subscription)

1. [Deploy the Group](#lra-console-deploy-group)

For a tutorial that uses the AWS Command Line Interface \(CLI\), see [How to Configure Local Resource Access Using the AWS Command Line Interface](lra-cli.md)\.

## Prerequisites<a name="lra-console-prerequisites"></a>

To complete this tutorial, you need:
+ A Greengrass group and a Greengrass core \(v1\.3\.0 or later\)\. To learn how to create a Greengrass group or core, see [Getting Started with AWS IoT Greengrass](gg-gs.md)\.
+ The following directories created on the Greengrass core device:
  + /src/LRAtest
  + /dest/LRAtest

  The owner group of these directories must have read and write access to the directories\. For example, you might use the following command to grant access:

  ```
  sudo chmod 0775 /src/LRAtest
  ```

## Step 1: Create a Lambda Function Deployment Package<a name="lra-console-create-package"></a>

In this step, you create a Lambda function deployment package, which is a ZIP file that contains the function's code and dependencies\. You also download the AWS IoT Greengrass Core SDK to include in the package as a dependency\.

1. On your computer, copy the following Python script to a local file named `lraTest.py`\. This is the app logic for the Lambda function\.

   ```
   # lraTest.py
   # Demonstrates a simple use case of local resource access.
   # This Lambda function writes a file "test" to a volume mounted inside
   # the Lambda environment under "/dest/LRAtest". Then it reads the file and 
   # publishes the content to the AWS IoT "LRA/test" topic. 
   
   import sys
   import greengrasssdk
   import platform
   import os
   import logging
   
   # Create a Greengrass Core SDK client.
   client = greengrasssdk.client('iot-data')
   volumePath = '/dest/LRAtest'
   
   def function_handler(event, context):
       client.publish(topic='LRA/test', payload='Sent from AWS Greengrass Core.')
       try:
           volumeInfo = os.stat(volumePath)
           client.publish(topic='LRA/test', payload=str(volumeInfo))
           with open(volumePath + '/test', 'a') as output:
               output.write('Successfully write to a file.\n')
           with open(volumePath + '/test', 'r') as myfile:
               data = myfile.read()
           client.publish(topic='LRA/test', payload=data)
       except Exception as e:
           logging.error("Experiencing error :{}".format(e))
       return
   ```

1. Download the AWS IoT Greengrass Core SDK Python 2\.7 version 1\.3\.0, as follows:

   1. In the [AWS IoT console](https://console.aws.amazon.com//iotv2/home), in the left pane, choose **Software**\.
**Note**  
You can download the SDK from the **Software** page in the console or from the [AWS IoT Greengrass Core SDK](what-is-gg.md#gg-core-sdk-download) downloads\. This procedure uses the console\.  
![\[The left pane of the AWS IoT console with Software highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-software.png)

   1. Under **SDKs**, for **AWS IoT Greengrass Core SDK**, choose **Configure download**\.  
![\[The SDKs section with Configure download highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-software-ggc-sdk.png)

   1. Choose **Python 2\.7 version 1\.3\.0**, and then choose **Download Greengrass Core SDK**\.  
![\[The AWS IoT Greengrass Core SDK page with Python 2.7 version 1.3.0 and Download Greengrass Core SDK highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-016.png)

1. Unpack the `greengrass-core-python-sdk-1.3.0.tar.gz` file\.
**Note**  
For ways that you can do this on different platforms, see [this step](create-lambda.md) in the Getting Started section\. For example, you might use the following `tar` command:  

   ```
   tar -xzf greengrass-core-python-sdk-1.3.0.tar.gz
   ```

1. Open the extracted aws\_greengrass\_core\_sdk/sdk folder, and unzip `python_sdk_1_3_0.zip`\.

1. Zip the following items into a file named `lraTestLambda.zip`:
   + **lraTest\.py**\. App logic\.
   + **greengrasssdk**\. Required library for all Python Lambda functions\.
   + **Greengrass AWS SW License \(IoT additiona\) vr6\.txt**\. Required Greengrass Core Software License Agreement\.

   The `lraTestLambda.zip` file is your Lambda function deployment package\. Now you're ready to create a Lambda function and upload the deployment package\.

## Step 2: Create and Publish a Lambda Function<a name="lra-console-create-function"></a>

In this step, you use the AWS Lambda console to create a Lambda function and configure it to use your deployment package\. Then, you publish a function version and create an alias\.

First, create the Lambda function\.

1. In the AWS Management Console, choose **Services**, and open the AWS Lambda console\.

1. Choose **Create function**\.

1. Choose **Author from scratch**\.

1. In the **Author from scratch** section, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lra-console.html)

1. At the bottom of the page, choose **Create function**\.  
![\[The Create function page with Create function highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-function.png)

 

Now, upload your Lambda function deployment package and register the handler\.

1. On the **Configuration** tab for the TestLRA function, in **Function code**, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lra-console.html)

1. Choose **Upload**\.  
![\[The Function code section with Upload highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/upload-deployment-package.png)

1. Choose your `lraTestLambda.zip` deployment package\.

1. At the top of the page, choose **Save**\.  
![\[The TestLRA function page with Save highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/save-function.png)
**Tip**  
You can see your code in the **Function code** section by choosing **Edit code inline** from the **Code entry type** menu\.

 

Next, publish the first version of your Lambda function\. Then, create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.

**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

1. From the **Actions** menu, choose **Publish new version**\.  
![\[The Publish new version option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/publish-function-option.png)

1. For **Version description**, type **First version**, and then choose **Publish**\.

1. On the **TestLRA: 1** configuration page, from the **Actions** menu, choose **Create alias**\.  
![\[The Create alias option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-alias-option.png)

1. On the **Create a new alias** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lra-console.html)
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

1. Choose **Create**\.  
![\[The Create a new alias page with Create highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-alias.png)

   You can now add the Lambda function to your Greengrass group\.

## Step 3: Add the Lambda Function to the Greengrass Group<a name="lra-console-add-function"></a>

In this step, you add the TestLRA function to your group and configure the function's lifecycle\.

First, add the Lambda function to your Greengrass group\.

1. In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.  
![\[The left pane in the AWS IoT console with Groups highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-groups.png)

1. Choose the Greengrass group where you want to add the Lambda function\.

1. On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group configuration page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1. On the **Add a Lambda to your Greengrass Group** page, choose **Use existing Lambda**\.  
![\[The Add a Lambda to your Greengrass Group page with Use existing Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-existing-lambda.png)

1. On the **Use existing Lambda** page, choose **TestLRA**, and then choose **Next**\.

1. On the **Select a Lambda version** page, choose **Alias:test**, and then choose **Finish**\.

 

Next, configure the lifecycle of the Lambda function\.

1. On the **Lambdas** page, choose the TestLRA Lambda function\.   
![\[The Lambdas page with the TestLRA Lambda function highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/new-lambda.png)

1. On the **TestLRA** configuration page, choose **Edit**\.

1. On the **Group\-specific Lambda configuration** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lra-console.html)

   For more information, see [Lifecycle Configuration for Greengrass Lambda Functions](lambda-functions.md#lambda-lifecycle)\.  
![\[The TestLRA page with updated properties.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/config-lambda.png)
**Important**  
Because this procedure uses local resources, you must run the Lambda function in a AWS IoT Greengrass container\. Deployment will fail if you try to deploy a Lambda function that uses local resources as described here\.

1. At the bottom of the page, choose **Update**\.

## Step 4: Add a Local Resource to the Greengrass Group<a name="lra-console-create-resource"></a>

In this step, you add a local volume resource to a Greengrass group and grant the function read and write access to the resource\. A local resource has a group\-level scope, which makes it accessible by all Lambda functions in the group\.

1. On the group configuration page, choose **Resources**\.  
![\[The group configuration page with Resources highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-resources.png)

1. On the **Local Resources** tab, choose **Add local resource**\.

1. On the **Create a local resource** page, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lra-console.html)

   The **Source path** is the local absolute path of the resource on the file system of the core device\. This path can't start with /sys\.

   The **Destination path** is the absolute path of the resource in the Lambda namespace\.

   The **Group owner file access permission** option lets you grant additional file access permissions to the Lambda process\. For more information, see [Group Owner File Access Permission](access-local-resources.md#lra-group-owner)\.  
![\[The Create a local resource page with Save highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-local-resource.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **TestLRA**, choose **Read and write access**, and then choose **Done**\.  
![\[Lambda function affiliation properties with Done highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-local-resource-lambda.png)

1. At the bottom of the page, choose **Save**\. The **Resources** page displays the new testDirectory resource\.

## Step 5: Add Subscriptions to the Greengrass Group<a name="lra-console-add-subscription"></a>

In this step, you add two subscriptions to the Greengrass group\. These subscriptions enable bidirectional communication between the Lambda function and AWS IoT\.

First, create a subscription for the Lambda function to send messages to AWS IoT Greengrass\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.  
![\[The group page with Subscriptions and Add Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-subscriptions.png)

1. On the **Select your source and target** page, configure the source and target, as follows:

   1. For **Select a source**, choose **Lambdas**, and then choose **TestLRA**\.

   1. For **Select a target**, choose **Services**, and then choose **IoT Cloud**\.

   1. Choose **Next**\.  
![\[The Select your source and target page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/inbound-subscription.png)

1. On the **Filter your data with a topic** page, for **Optional topic filter**, type **LRA/test**, and then choose **Next**\.  
![\[The Filter your data with a topic page with LRA/test and Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/inbound-subscription-filter.png)

1. Choose **Finish**\. The **Subscriptions** page displays the new subscription\.

 

Next, configure a subscription that invokes the function from AWS IoT\.

1. On the **Subscriptions** page, choose **Add Subscription**\.

1. On the **Select your source and target** page, configure the source and target, as follows:

   1. For **Select a source**, choose **Services**, and then choose **IoT Cloud**\.

   1. For **Select a target**, choose **Lambdas**, and then choose **TestLRA**\.

   1. Choose **Next**\.  
![\[The Select your source and target page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/outbound-subscription.png)

1. On the **Filter your data with a topic** page, for **Optional topic filter**, type **invoke/LRAFunction**, and then choose **Next**\.  
![\[The Filter your data with a topic page with invoke/LRAFunction and Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/outbound-subscription-filter.png)

1. Choose **Finish**\. The **Subscriptions** page displays both subscriptions\.

## Step 6: Deploy the AWS IoT Greengrass Group<a name="lra-console-deploy-group"></a>

In this step, you deploy the current version of the group definition\.

1. Make sure that the AWS IoT Greengrass core is running\. Run the following commands in your Raspberry Pi terminal, as needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/1.7.0/bin/daemon`, then the daemon is running\.
**Note**  
The version in the path depends on the AWS IoT Greengrass Core software version that's installed on your core device\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

1. On the group configuration page, choose **Deployments**, and from the **Actions** menu, choose **Deploy**\.  
![\[The group page with Deployments and Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-deployments-deploy.png)
**Note**  
Deployment will fail if you run your Lambda function without containerization and try to access attached local resources\.

1. On the **Configure how devices discover your core** page, choose **Automatic detection**\.

   This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS IoT Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.  
![\[The Configure how devices discover your core page with Automatic detection highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-discovery.png)
**Note**  
If prompted, grant permission to create the AWS IoT Greengrass service role on your behalf, which allows AWS IoT Greengrass to access other AWS services\. You need to do this only one time per account\.

   The **Deployments** page shows the deployment timestamp, version ID, and status\. When completed, the deployment should show a **Successfully completed** status\.

   For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## Test Local Resource Access<a name="lra-console-test-results"></a>

Now you can verify whether the local resource access is configured correctly\. To test, you subscribe to the **LRA/test** topic and publish to the **invoke/LRAFunction** topic\. The test is successful if the Lambda function sends the expected payload to AWS IoT\.

1. On the AWS IoT console home page, in the left pane, choose **Test**\.  
![\[The left pane in the AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-test.png)

1. In the **Subscriptions** section, use the following values:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lra-console.html)

1. Choose **Subscribe to topic**\. Your Lambda function publishes to the LRA/test topic\.  
![\[The Subscriptions page with Subscribe to topic highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/test-subscribe.png)

1. In the **Publish** section, type **invoke/LRAFunction**, and then choose **Publish to topic** to invoke your Lambda function\. The test is successful if the page displays the function's three message payloads\.  
![\[The Subscriptions page with the invoke/LRAFunction topic and Publish to topic highlighted, and test results with message data.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/test-publish.png)

You can see the test file that the Lambda creates by looking in the /src/LRAtest directory on the Greengrass core device\. Although the Lambda writes to a file in the /dest/LRAtest directory, that file is visible in the Lambda namespace only—you cant' see it in a regular Linux namespace\. However, any changes to the destination path are reflected in the source path on the actual file system\.

For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.