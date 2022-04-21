--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Create and package the Lambda function<a name="package"></a>

In this step, you:
+ Create a Lambda function deployment package that contains the function code and dependencies\.
+ Use the Lambda console to create a Lambda function and upload the deployment package\.
+ Publish a version of the Lambda function and create an alias that points to the version\.

 

1. On your computer, go to the AWS IoT Greengrass Core SDK for Python that you downloaded and extracted in [Create and package a Lambda function](create-lambda.md) in Module 3\-1\.

   The Lambda function in this module uses:
   + The `greengrassHelloWorldCounter.py` file in `examples\HelloWorldCounter`\. This is your Lambda function code\.
   + The `greengrasssdk` folder\. This is the SDK\.

1. Create a Lambda function deployment package:

   1. Copy the `greengrasssdk` folder into the `HelloWorldCounter` folder that contains `greengrassHelloWorldCounter.py`\.

   1. Save `greengrassHelloWorldCounter.py` and the `greengrasssdk` folder to a `zip` file named `hello_world_counter_python_lambda.zip`\. The `py` file and `greengrasssdk` folder must be in the root of the directory\.  
![\[Screenshot showing zipped contents of hello_word_counter_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-046.png)

      On UNIX\-like systems \(including the Mac terminal\) that have `zip` installed, you can use the following command to package the file and folder:

      ```
      zip -r hello_world_counter_python_lambda.zip greengrasssdk greengrassHelloWorldCounter.py
      ```

   Now you're ready to create your Lambda function and upload the deployment package\.

1. Open the Lambda console and choose **Create function**\.

1. Choose **Author from scratch**\.

1. Name your function **Greengrass\_HelloWorld\_Counter**, and set the remaining fields as follows:
   + For **Runtime**, choose **Python 3\.7**\.
   + For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\. Or, you can reuse the role that you created in Module 3\-1\.

   Choose **Create function**\.  
![\[The "Basic information" section with the "Function name" field set to "Greengrass_HelloWorld_Counter" and the "Runtime" field set to "Python 3.7".\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-023-3.png)

1. Upload your Lambda function deployment package\.

   1. <a name="lambda-console-upload"></a>On the **Code** tab, under **Code source**, choose **Upload from**\. From the dropdown, choose **\.zip file**\.  
![\[The Upload from dropdown with .zip file highlighted.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/lra-console/upload-deployment-package.png)

   1. Choose **Upload**, and then choose your `hello_world_counter_python_lambda.zip` deployment package\. Then, choose **Save**\. 

   1. <a name="lambda-console-runtime-settings-para"></a>On the **Code** tab for the function, under **Runtime settings**, choose **Edit**, and then enter the following values\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **greengrassHelloWorldCounter\.function\_handler**

   1. <a name="lambda-console-save-config"></a>Choose **Save**\.
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.

1. Publish the first version of the function\.

   1. From the **Actions** menu at the top of the page, choose **Publish new version**\. For **Version description**, enter **First version**\.

   1. Choose **Publish**\.

1. Create an alias for the function version\.

   1. From the **Actions** menu at the top of the page, choose **Create alias**\.  
![\[Screenshot of the Actions menu set to Create alias.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-028.png)

   1. For **Name**, enter **GG\_HW\_Counter**\.

   1. For **Version**, choose **1**\.

   1. Choose **Save**\.  
![\[Create alias screenshot with the Name field set to GG_HW_Counter and the Version field set to 1.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-048.png)

   Aliases create a single entity for your Lambda function that Greengrass devices can subscribe to\. This way, you don't have to update subscriptions with new Lambda function version numbers every time the function is modified\.