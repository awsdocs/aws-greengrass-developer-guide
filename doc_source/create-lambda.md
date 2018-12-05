# Create and Package a Lambda Function<a name="create-lambda"></a>

For a Python Lambda function to run on an AWS IoT Greengrass core, it must be packaged with specific folders from the Python AWS IoT Greengrass Core SDK\. In the this section, you:
+ Download the Python AWS IoT Greengrass Core SDK to your computer \(not the AWS IoT Greengrass core device\)\. You can download the SDK from the **Software** page in the AWS IoT console or from the [AWS IoT Greengrass Core SDK](what-is-gg.md#gg-core-sdk-download) downloads\. This procedure uses the console\.
+ Decompress the downloaded SDK file\.
+ Obtain the Python Lambda function \(named `greengrassHelloWorld.py`\) from the decompressed SDK\.
+ Create a Lambda function deployment package named `hello_world_python_lambda.zip` that contains `greengrassHelloWorld.py` and the `greengrasssdk` folder\.
+ Use the Lambda console to upload the `hello_world_python_lambda.zip` package\. 
+ Use the AWS IoT console to transfer the package to the AWS IoT Greengrass core device\.

1. In the AWS IoT console, choose **Software**\.  
![\[The navigation pane of the AWS IoT console page with Software highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-software.png)

1. Under **SDKs**, for **AWS IoT Greengrass Core SDK**, choose **Configure download**\.
**Note**  
You can download the SDK from the **Software** page in the console or from the [AWS IoT Greengrass Core SDK](what-is-gg.md#gg-core-sdk-download) downloads\. This procedure uses the console\.  
![\[The SDKs section with Configure download highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-software-ggc-sdk.png)

1. Choose **Python 2\.7 version 1\.3\.0**, and then choose **Download Greengrass Core SDK**\.  
![\[Python 2.7 version 1.3.0 and Download Greengrass Core SDK.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-016.png)

1. Decompress the downloaded SDK\. For instructions, choose the tab that corresponds to your operating system\.

------
#### [ Windows ]

   Use a tool for decompressing `.tar.gz` files on Windows such as [7\-Zip](http://www.7-zip.org/), [WinZip](http://www.winzip.com/), or similar\. As an example, the 7\-Zip tool can be used to decompress `greengrass-core-python-sdk-1.3.0.tar.gz` as follows:

   1. After you install 7\-Zip, use Windows File Explorer \(Windows logo key \+ E\) to navigate to `greengrass-core-python-sdk-1.3.0.tar.gz`, right\-click the file, choose **7\-Zip**, and then choose **Open archive**\.

   1. In the 7\-Zip window, double\-click `greegrass-core-python-sdk-1.3.0.tar`, `aws_greengrass_core_sdk`, `examples`, `HelloWorld`, and `greengrassHelloWorld.zip`\.

   1. \(Optional\) Using the Ctrl key, select the `greengrasssdk` folder and the Python `greengrassHelloWorld.py` Lambda file\. Choose **Extract**, choose a location to extract the files to, and then choose **OK**\.

------
#### [ macOS ]

   1. Using Finder, navigate to the `greengrass-core-python-sdk-1.3.0.tar.gz` file and double\-click it\. This creates the `aws_greengrass_core_sdk` folder\.

   1. Expand the `aws_greengrass_core_sdk` folder, the `examples` folder, and the `HelloWorld` folder\.

   1. Double\-click the `greengrassHelloWorld.zip` file\. This creates the `greengrassHelloWorld` folder\. Expand this folder\.

------
#### [ UNIX\-like system ]

   1. Open a terminal window and navigate to the directory that contains the `greengrass-core-python-sdk-1.3.0.tar.gz` file\.

   1. Run the following command to decompress the file:

      ```
      sudo tar -xzf greengrass-core-python-sdk-1.3.0.tar.gz
      ```

      This creates the `aws_greengrass_core_sdk` directory\. Next, run the following commands:

      ```
      cd /aws_greengrass_core_sdk/examples/HelloWorld
      sudo unzip greengrassHelloWorld.zip
      ```

------

   You use the `greengrasssdk` folder and the Python `greengrassHelloWorld.py` Lambda function code in the next step\.

   Note that every five seconds, the `greengrassHelloWorld.py` Lambda function publishes one of two possible messages to the `hello/world` topic, as shown in the following code\. \(To save space, all code comments have been removed\.\)

   ```
   import greengrasssdk
   import platform
   from threading import Timer
   import time
   
   client = greengrasssdk.client('iot-data')
   my_platform = platform.platform()
   
   def greengrass_hello_world_run():
       if not my_platform:
           client.publish(topic='hello/world', payload='Hello world! Sent from Greengrass Core.')
       else:
           client.publish(topic='hello/world', payload='Hello world! Sent from Greengrass Core running on platform: {}'.format(my_platform))
       Timer(5, greengrass_hello_world_run).start()
   
   greengrass_hello_world_run()
   
   def function_handler(event, context):
       return
   ```

1. To create the `greengrassHelloWorld.py` Lambda function, you must package the `greengrassHelloWorld.py` file and the `greengrasssdk` folder into a compressed `.zip` file\. This is your Lambda function deployment package\. For this tutorial, name the package `hello_world_python_lambda.zip`:   
![\[Screenshot showing greengrasssdk and greengrassHelloWorld.py compressed into the file hello_world_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-019.png)

   For UNIX\-like systems \(including the Mac terminal\), this can use the following command to package the file and folder:

   ```
   sudo zip -r hello_world_python_lambda.zip greengrasssdk greengrassHelloWorld.py
   ```
**Note**  
Depending on your distribution, you might need to install `zip` first \(for example, by running `sudo apt-get install zip`\)\. This installation command for your distribution might be different\.

   Now you're ready to create your Lambda function and upload the deployment package\.

1. Open the Lambda console and choose **Create function**\.

1. Choose **Author from scratch**\.

1. Name your function **Greengrass\_HelloWorld**, and set the remaining fields as follows:
   + For **Runtime**, choose **Python 2\.7**\.
   + For **Role**, choose **Create new role from one or more templates**\.
   + For **Role name**, enter a unique name for the role\. \(This role isn't used by AWS IoT Greengrass\.\)

   Choose **Create function**\.  
![\[The "Author from scratch" section with the "Name" field set to "Greengrass_HelloWorld", the "Runtime" field set to "Python 2.7", the "Role" field set to "Create new role from one or more templates".\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-023.png)

1. Upload your Lambda function deployment package, as follows:

   1. On the **Configuration** tab, under **Function code**, set the following fields:
      + For **Code entry type**, choose **Upload a \.zip file**\.
      + For **Runtime**, choose **Python 2\.7**\.
      + For **Handler**, enter **greengrassHelloWorld\.function\_handler**\.

   1. Choose **Upload**, and then choose `hello_world_python_lambda.zip`\. \(The size of your `hello_world_python_lambda.zip` file might be different from what's shown here\.\)  
![\[Screenshot of the Configuration tab with Upload a .zip file, Python 2.7, greengrassHelloWorld.function_handler, and Upload highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-024.png)

   1. Choose **Save**\.  
![\[Screenshot with the Save button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-025.png)
**Tip**  
To see your uploaded code, from **Code entry type**, choose **Edit code inline**\.

1. <a name="publish-function-version"></a>Publish the Lambda function, as follows:

   1. From **Actions**, choose **Publish new version**:  
![\[Screenshot of the Actions menu with Publish new version highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-026.png)

   1. For **Version description**, enter **First version**, and then choose **Publish**:  
![\[Screenshot with the Version description field set to First version and the Publish button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-027.png)

1. <a name="create-version-alias"></a>Create an [alias](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html) for the Lambda function version, as follows:
**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

   1. From **Actions**, choose **Create alias**\.  
![\[Screenshot of the Actions menu set to Create alias.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-028.png)

   1. Name the alias **GG\_HelloWorld**, set the version to **1** \(which corresponds to the version that you just published\), and then choose **Create**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

         
![\[Screenshot of Create a new alias with the Name field set to GG_HelloWorld, the Version field set to 1, and the Create button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-029.png)