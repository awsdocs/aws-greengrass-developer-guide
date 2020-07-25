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
![\[Screenshot showing zipped contents of hello_word_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-017.png)

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
![\[The "Basic information" section with the "Function name" field set to "Greengrass_HelloWorld" and the "Runtime" field set to "Python 3.7".\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-023.png)

1. Upload your Lambda function deployment package:

   1.  On the **Configuration** tab, under **Function code**, choose **Actions**\. From the dropdown, choose **Upload a \.zip file**\. Then, choose `hello_world_python_lambda.zip`\. 

   1.  Under **Basic settings**, choose **Edit**\. On the **Edit basic settings** page, set the remaining fields as follows: 
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **greengrassHelloWorld\.function\_handler**

   1. Choose **Save**\.
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.

1. <a name="publish-function-version"></a>Publish the Lambda function:

   1. From **Actions**, choose **Publish new version**\.

   1. For **Version description**, enter **First version**, and then choose **Publish**\.  
![\[Screenshot with the Version description field set to First version and the Publish button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-027.png)

1. <a name="create-version-alias"></a>Create an [alias](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html) for the Lambda function version:
**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

   1. From **Actions**, choose **Create alias**\.  
![\[Screenshot of the Actions menu set to Create alias.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-028.png)

   1. Name the alias **GG\_HelloWorld**, set the version to **1** \(which corresponds to the version that you just published\), and then choose **Create**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

         
![\[Screenshot of Create a new alias with the Name field set to GG_HelloWorld, the Version field set to 1, and the Create button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-029.png)