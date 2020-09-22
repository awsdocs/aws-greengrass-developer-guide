# Export data streams to the AWS Cloud \(console\)<a name="stream-manager-console"></a>

This tutorial shows you how to use the AWS IoT console to configure and deploy an AWS IoT Greengrass group with stream manager enabled\. The group contains a user\-defined Lambda function that writes to a stream in stream manager, which is then exported automatically to the AWS Cloud\.

## <a name="w64aac25c27b6"></a>

Stream manager makes ingesting, processing, and exporting high\-volume data streams easier and more reliable\. In this tutorial, you create a `TransferStream` Lambda function that consumes IoT data\. The Lambda function uses the AWS IoT Greengrass Core SDK to create a stream in stream manager and then read and write to it\. Stream manager then exports the stream to Kinesis Data Streams\. The following diagram shows this workflow\.

![\[Diagram of the stream management workflow.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/stream-manager-scenario.png)

The focus of this tutorial is to show how user\-defined Lambda functions use the `StreamManagerClient` object in the AWS IoT Greengrass Core SDK to interact with stream manager\. For simplicity, the Python Lambda function that you create for this tutorial generates simulated device data\.

## Prerequisites<a name="stream-manager-console-prerequisites"></a>

To complete this tutorial, you need:<a name="stream-manager-howto-prereqs"></a>
+ A Greengrass group and a Greengrass core \(v1\.10 or later\)\. To learn how to create a Greengrass group and core, see [Getting started with AWS IoT Greengrass](gg-gs.md)\. The Getting Started tutorial also includes steps for installing the AWS IoT Greengrass Core software\.
**Note**  <a name="stream-manager-not-supported-openwrt"></a>
Stream manager is not supported on OpenWrt distributions\.
+ The Java 8 runtime \(JDK 8\) installed on the core device\.<a name="install-java8-runtime-general"></a>
  + For Debian\-based distributions \(including Raspbian\) or Ubuntu\-based distributions, run the following command:

    ```
    sudo apt install openjdk-8-jdk
    ```
  + For Red Hat\-based distributions \(including Amazon Linux\), run the following command:

    ```
    sudo yum install java-1.8.0-openjdk
    ```

    For more information, see [ How to download and install prebuilt OpenJDK packages](https://openjdk.java.net/install/) in the OpenJDK documentation\.
+ AWS IoT Greengrass Core SDK for Python v1\.5\.0 or later\. To use `StreamManagerClient` in the AWS IoT Greengrass Core SDK for Python, you must:
  + Install Python 3\.7 or later on the core device\.
  + Include the SDK and its dependencies in your Lambda function deployment package\. Instructions are provided in this tutorial\.
**Tip**  
You can use `StreamManagerClient` with Java or NodeJS\. For example code, see the [ AWS IoT Greengrass Core SDK for Java](https://github.com/aws/aws-greengrass-core-sdk-java/blob/master/samples/StreamManagerKinesis/src/main/java/com/amazonaws/greengrass/examples/StreamManagerKinesis.java) and [AWS IoT Greengrass Core SDK for Node\.js](https://github.com/aws/aws-greengrass-core-sdk-js/blob/master/greengrassExamples/StreamManagerKinesis/index.js) on GitHub\.
+ A destination stream named **MyKinesisStream** created in Amazon Kinesis Data Streams in the same AWS Region as your Greengrass group\. For more information, see [Create a stream](https://docs.aws.amazon.com/streams/latest/dev/fundamental-stream.html#create-stream) in the *Amazon Kinesis Developer Guide*\.
**Note**  
In this tutorial, stream manager exports data to Kinesis Data Streams, which results in charges to your AWS account\. For information about pricing, see [Kinesis Data Streams pricing](https://aws.amazon.com/kinesis/data-streams/pricing/)\.  
To avoid incurring charges, you can run this tutorial without creating a Kinesis data stream\. In this case, you check the logs to see that stream manager attempted to export the stream to Kinesis Data Streams\.
+ An IAM policy added to the [Greengrass group role](group-role.md) that allows the `kinesis:PutRecords` action on the target data stream, as shown in the following example:

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "kinesis:PutRecords"
              ],
              "Resource": [
                  "arn:aws:kinesis:region:account-id:stream/MyKinesisStream"
              ]
          }
      ]
  }
  ```

The tutorial contains the following high\-level steps:

1. [Create a Lambda function deployment package](#stream-manager-console-create-deployment-package)

1. [Create a Lambda function](#stream-manager-console-create-function)

1. [Add a function to the group](#stream-manager-console-create-gg-function)

1. [Enable stream manager](#stream-manager-console-enable-stream-manager)

1. [Configure local logging](#stream-manager-console-configure-logging)

1. [Deploy the group](#stream-manager-console-create-deployment)

1. [Test the application](#stream-manager-console-test-application)

The tutorial should take about 20 minutes to complete\.

## Step 1: Create a Lambda function deployment package<a name="stream-manager-console-create-deployment-package"></a>

### <a name="w64aac25c27c17b4"></a>

In this step, you create a Lambda function deployment package that contains Python function code and dependencies\. You upload this package later when you create the Lambda function in AWS Lambda\. The Lambda function uses the AWS IoT Greengrass Core SDK to create and interact with local streams\.

**Note**  
 Your user\-defined Lambda functions must use the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) to interact with stream manager\. For more information about requirements for the Greengrass stream manager, see [Greengrass stream manager requirements](stream-manager.md#stream-manager-requirements)\. 

1.  Download the [AWS IoT Greengrass Core SDK for Python](lambda-functions.md#lambda-sdks-core) v1\.5\.0 or later\.

1. <a name="unzip-ggc-sdk"></a>Unzip the downloaded package to get the SDK\. The SDK is the `greengrasssdk` folder\.

1. <a name="install-python-sdk-dependencies-stream-manager"></a>Install package dependencies to include with the SDK in your Lambda function deployment package\.<a name="python-sdk-dependencies-stream-manager"></a>

   1. Navigate to the SDK directory that contains the `requirements.txt` file\. This file lists the dependencies\.

   1. Install the SDK dependencies\. For example, run the following `pip` command to install them in the current directory:

      ```
      pip install --target . -r requirements.txt
      ```

1. Save the following Python code function in a local file named `transfer_stream.py`\.
**Tip**  
 For example code that uses Java and NodeJS, see the [ AWS IoT Greengrass Core SDK for Java](https://github.com/aws/aws-greengrass-core-sdk-java/blob/master/samples/StreamManagerKinesis/src/main/java/com/amazonaws/greengrass/examples/StreamManagerKinesis.java) and [AWS IoT Greengrass Core SDK for Node\.js](https://github.com/aws/aws-greengrass-core-sdk-js/blob/master/greengrassExamples/StreamManagerKinesis/index.js) on GitHub\.

   ```
   import asyncio
   import logging
   import random
   import time
   
   from greengrasssdk.stream_manager import (
       ExportDefinition,
       KinesisConfig,
       MessageStreamDefinition,
       ReadMessagesOptions,
       ResourceNotFoundException,
       StrategyOnFull,
       StreamManagerClient,
   )
   
   
   # This example creates a local stream named "SomeStream".
   # It starts writing data into that stream and then stream manager automatically exports  
   # the data to a customer-created Kinesis data stream named "MyKinesisStream". 
   # This example runs forever until the program is stopped.
   
   # The size of the local stream on disk will not exceed the default (which is 256 MB).
   # Any data appended after the stream reaches the size limit continues to be appended, and
   # stream manager deletes the oldest data until the total stream size is back under 256 MB.
   # The Kinesis data stream in the cloud has no such bound, so all the data from this script is
   # uploaded to Kinesis and you will be charged for that usage.
   
   
   def main(logger):
       try:
           stream_name = "SomeStream"
           kinesis_stream_name = "MyKinesisStream"
   
           # Create a client for the StreamManager
           client = StreamManagerClient()
   
           # Try deleting the stream (if it exists) so that we have a fresh start
           try:
               client.delete_message_stream(stream_name=stream_name)
           except ResourceNotFoundException:
               pass
   
           exports = ExportDefinition(
               kinesis=[KinesisConfig(identifier="KinesisExport" + stream_name, kinesis_stream_name=kinesis_stream_name)]
           )
           client.create_message_stream(
               MessageStreamDefinition(
                   name=stream_name, strategy_on_full=StrategyOnFull.OverwriteOldestData, export_definition=exports
               )
           )
   
           # Append two messages and print their sequence numbers
           logger.info(
               "Successfully appended message to stream with sequence number %d",
               client.append_message(stream_name, "ABCDEFGHIJKLMNO".encode("utf-8")),
           )
           logger.info(
               "Successfully appended message to stream with sequence number %d",
               client.append_message(stream_name, "PQRSTUVWXYZ".encode("utf-8")),
           )
   
           # Try reading the two messages we just appended and print them out
           logger.info(
               "Successfully read 2 messages: %s",
               client.read_messages(stream_name, ReadMessagesOptions(min_message_count=2, read_timeout_millis=1000)),
           )
   
           logger.info("Now going to start writing random integers between 0 and 1000 to the stream")
           # Now start putting in random data between 0 and 1000 to emulate device sensor input
           while True:
               logger.debug("Appending new random integer to stream")
               client.append_message(stream_name, random.randint(0, 1000).to_bytes(length=4, signed=True, byteorder="big"))
               time.sleep(1)
   
       except asyncio.TimeoutError:
           logger.exception("Timed out while executing")
       except Exception:
           logger.exception("Exception while running")
   
   
   def function_handler(event, context):
       return
   
   
   logging.basicConfig(level=logging.INFO)
   # Start up this sample code
   main(logger=logging.getLogger())
   ```

1. Zip the following items into a file named `transfer_stream_python.zip`\. This is your Lambda function deployment package\.
   + **transfer\_stream\.py**\. App logic\.
   + **greengrasssdk**\. Required library for Python Greengrass Lambda functions that publish MQTT messages\.

     [Stream manager operations](work-with-streams.md) are available in version 1\.5\.0 or later of the AWS IoT Greengrass Core SDK for Python\.
   + The dependencies you installed for the AWS IoT Greengrass Core SDK for Python \(for example, the `cbor2` directories\)\.

   When you create the `zip` file, include only these items, not the containing folder\.

## Step 2: Create a Lambda function<a name="stream-manager-console-create-function"></a>

In this step, you use the AWS Lambda console to create a Lambda function and configure it to use your deployment package\. Then, you publish a function version and create an alias\.

1. First, create the Lambda function\.

   1. <a name="lambda-console-open"></a>In the AWS Management Console, choose **Services**, and open the AWS Lambda console\.

   1. <a name="lambda-console-create-function"></a>Choose **Create function** and then choose **Author from scratch**\.

   1. In the **Basic information** section, use the following values:
      + For **Function name**, enter **TransferStream**\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Permissions**, keep the default setting\. This creates an execution role that grants basic Lambda permissions\. This role isn't used by AWS IoT Greengrass\.

   1. <a name="lambda-console-save-function"></a>At the bottom of the page, choose **Create function**\.

1. Next, register the handler and upload your Lambda function deployment package\.

   1. On the **Configuration** tab for the `TransferStream` function, in **Function code**, use the following values:
      + For **Code entry type**, choose **Upload a \.zip file**\.
      + For **Runtime**, choose **Python 3\.7**\.
      + For **Handler**, enter **transfer\_stream\.function\_handler**

   1. <a name="lambda-console-upload"></a>Choose **Upload**\.

   1. Choose your `transfer_stream_python.zip` deployment package\.

   1. <a name="lambda-console-save-config"></a>Choose **Save**\.
**Note**  
The **Test** button on the AWS Lambda console doesn't work with this function\. The AWS IoT Greengrass Core SDK doesn't contain modules that are required to run your Greengrass Lambda functions independently in the AWS Lambda console\. These modules \(for example, `greengrass_common`\) are supplied to the functions after they are deployed to your Greengrass core\.

1. Now, publish the first version of your Lambda function and create an [alias for the version](https://docs.aws.amazon.com/lambda/latest/dg/versioning-aliases.html)\.
**Note**  
Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\.

   1. <a name="shared-publish-function-version"></a>From the **Actions** menu, choose **Publish new version**\.

   1. <a name="shared-publish-function-version-description"></a>For **Version description**, enter **First version**, and then choose **Publish**\.

   1. On the **TransferStream: 1** configuration page, from the **Actions** menu, choose **Create alias**\.

   1. On the **Create a new alias** page, use the following values:
      + For **Name**, enter **GG\_TransferStream**\.
      + For **Version**, choose **1**\.
**Note**  
AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\.

   1. Choose **Create**\.

Now you're ready to add the Lambda function to your Greengrass group\.

## Step 3: Add a Lambda function to the Greengrass group<a name="stream-manager-console-create-gg-function"></a>

In this step, you add the Lambda function to the group and then configure its lifecycle and environment variables\. For more information, see [Controlling execution of Greengrass Lambda functions by using group\-specific configuration](lambda-group-config.md)\.

1. <a name="console-gg-groups"></a>In the AWS IoT console, choose **Greengrass**, and then choose **Groups**\.

1. <a name="group-choose-target-group"></a>Choose the target group\.

1. <a name="choose-add-lambda"></a>On the group configuration page, choose **Lambdas**, and then choose **Add Lambda**\.  
![\[The group page with Lambdas and Add Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas.png)

1. <a name="add-lambda-to-group-console"></a>On the **Add a Lambda to your Greengrass Group** page, choose **Use existing Lambda**\.  
![\[The Add a Lambda to your Greengrass Group page with Use existing Lambda highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-lambdas-existing-lambda.png)

1. On the **Use existing Lambda** page, choose **TransferStream**, and then choose **Next**\.

1. On the **Select a Lambda version** page, choose **Alias:GG\_TransferStream**, and then choose **Finish**\.

   Now, configure properties that determine the behavior of the Lambda function in the Greengrass group\.

1. For the **TransferStream** Lambda function, choose the ellipsis \(**…**\), and then choose **Edit Configuration**\.

1. On the **Group\-specific Lambda configuration** page, make the following changes:
   + Set **Memory limit** to 32 MB\.
   + For **Lambda lifecycle**, choose **Make this function long\-lived and keep it running indefinitely**\.
**Note**  
<a name="long-lived-lambda"></a>A *long\-lived* \(or *pinned*\) Lambda function starts automatically after AWS IoT Greengrass starts and keeps running in its own container\. This is in contrast to an *on\-demand* Lambda function, which starts when invoked and stops when there are no tasks left to execute\. For more information, see [Lifecycle configuration for Greengrass Lambda functions](lambda-functions.md#lambda-lifecycle)\.

1. Choose **Update**\.

## Step 4: Enable stream manager<a name="stream-manager-console-enable-stream-manager"></a>

In this step, you make sure that stream manager is enabled\.

1. <a name="shared-group-settings"></a>On the group configuration page, choose **Settings**\.  
![\[Group settings page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-settings.png)

1. Under **Stream manager**, check the enabled or disabled status\. If disabled, choose **Edit**\. Then, choose **Enable** and **Save**\. You can use the default parameter settings for this tutorial\. For more information, see [Configure AWS IoT Greengrass stream manager](configure-stream-manager.md)\.  
![\[The Stream manager section on the group's Settings page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-settings-stream-manager-edit.png)

**Note**  
When you use the console to enable stream manager and deploy the group, the memory limit for stream manager is set to 4 GB\.

## Step 5: Configure local logging<a name="stream-manager-console-configure-logging"></a>

In this step, you configure AWS IoT Greengrass system components, user\-defined Lambda functions, and connectors in the group to write logs to the file system of the core device\. You can use logs to troubleshoot any issues you might encounter\. For more information, see [Monitoring with AWS IoT Greengrass logs](greengrass-logs-overview.md)\.

1. <a name="shared-group-settings-local-logs-configuration"></a>Under **Local logs configuration**, check if local logging is configured\.  
![\[Logs configuration section showing Greengrass system logs and user Lambda logs configuration.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/local-logs.png)

1. <a name="shared-group-settings-local-logs-edit"></a>If logs aren't configured for Greengrass system components or user\-defined Lambda functions, choose **Edit**\.

1. <a name="shared-group-settings-local-logs-event-source"></a>Choose **Add another log type**, choose **User Lambdas** and **Greengrass system**, and then choose **Update**\.

1. <a name="shared-group-settings-local-logs-save"></a>Keep the default values for logging level and disk space limit, and then choose **Save**\.

## Step 6: Deploy the Greengrass group<a name="stream-manager-console-create-deployment"></a>

Deploy the group to the core device\.

1. <a name="shared-deploy-group-checkggc"></a>Make sure that the AWS IoT Greengrass core is running\. Run the following commands in your Raspberry Pi terminal, as needed\.

   1. To check whether the daemon is running:

      ```
      ps aux | grep -E 'greengrass.*daemon'
      ```

      If the output contains a `root` entry for `/greengrass/ggc/packages/ggc-version/bin/daemon`, then the daemon is running\.
**Note**  
The version in the path depends on the AWS IoT Greengrass Core software version that's installed on your core device\.

   1. To start the daemon:

      ```
      cd /greengrass/ggc/core/
      sudo ./greengrassd start
      ```

1. <a name="shared-deploy-group-deploy"></a>On the group configuration page, choose **Deployments**, and from the **Actions** menu, choose **Deploy**\.  
![\[The group page with Deployments and Deploy highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-deployments-deploy.png)

1. <a name="shared-deploy-group-ipconfig"></a>If prompted, on the **Configure how devices discover your core** page, choose **Automatic detection**\.

   This enables devices to automatically acquire connectivity information for the core, such as IP address, DNS, and port number\. Automatic detection is recommended, but AWS IoT Greengrass also supports manually specified endpoints\. You're only prompted for the discovery method the first time that the group is deployed\.  
![\[The Configure how devices discover your core page with Automatic detection highlighted.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-discovery.png)
**Note**  
If prompted, grant permission to create the [Greengrass service role](service-role.md) and associate it with your AWS account in the current AWS Region\. This role allows AWS IoT Greengrass to access your resources in AWS services\.

   The **Deployments** page shows the deployment timestamp, version ID, and status\. When completed, the status displayed for the deployment should be **Successfully completed**\.

   For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## Step 7: Test the application<a name="stream-manager-console-test-application"></a>

The `TransferStream` Lambda function generates simulated device data\. It writes data to a stream that stream manager exports to the target Kinesis data stream\.

1. <a name="stream-manager-howto-test-open-kinesis-console"></a>In the Amazon Kinesis console, under **Kinesis data streams**, choose **MyKinesisStream**\.
**Note**  
If you ran the tutorial without a target Kinesis data stream, [check the log file](stream-manager-cli.md#stream-manager-cli-logs) for the stream manager \(`GGStreamManager`\)\. If it contains `export stream MyKinesisStream doesn't exist` in an error message, then the test is successful\. This error means that the service tried to export to the stream but the stream doesn't exist\.

1. <a name="stream-manager-howto-view-put-records"></a>On the **MyKinesisStream** page, choose **Monitoring**\. If the test is successful, you should see data in the **Put Records** charts\. Depending on your connection, it might take a minute before the data is displayed\.
**Important**  
When you're finished testing, delete the Kinesis data stream to avoid incurring more charges\.  
Or, run the following commands to stop the Greengrass daemon\. This prevents the core from sending messages until you're ready to continue testing\.  

   ```
   cd /greengrass/ggc/core/
   sudo ./greengrassd stop
   ```

1. Remove the **TransferStream** Lambda function from the core\.

   1. In the AWS IoT console, choose **Greengrass**, choose **Groups**, and then choose your group\.

   1. On the **Lambdas** page, choose the ellipses \(**…**\) for the **TransferStream** function, and then choose **Remove function**\.

   1. From **Actions**, choose **Deploy**\.

To view logging information or troubleshoot issues with streams, check the logs for the `TransferStream` and `GGStreamManager` functions\. You must have `root` permissions to read AWS IoT Greengrass logs on the file system\.
+ `TransferStream` writes log entries to `greengrass-root/ggc/var/log/user/region/account-id/TransferStream.log`\.
+ `GGStreamManager` writes log entries to `greengrass-root/ggc/var/log/system/GGStreamManager.log`\.

If you need more troubleshooting information, you can [set the logging level](#stream-manager-console-configure-logging) for **User Lambda logs** to **Debug logs** and then deploy the group again\.

## See also<a name="stream-manager-console-see-also"></a>
+ [Manage data streams on the AWS IoT Greengrass core](stream-manager.md)
+ [Configure AWS IoT Greengrass stream manager](configure-stream-manager.md)
+ [Use StreamManagerClient to work with streams](work-with-streams.md)
+ [Export configurations for supported AWS Cloud destinations](stream-export-configurations.md)
+ [Export data streams to the AWS Cloud \(CLI\)](stream-manager-cli.md)