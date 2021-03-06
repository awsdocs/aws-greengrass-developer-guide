--------

You are viewing the documentation for AWS IoT Greengrass Version 1\. AWS IoT Greengrass Version 2 is the latest major version of AWS IoT Greengrass\. For more information about using AWS IoT Greengrass Version 2, see the [https://docs.aws.amazon.com/greengrass/v2/developerguide](https://docs.aws.amazon.com/greengrass/v2/developerguide)\.

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

1. Upload your Lambda function deployment package:

   1.  On the **Configuration** tab, under **Function code**, choose **Actions**\. From the dropdown, choose **Upload a \.zip file**\. Then, choose `hello_world_counter_python_lambda.zip`\. 

   1.  Under **Runtime settings**, choose **Edit**\. On the **Edit runtime settings** page, set the remaining fields as follows: 
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **greengrassHelloWorldCounter\.function\_handler**

   1. Choose **Save**\.
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.

1. Publish the first version of the function:

   1. From the **Actions** menu at the top of the page, choose **Publish new version**\. For **Version description**, enter **First version**\.

   1. Choose **Publish**\.

1. Create an alias for the function version:

   1. From the **Actions** menu at the top of the page, choose **Create alias**, and set the following values:
      + For **Name**, enter **GG\_HW\_Counter**\.
      + For **Version**, choose **1**\.

   1. Choose **Save**\.  
![\[Create alias screenshot with the Name field set to GG_HW_Counter and the Version field set to 1.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-048.png)

   Aliases create a single entity for your Lambda function that Greengrass devices can subscribe to\. This way, you don't have to update subscriptions with new Lambda function version numbers every time the function is modified\.