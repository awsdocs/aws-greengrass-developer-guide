# Create and Package the Lambda Function<a name="package"></a>

1. Download the Lambda function code to your computer \(not the Greengrass core device\), as follows:

   1. In a web browser, open the [greengrassHelloWorldCounter\.py](https://github.com/aws-samples/aws-greengrass-samples/blob/master/hello-world-counter-python/greengrassHelloWorldCounter.py) file on GitHub\.

   1. Choose **Raw** to open the unformatted version of the file\.  
![\[GitHub controls with the Raw button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-045.1.png)

   1. Use Ctrl \+ S \(or Command \+ S for the Mac\) to save a copy of the `greengrassHelloWorldCounter.py` file\. Save the file to the folder that contains the `greengrasssdk`, `greengrass_common`, and `greengrass_ipc_python_sdk` SDK folders\.
**Note**  
For UNIX\-like systems, you can run the following Terminal command to download the `greengrassHelloWorldCounter.py` file:  

   ```
   sudo wget https://raw.githubusercontent.com/aws-samples/aws-greengrass-samples/master/hello-world-counter-python/greengrassHelloWorldCounter.py
   ```

1. Package the `greengrassHelloWorldCounter.py` file with the three SDK folders into a `.zip` file, as described in [Module 3 \(Part 1\)](module3-I.md)\. Name the package **hello\_world\_counter\_python\_lambda\.zip**\.  
![\[Screenshot showing zipped contents of hello_word_counter_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-046.png)

1. In the Lambda console, create a Python 2\.7 function named **Greengrass\_HelloWorld\_Counter**, as described in [Module 3 \(Part 1\)](module3-I.md)\. You can use the existing role\.

1. Upload your Lambda function deployment package, as follows:

   1. On the **Configuration** tab, under **Function code**, set the following fields:
      + **Code entry type** \- choose **Upload a \.ZIP file**\.
      + **Runtime** \- choose **Python 2\.7**\.
      + **Handler** \- type **greengrassHelloWorldCounter\.function\_handler**\.

   1. Choose **Upload**, and then choose `hello_world_counter_python_lambda.zip`\.  
![\[Function code screenshot with Code entry type set to Upload a .ZIP file, Runtime set to Python 2.7, Handler set to greengrassHelloWorldCounter.function_handler, and Function package set to hello_world_counter_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-047.png)

   1. At the top of the page, choose **Save**\.

1. Publish the first version of the function, as follows:

   1. From the **Actions** menu, choose **Publish new version**\. For **Version description**, type **First version**\.

   1. Choose **Publish**\.

1. Create an alias for the function version, as follows:

   1. From the **Actions** menu, choose **Create alias**, and set the following values:
      + **Name** \- type **GG\_HW\_Counter**\.
      + **Version** \- choose **1**\.

   1. Choose **Create**\.  
![\[Create a new alias screenshot with the Name field set to GG_HW_Counter, the Version field set to 1, and the Create button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-048.png)

   Recall that aliases create a single entity for your Lambda function that AWS Greengrass devices can subscribe to without having to update subscriptions with Lambda version numbers every time the function is modified\.