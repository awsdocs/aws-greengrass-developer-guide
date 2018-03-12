# Create and Package a Lambda Function<a name="create-lambda"></a>

In order for a Python Lambda function to run on an AWS Greengrass core, it must be packaged with specific folders from the Python AWS Greengrass Core SDK\. In the following, you will:

+ Download the Python AWS Greengrass Core SDK to your computer \(not the AWS Greengrass core device\)\.

+ Decompress the downloaded SDK file\.

+ Obtain the Python Lambda function \(named `greengrassHelloWorld.py`\) from the decompressed SDK\.

+ Create a Lambda function deployment package named `hello_world_python_lambda.zip` that contains `greengrassHelloWorld.py` and three required SDK folders\.

+ Upload the `hello_world_python_lambda.zip` package by using the Lambda console\. 

+ Transfer the package to the AWS Greengrass core device by using the AWS Greengrass console\.

1. In the AWS IoT console, choose **Software**\.  
![\[The left navigation pane of the AWS IoT console page with Software highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-014.png)

   To get the AWS Greengrass Core SDK, on the home page, scroll down to **SDKs**, and choose **Configure download**\.  
![\[The SDKs section with Configure download highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-015.png)

   From the drop\-down list, choose **Python 2\.7 version 1\.0\.0**, and then choose **Download Greengrass Core SDK**\.  
![\[Python 2.7 version 1.0.0 and Download Greengrass Core SDK.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-016.png)

1. Based on your operating system, choose a tab to decompress the downloaded SDK\.

------
#### [ Windows ]

   Use a tool for decompressing `.tar.gz` files on Windows such as [7\-Zip](http://www.7-zip.org/), [WinZip](http://www.winzip.com/), or similar\. As an example, the 7\-Zip tool can be used to decompress `greengrass-core-python-sdk-1.0.0.tar.gz` as follows:

   1. After installing 7\-Zip, navigate to the `greengrass-core-python-sdk-1.0.0.tar.gz` file using Windows File Explorer \(Windows logo key \+ E\), right\-click the file, choose **7\-Zip**, then choose **Open archive**\.

   1. In the resulting 7\-Zip window, double\-click `greegrass-core-python-sdk-1.0.0.tar`, `aws_greengrass_core_sdk`, `examples`, `HelloWorld`, and then `greengrassHelloWorld.zip`\.

   1. Optionally using the Ctrl key, select the three SDK folders `greengrasssdk`, `greengrass_common`, `greengrass_ipc_python_sdk` and the Python `greengrassHelloWorld.py` Lambda file\. Next, choose **Extract**, pick a location to extract the files to, and choose **OK**\.

------
#### [ macOS ]

   1. Using Finder, navigate to the `greengrass-core-python-sdk-1.0.0.tar.gz` file and double\-click it\. This creates the `aws_greengrass_core_sdk` folder\.

   1. Expand the `aws_greengrass_core_sdk` folder, then the `examples` folder, and then the `HelloWorld` folder\.

   1. Double\-click the `greengrassHelloWorld.zip` file\. This creates the `greengrassHelloWorld` folder – expand this folder\.

   1. Note the three SDK subfolders `greengrasssdk`, `greengrass_common`, `greengrass_ipc_python_sdk` and the Python `greengrassHelloWorld.py` Lambda script file\. 

------
#### [ UNIX\-like system ]

   1. Open a terminal window and navigate to the directory containing the `greengrass-core-python-sdk-1.0.0.tar.gz` file\.

   1. Run the following command to decompress the file:

      ```
      sudo tar -xzf greengrass-core-python-sdk-1.0.0.tar.gz
      ```

      This creates the `aws_greengrass_core_sdk` directory\. Next, run the following commands:

      ```
      cd /aws_greengrass_core_sdk/examples/HelloWorld
      sudo unzip greengrassHelloWorld.zip
      ```

      This creates the three SDK folders `greengrass_common`, `greengrass_ipc_python_sdk`, `greengrasssdk` and the Python `greengrassHelloWorld.py` Lambda file\.

------

   Note that the `greengrassHelloWorld.py` Python Lambda function publishes one of two possible messages every 5 seconds to the `hello/world` topic, as shown next \(to save space, all code comments have been removed\):

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

1. In order to run the Python `greengrassHelloWorld.py` Lambda function in the cloud, you must package it with the AWS Greengrass core SDK\.\. Therefore, after you have extracted the SDK folders `greengrass_common`, `greengrass_ipc_python_sdk`, `greengrasssdk` and the `greengrassHelloWorld.py` Python Lambda file, package them into a compressed `.zip` file named `hello_world_python_lambda.zip`:   
![\[Screenshot showing greengrass_common, greengrass_ipc_python_sdk, greengrasssdk, and greengrassHelloWorld.py getting compressed into the file hello_world_python_lambda.zip.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-019.png)

   For UNIX\-like systems \(including the Mac terminal\), this can be accomplished with the following command:

   ```
   zip -r hello_world_python_lambda.zip greengrass_common/greengrass_ipc_python_sdk/greengrasssdk/greengrassHelloWorld.py
   ```
**Note**  
Depending on your distribution, you may need to install `zip` first\. For example, `sudo apt-get install zip` \(this installation command may differ for your distribution\)\.

   Now you're ready to create your Lambda function and upload the function code\.

1. Open the Lambda console and choose **Create function**\.

1. Choose **Author from scratch**\.

1. Name your function **Greengrass\_HelloWorld**, and set the remaining fields as follows:

   + **Runtime** \- choose **Python 2\.7**\.

   + **Role** \- choose **Create new role from templates\(s\)**\.

   + **Role name** \- type a unique name for the role\.

     Note that this role isn't used by AWS Greengrass, so you can optionally use any existing role\.

   Then, choose **Create function**\.  
![\[The "Author from scratch" region with the "Name" field set to "Greengrass_HelloWorld", the "Runtime" field set to "Python 2.7", the "Role" field set to "Create new role from templates(s)".\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-023.png)

1. Upload your Lambda function deployment package, as follows:

   1. On the **Configuration** tab, under **Function code**, set the following fields:

      + **Code entry type** \- choose **Upload a \.ZIP file**\.

      + **Runtime** \- choose **Python 2\.7**\.

      + **Handler** \- type **greengrassHelloWorld\.function\_handler**\.

   1. Choose **Upload**, and then choose `hello_world_python_lambda.zip`\. Your `hello_world_python_lambda.zip` file size may vary\.  
![\[Screenshot of the Configuration tab with Upload a .ZIP file, Python 2.7, greengrassHelloWorld.function_handler, and Upload highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-024.png)

   1. Choose **Save**\.  
![\[Screenshot with the Save button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-025.png)
**Tip**  
To see your uploaded code, choose **Edit code inline** from the **Code entry type** menu\.

1. <a name="publish-function-version"></a>Publish the Lambda function, as follows:

   1. From the **Actions** menu, choose **Publish new version**:  
![\[Screenshot of the Actions menu with Publish new version highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-026.png)

   1. For **Version description**, type **First version**, and then choose **Publish**:  
![\[Screenshot with the Version description field set to First version and the Publish button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-027.png)

   Next, create an alias for the Lambda function version\. Aliases create a single entity for your Lambda function that AWS Greengrass devices can subscribe to without having to update subscriptions with Lambda version numbers every time a function is modified\. If future versions of this Lambda are published, you must point the alias to the new version\.
**Note**  
AWS Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

1. <a name="create-version-alias"></a>Create an alias, as follows:

   1. From the **Actions** menu, choose **Create alias**\.  
![\[Screenshot of the Actions menu set to Create alias.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-028.png)

   1. Name the alias **GG\_HelloWorld**, set the version to **1** \(which corresponds to the version that you just published\), and then choose **Create**\.  
![\[Screenshot of Create a new alias with the Name field set to GG_HelloWorld, the Version field set to 1, and the Create button highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-029.png)