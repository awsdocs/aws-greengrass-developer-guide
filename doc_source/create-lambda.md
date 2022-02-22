--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

--------

# Create and package a Lambda function<a name="create-lambda"></a>

The example Python Lambda function in this module uses the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) for Python to publish MQTT messages\.

In this step, you:
+ Download the AWS IoT Greengrass Core SDK for Python to your computer \(not the AWS IoT Greengrass core device\)\.
+ Create a Lambda function deployment package that contains the function code and dependencies\.
+ Use the Lambda console to create a Lambda function and upload the deployment package\.
+ Publish a version of the Lambda function and create an alias that points to the version\.

To complete this module, Python 3\.7 must be installed on your core device\.

 <a name="create-lambda-procedure"></a>

1. <a name="download-ggc-sdk"></a> From the [AWS IoT Greengrass Core SDK](what-is-gg.md#gg-core-sdk-download) downloads page, download the AWS IoT Greengrass Core SDK for Python to your computer\.

1. Unzip the downloaded package to get the Lambda function code and the SDK\.

   The Lambda function in this module uses:
   + The `greengrassHelloWorld.py` file in `examples\HelloWorld`\. This is your Lambda function code\. Every five seconds, the function publishes one of two possible messages to the `hello/world` topic\.
   + The `greengrasssdk` folder\. This is the SDK\.

1. Copy the `greengrasssdk` folder into the `HelloWorld` folder that contains `greengrassHelloWorld.py`\.

1. To create the Lambda function deployment package, save `greengrassHelloWorld.py` and the `greengrasssdk` folder to a compressed `zip` file named `hello_world_python_lambda.zip`\. The `py` file and `greengrasssdk` folder must be in the root of the directory\.  
![\[Screenshot showing zipped contents of hello_word_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-017.png)

   On UNIX\-like systems \(including the Mac terminal\), you can use the following command to package the file and folder:

   ```
   zip -r hello_world_python_lambda.zip greengrasssdk greengrassHelloWorld.py
   ```
**Note**  
Depending on your distribution, you might need to install `zip` first \(for example, by running `sudo apt-get install zip`\)\. The installation command for your distribution might be different\.

   Now you're ready to create your Lambda function and upload the deployment package\.

1. Open the Lambda console and choose **Create function**\.

1. Choose **Author from scratch**\.

1. Name your function **Greengrass\_HelloWorld**, and set the remaining fields as follows:
   + For **Runtime**, choose **Python 3\.7**\.
   + For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.

   Choose **Create function**\.  
![\[The "Basic information" section with the "Function name" field set to "Greengrass_HelloWorld" and the "Runtime" field set to "Python 3.7".\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-023.png)

1. Upload your Lambda function deployment package:

   1. <a name="lambda-console-upload"></a>On the **Code** tab, under **Code source**, choose **Upload from**\. From the dropdown, choose **\.zip file**\.  
![\[The Upload from dropdown with .zip file highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/lra-console/upload-deployment-package.png)

   1. Choose **Upload**, and then choose your `hello_world_python_lambda.zip` deployment package\. Then, choose **Save**\.

   1. <a name="lambda-console-runtime-settings-para"></a>On the **Code** tab for the function, under **Runtime settings**, choose **Edit**, and then enter the following values\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **greengrassHelloWorld\.function\_handler**  
![\[The "Runtime settings" section with the "Runtime" field set to "Python 3.7" and the "Handler" field set to "greengrassHelloWorld.function_handler".\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-023-2.png)

   1. <a name="lambda-console-save-config"></a>Choose **Save**\.
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.

1. <a name="publish-function-version"></a>Publish the Lambda function:

   1. From the **Actions** menu at the top of the page, choose **Publish new version**\.  
![\[Screenshot of the Actions menu with Publish new version highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-026.png)

   1. For **Version description**, enter **First version**, and then choose **Publish**\.  
![\[Screenshot with the Version description field set to First version and the Publish button highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-027.png)

1. <a name="create-version-alias"></a>Create an [alias](https://docs.aws.amazon.com/lambda/latest/dg/configuration-aliases.html) for the Lambda function [version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html):
**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

   1. From the **Actions** menu at the top of the page, choose **Create alias**\.  
![\[Screenshot of the Actions menu set to Create alias.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-028.png)

   1. Name the alias **GG\_HelloWorld**, set the version to **1** \(which corresponds to the version that you just published\), and then choose **Save**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

         
![\[Screenshot of Create a new alias with the Name field set to GG_HelloWorld, and the Version field set to 1.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-029.png)