# How to configure local resource access using the AWS Management Console<a name="lra-console"></a>

This feature is available for AWS IoT Greengrass Core v1\.3 and later\.

You can configure Lambda functions to securely access local resources on the host Greengrass core device\. *Local resources* refer to buses and peripherals that are physically present on the host, or file system volumes on the host OS\. For more information, including requirements and constraints, see [Access local resources with Lambda functions and connectors](access-local-resources.md)\.

This tutorial describes how to use the AWS Management Console to configure access to local resources that are present on an AWS IoT Greengrass core device\. It contains the following high\-level steps:

1. [Create a Lambda function deployment package](#lra-console-create-package)

1. [Create and publish a Lambda function](#lra-console-create-function)

1. [Add the Lambda function to the group](#lra-console-add-function)

1. [Add a local resource to the group](#lra-console-create-resource)

1. [Add subscriptions to the group](#lra-console-add-subscription)

1. [Deploy the group](#lra-console-deploy-group)

For a tutorial that uses the AWS Command Line Interface, see [How to configure local resource access using the AWS command line interface](lra-cli.md)\.

## Prerequisites<a name="lra-console-prerequisites"></a>

To complete this tutorial, you need:
+ A Greengrass group and a Greengrass core \(v1\.3 or later\)\. To create a Greengrass group or core, see [Getting started with AWS IoT Greengrass](gg-gs.md)\.
+ The following directories on the Greengrass core device:
  + /src/LRAtest
  + /dest/LRAtest

  The owner group of these directories must have read and write access to the directories\. You might use the following command to grant access:

  ```
  sudo chmod 0775 /src/LRAtest
  ```

## Step 1: Create a Lambda function deployment package<a name="lra-console-create-package"></a>

In this step, you create a Lambda function deployment package, which is a ZIP file that contains the function's code and dependencies\. You also download the AWS IoT Greengrass Core SDK to include in the package as a dependency\.

1. On your computer, copy the following Python script to a local file named `lraTest.py`\. This is the app logic for the Lambda function\.

   ```
   # Demonstrates a simple use case of local resource access.
   # This Lambda function writes a file test to a volume mounted inside
   # the Lambda environment under destLRAtest. Then it reads the file and 
   # publishes the content to the AWS IoT LRAtest topic. 
   
   import sys
   import greengrasssdk
   import platform
   import os
   import logging
   
   # Setup logging to stdout
   logger = logging.getLogger(__name__)
   logging.basicConfig(stream=sys.stdout, level=logging.DEBUG)
   
   # Create a Greengrass Core SDK client.
   client = greengrasssdk.client('iot-data')
   volumePath = '/dest/LRAtest'
   
   def function_handler(event, context):
       try:
           client.publish(topic='LRA/test', payload='Sent from AWS IoT Greengrass Core.')
           volumeInfo = os.stat(volumePath)
           client.publish(topic='LRA/test', payload=str(volumeInfo))
           with open(volumePath + '/test', 'a') as output:
               output.write('Successfully write to a file.')
           with open(volumePath + '/test', 'r') as myfile:
               data = myfile.read()
           client.publish(topic='LRA/test', payload=data)
       except Exception as e:
           logger.error('Failed to publish message: ' + repr(e))
       return
   ```

1. <a name="download-ggc-sdk"></a> From the [AWS IoT Greengrass Core SDK](what-is-gg.md#gg-core-sdk-download) downloads page, download the AWS IoT Greengrass Core SDK for Python to your computer\.

1. <a name="unzip-ggc-sdk"></a>Unzip the downloaded package to get the SDK\. The SDK is the `greengrasssdk` folder\.

1. Zip the following items into a file named `lraTestLambda.zip`:
   + `lraTest.py`\. App logic\.
   + `greengrasssdk`\. Required library for all Python Lambda functions\.

   The `lraTestLambda.zip` file is your Lambda function deployment package\. Now you're ready to create a Lambda function and upload the deployment package\.

## Step 2: Create and publish a Lambda function<a name="lra-console-create-function"></a>

In this step, you use the AWS Lambda console to create a Lambda function and configure it to use your deployment package\. Then, you publish a function version and create an alias\.

First, create the Lambda function\.

1. In the AWS Management Console, choose **Services**, and open the AWS Lambda console\.

1. Choose **Create function**\.

1. Choose **Author from scratch**\.

1. In the **Basic information** section, use the following values\.

   1. For **Function name**, enter **TestLRA**\.

   1. For **Runtime**, choose **Python 3\.7**\.

   1. For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.

1. Choose **Create function**\.  
![\[The Create function page with Create function highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-function.png)

    

   Now, upload your Lambda function deployment package and register the handler\.

1. On the **Configuration** tab for the function, in **Function code**, use the following values\.

   1. For **Code entry type**, choose **Upload a \.zip file**\.

   1. For **Runtime**, choose **Python 3\.7**\.

   1. For **Handler**, enter **lraTest\.function\_handler**\.

1. Choose **Upload**\.  
![\[The Function code section with Upload highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/upload-deployment-package.png)

1. Choose your `lraTestLambda.zip` deployment package\.

1. At the top of the page, choose **Save**\.  
![\[The TestLRA function page with Save highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/save-function.png)
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.  
You can see your code in the **Function code** section by choosing **Edit code inline** from the **Code entry type** menu\.

   Next, publish the first version of your Lambda function\. Then, create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.

   Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

1. From **Actions**, choose **Publish new version**\.  
![\[The Publish new version option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/publish-function-option.png)

1. For **Version description**, enter **First version**, and then choose **Publish**\.

1. On the **TestLRA: 1** configuration page, from **Actions**, choose **Create alias**\.  
![\[The Create alias option in the Actions menu.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-alias-option.png)

1. On the **Create a new alias** page, for **Name**, enter **test**\. For **Version**, enter **1**\. 
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

1. Choose **Create**\.  
![\[The Create a new alias page with Create highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-alias.png)

   You can now add the Lambda function to your Greengrass group\.

## Step 3: Add the Lambda function to the Greengrass group<a name="lra-console-add-function"></a>

In this step, you add the function to your group and configure the function's lifecycle\.

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

1. On the **Group\-specific Lambda configuration** page, for **Timeout**, choose **30 seconds**\.
**Important**  
Lambda functions that use local resources \(as described in this procedure\) must run in a Greengrass container\. Otherwise, deployment fails if you try to deploy the function\. For more information, see [Containerization](lambda-group-config.md#lambda-function-containerization)\.

1. At the bottom of the page, choose **Update**\.

## Step 4: Add a local resource to the Greengrass group<a name="lra-console-create-resource"></a>

In this step, you add a local volume resource to the Greengrass group and grant the function read and write access to the resource\. A local resource has a group\-level scope\. You can grant permissions for any Lambda function in the group to access the resource\.

1. On the group configuration page, choose **Resources**\.  
![\[The group configuration page with Resources highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-resources.png)

1. On the **Local** tab, choose **Add a local resource**\.

1. On the **Create a local resource** page, use the following values\.

   1. For **Resource name**, enter **testDirectory**\.

   1. For **Resource type**, choose **Volume**\.

   1. For **Source path**, enter **/src/LRAtest**\. This path must exist on the host OS\. 

      The source path is the local absolute path of the resource on the file system of the core device\. This location is outside of the [container](lambda-group-config.md#lambda-function-containerization) that the function runs in\. The path can't start with `/sys`\.

   1. For **Destination path**, enter **/dest/LRAtest**\. This path must exist on the host OS\.

      The destination path is the absolute path of the resource in the Lambda namespace\. This location is inside the container that the function runs in\.

   1. Under **Group owner file access permission**, select **Automatically add OS group permissions of the Linux group that owns the resource**\.

      The **Group owner file access permission** option lets you grant additional file access permissions to the Lambda process\. For more information, see [Group owner file access permission](access-local-resources.md#lra-group-owner)\.  
![\[The Create a local resource page with Save highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-local-resource.png)

1. Under **Lambda function affiliations**, choose **Select**\.

1. Choose **TestLRA**, choose **Read and write access**, and then choose **Done**\.  
![\[Lambda function affiliation properties with Done highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/create-local-resource-lambda.png)

1. At the bottom of the page, choose **Save**\. The **Resources** page displays the new testDirectory resource\.

## Step 5: Add subscriptions to the Greengrass group<a name="lra-console-add-subscription"></a>

In this step, you add two subscriptions to the Greengrass group\. These subscriptions enable bidirectional communication between the Lambda function and AWS IoT\.

First, create a subscription for the Lambda function to send messages to AWS IoT\.

1. On the group configuration page, choose **Subscriptions**, and then choose **Add Subscription**\.  
![\[The group page with Subscriptions and Add Subscription highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-subscriptions.png)

1. On the **Select your source and target** page, configure the source and target, as follows:

   1. For **Select a source**, choose **Lambdas**, and then choose **TestLRA**\.

   1. For **Select a target**, choose **Services**, and then choose **IoT Cloud**\.

   1. Choose **Next**\.  
![\[The Select your source and target page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/inbound-subscription.png)

1. On the **Filter your data with a topic** page, for **Topic filter**, enter **LRA/test**, and then choose **Next**\.  
![\[The Filter your data with a topic page with LRA/test and Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/inbound-subscription-filter.png)

1. Choose **Finish**\. The **Subscriptions** page displays the new subscription\.

    

   Next, configure a subscription that invokes the function from AWS IoT\.

1. On the **Subscriptions** page, choose **Add Subscription**\.

1. On the **Select your source and target** page, configure the source and target, as follows:

   1. For **Select a source**, choose **Services**, and then choose **IoT Cloud**\.

   1. For **Select a target**, choose **Lambdas**, and then choose **TestLRA**\.

   1. Choose **Next**\.  
![\[The Select your source and target page with Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/outbound-subscription.png)

1. On the **Filter your data with a topic** page, for **Topic filter**, enter **invoke/LRAFunction**, and then choose **Next**\.  
![\[The Filter your data with a topic page with invoke/LRAFunction and Next highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/outbound-subscription-filter.png)

1. Choose **Finish**\. The **Subscriptions** page displays both subscriptions\.

## Step 6: Deploy the AWS IoT Greengrass group<a name="lra-console-deploy-group"></a>

In this step, you deploy the current version of the group definition\.

1. Make sure that the AWS IoT Greengrass core is running\. Run the following commands in your Raspberry Pi terminal, as needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/1.11.0/bin/daemon`, then the daemon is running\.
**Note**  
The version in the path depends on the AWS IoT Greengrass Core software version that's installed on your core device\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

1. On the group configuration page, choose **Deployments**, and from **Actions**, choose **Deploy**\.  
![\[The group page with Deployments and Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-deployments-deploy.png)
**Note**  
Deployment fails if you run your Lambda function without containerization and try to access attached local resources\.

1. If prompted, on the **Configure how devices discover your core** page, choose **Automatic detection**\.

   This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS IoT Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.  
![\[The Configure how devices discover your core page with Automatic detection highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-discovery.png)
**Note**  
If prompted, grant permission to create the [Greengrass service role](service-role.md) and associate it with your AWS account in the current AWS Region\. This role allows AWS IoT Greengrass to access your resources in AWS services\.

   The **Deployments** page shows the deployment timestamp, version ID, and status\. When completed, the deployment status is **Successfully completed**\.

   For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## Test local resource access<a name="lra-console-test-results"></a>

Now you can verify whether the local resource access is configured correctly\. To test, you subscribe to the `LRA/test` topic and publish to the `invoke/LRAFunction` topic\. The test is successful if the Lambda function sends the expected payload to AWS IoT\.

1. On the AWS IoT console home page, in the left pane, choose **Test**\.  
![\[The left pane in the AWS IoT console with Test highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-test.png)

1. In the **Subscriptions** section, for **Subscription topic**, enter **LRA/test**\. For **MQTT payload display**, select **Display payloads as strings**\. 

1. Choose **Subscribe to topic**\. Your Lambda function publishes to the LRA/test topic\.  
![\[The Subscriptions page with Subscribe to topic highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/test-subscribe.png)

1. In the **Publish** section, enter **invoke/LRAFunction**, and then choose **Publish to topic** to invoke your Lambda function\. The test is successful if the page displays the function's three message payloads\.  
![\[The Subscriptions page with the invoke/LRAFunction topic and Publish to topic highlighted, and test results with message data.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/lra-console/test-publish.png)

The test file created by the Lambda function is in the `/src/LRAtest` directory on the Greengrass core device\. Although the Lambda function writes to a file in the `/dest/LRAtest` directory, that file is visible in the Lambda namespace only\. You can't see it in a regular Linux namespace\. Any changes to the destination path are reflected in the source path on the file system\.

For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.