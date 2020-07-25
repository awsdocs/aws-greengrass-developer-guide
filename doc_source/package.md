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
![\[Screenshot showing zipped contents of hello_word_counter_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-046.png)

      On UNIX\-like systems \(including the Mac terminal\) that have `zip` installed, you can use the following command to package the file and folder:

      ```
      zip -r hello_world_counter_python_lambda.zip greengrasssdk greengrassHelloWorldCounter.py
      ```

   Now you're ready to create your Lambda function and upload the deployment package\.

1. In the Lambda console, choose **Create function**\.

1. Choose **Author from scratch**\. Name your function **Greengrass\_HelloWorld\_Counter**, and set the remaining fields as follows:
   + For **Runtime**, choose **Python 3\.7**\.
   + For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\. Or, you can reuse the role that you created in Module 3\-1\.

   Choose **Create function**\.

1. Upload your Lambda function deployment package:

   1. On the **Configuration** tab, under **Function code**, set the following fields:
      + For **Code entry type**, choose **Upload a \.zip file**\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **greengrassHelloWorldCounter\.function\_handler**

   1. Choose **Upload**, and then choose `hello_world_counter_python_lambda.zip`\.  
![\[Function code screenshot with Code entry type set to Upload a .zip file, Runtime set to Python 3.7, Handler set to greengrassHelloWorldCounter.function_handler, and Function package set to hello_world_counter_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-047.png)

   1. At the top of the page, choose **Save**\.
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.

1. Publish the first version of the function:

   1. From **Actions**, choose **Publish new version**\. For **Version description**, enter **First version**\.

   1. Choose **Publish**\.

1. Create an alias for the function version:

   1. From the **Actions** menu, choose **Create alias**, and set the following values:
      + For **Name**, enter **GG\_HW\_Counter**\.
      + For **Version**, choose **1**\.

   1. Choose **Create**\.  
![\[Create a new alias screenshot with the Name field set to GG_HW_Counter, the Version field set to 1, and the Create button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-048.png)

   Aliases create a single entity for your Lambda function that Greengrass devices can subscribe to\. This way, you don't have to update subscriptions with new Lambda function version numbers every time the function is modified\.