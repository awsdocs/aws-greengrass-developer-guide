# Create and Package the Lambda Function<a name="package"></a>

1. Download the Lambda function code to your computer \(not the Greengrass core device\):

   1. In a web browser, open the [greengrassHelloWorldCounter\.py](https://github.com/aws-samples/aws-greengrass-samples/blob/master/hello-world-counter-python/greengrassHelloWorldCounter.py) file on GitHub\.

   1. Choose **Raw** to open the unformatted version of the file\.  
![\[GitHub controls with the Raw button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-045.1.png)

   1. Use Ctrl \+ S \(or Command \+ S for the Mac\) to save a copy of the `greengrassHelloWorldCounter.py` file\. Save the file to a folder that contains the `greengrasssdk` folder\.
**Note**  
For UNIX\-like systems, you can run the following Terminal command to download the `greengrassHelloWorldCounter.py` file:  

   ```
   wget https://raw.githubusercontent.com/aws-samples/aws-greengrass-samples/master/hello-world-counter-python/greengrassHelloWorldCounter.py
   ```

1. Package the `greengrassHelloWorldCounter.py` file with the SDK into a `.zip` file, as described in [Module 3 \(Part 1\)](module3-I.md)\. Name the package **hello\_world\_counter\_python\_lambda\.zip**\.  
![\[Screenshot showing zipped contents of hello_word_counter_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-046.png)

1. In the Lambda console, create a Python 3\.7 function named **Greengrass\_HelloWorld\_Counter**, as described in [Module 3 \(Part 1\)](module3-I.md)\. You can use the existing role\.

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

   Aliases create a single entity for your Lambda function that AWS IoT Greengrass devices can subscribe to without having to update subscriptions with Lambda version numbers every time the function is modified\.