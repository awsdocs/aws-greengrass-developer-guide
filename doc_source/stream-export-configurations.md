# Export configurations for supported AWS Cloud destinations<a name="stream-export-configurations"></a>

User\-defined Lambda functions use `StreamManagerClient` in the AWS IoT Greengrass Core SDK to interact with stream manager\. When a Lambda function [creates a stream](work-with-streams.md#streammanagerclient-create-message-stream) or [updates a stream](work-with-streams.md#streammanagerclient-create-message-stream), it passes a `MessageStreamDefinition` object that represents stream properties, including the export definition\. The `ExportDefinition` object contains the export configurations defined for the stream\. Stream manager uses these export configurations to determine where and how to export the stream\.

![\[Object model diagram of the ExportDefinition property type.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/stream-manager-exportconfigs.png)

You can define zero or more export configurations on a stream, including multiple export configurations for a single destination type\. For example, you can export a stream to two AWS IoT Analytics channels and one Kinesis data stream\.

**Note**  
<a name="streammanagerclient-http-config"></a>`StreamManagerClient` also provides a target destination you can use to export streams to an HTTP server\. This target is intended for testing purposes only\. It is not stable or supported for use in production environments\.

**Topics**
+ [AWS IoT Analytics channels](#export-to-iot-analytics)
+ [Amazon Kinesis data streams](#export-to-kinesis)
+ [AWS IoT SiteWise asset properties](#export-to-iot-sitewise)
+ [Amazon S3 objects](#export-to-s3)

You are reponsible for maintaining these AWS Cloud resources\.

## AWS IoT Analytics channels<a name="export-to-iot-analytics"></a>

Stream manager supports automatic exports to AWS IoT Analytics\. <a name="ita-export-destination"></a>AWS IoT Analytics lets you perform advanced analysis on your data to help make business decisions and improve machine learning models\. For more information, see [What is AWS IoT Analytics?](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html) in the *AWS IoT Analytics User Guide*\.

In the AWS IoT Greengrass Core SDK, your Lambda functions use the `IoTAnalyticsConfig` to define the export configuration for this destination type\. For more information, see the SDK reference for your target language:
+ [IoTAnalyticsConfig](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.IoTAnalyticsConfig) in the Python SDK
+ [IoTAnalyticsConfig](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/export/IoTAnalyticsConfig.html) in the Java SDK
+ [IoTAnalyticsConfig](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.IoTAnalyticsConfig.html) in the Node\.js SDK

### Requirements<a name="export-to-iot-analytics-reqs"></a>

This export destination has the following requirements:
+ Target channels in AWS IoT Analytics must be in the same AWS account and AWS Region as the Greengrass group\.
+ The [Greengrass group role](group-role.md) must allow the `iotanalytics:BatchPutMessage` permission to target channels\. For example:

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "iotanalytics:BatchPutMessage"
              ],
              "Resource": [
                  "arn:aws:iotanalytics:region:account-id:channel/channel_1_name",
                  "arn:aws:iotanalytics:region:account-id:channel/channel_2_name"
              ]
          }
      ]
  }
  ```

  <a name="wildcards-grant-granular-conditional-access"></a>You can grant granular or conditional access to resources, for example, by using a wildcard `*` naming scheme\. For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

### Exporting to AWS IoT Analytics<a name="export-streams-to-iot-analytics"></a>

To create a stream that exports to AWS IoT Analytics, your Lambda functions [create a stream](work-with-streams.md#streammanagerclient-create-message-stream) with an export definition that includes one or more `IoTAnalyticsConfig` objects\. This object defines export settings, such as the target channel, batch size, batch interval, and priority\.

When your Lambda functions receive data from devices, they [append messages](work-with-streams.md#streammanagerclient-append-message) that contain a blob of data to the target stream\.

Then, stream manager exports the data based on the batch settings and priority defined in the stream's export configurations\.

 

## Amazon Kinesis data streams<a name="export-to-kinesis"></a>

Stream manager supports automatic exports to Amazon Kinesis Data Streams\. <a name="aks-export-destination"></a>Kinesis Data Streams is commonly used to aggregate high\-volume data and load it into a data warehouse or map\-reduce cluster\. For more information, see [What is Amazon Kinesis Data Streams?](https://docs.aws.amazon.com/streams/latest/dev/what-is-this-service.html) in the *Amazon Kinesis Developer Guide*\.

In the AWS IoT Greengrass Core SDK, your Lambda functions use the `KinesisConfig` to define the export configuration for this destination type\. For more information, see the SDK reference for your target language:
+ [KinesisConfig](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.KinesisConfig) in the Python SDK
+ [KinesisConfig](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/export/KinesisConfig.html) in the Java SDK
+ [KinesisConfig](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.KinesisConfig.html) in the Node\.js SDK

### Requirements<a name="export-to-kinesis-reqs"></a>

This export destination has the following requirements:
+ Target streams in Kinesis Data Streams must be in the same AWS account and AWS Region as the Greengrass group\.
+ The [Greengrass group role](group-role.md) must allow the `kinesis:PutRecords` permission to target data streams\. For example:

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
                  "arn:aws:kinesis:region:account-id:stream/stream_1_name",
                  "arn:aws:kinesis:region:account-id:stream/stream_2_name"
              ]
          }
      ]
  }
  ```

  <a name="wildcards-grant-granular-conditional-access"></a>You can grant granular or conditional access to resources, for example, by using a wildcard `*` naming scheme\. For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

### Exporting to Kinesis Data Streams<a name="export-streams-to-kinesis"></a>

To create a stream that exports to Kinesis Data Streams, your Lambda functions [create a stream](work-with-streams.md#streammanagerclient-create-message-stream) with an export definition that includes one or more `KinesisConfig` objects\. This object defines export settings, such as the target data stream, batch size, batch interval, and priority\.

When your Lambda functions receive data from devices, they [append messages](work-with-streams.md#streammanagerclient-append-message) that contain a blob of data to the target stream\.

Then, stream manager exports the data based on the batch settings and priority defined in the stream's export configurations\.

 

## AWS IoT SiteWise asset properties<a name="export-to-iot-sitewise"></a>

Stream manager supports automatic exports to AWS IoT SiteWise\. <a name="itsw-export-destination"></a>AWS IoT SiteWise lets you collect, organize, and analyze data from industrial equipment at scale\. For more information, see [What is AWS IoT SiteWise?](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/what-is-sitewise.html) in the *AWS IoT SiteWise User Guide*\.

In the AWS IoT Greengrass Core SDK, your Lambda functions use the `IoTSiteWiseConfig` to define the export configuration for this destination type\. For more information, see the SDK reference for your target language:
+ [IoTSiteWiseConfig](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.IoTSiteWiseConfig) in the Python SDK
+ [IoTSiteWiseConfig](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/export/IoTSiteWiseConfig.html) in the Java SDK
+ [IoTSiteWiseConfig](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.IoTSiteWiseConfig.html) in the Node\.js SDK

**Note**  
AWS also provides the [IoT SiteWise connector](iot-sitewise-connector.md), which is a pre\-built solution that you can use with OPC\-UA sources\.

### Requirements<a name="export-to-iot-sitewise-reqs"></a>

This export destination has the following requirements:
+ Target asset properties in AWS IoT SiteWise must be in the same AWS account and AWS Region as the Greengrass group\.
**Note**  
For the list of AWS Regions that AWS IoT SiteWise supports, see [AWS IoT SiteWise endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/iot-sitewise.html#iot-sitewise_region) in the *AWS General Reference*\.
+ The [Greengrass group role](group-role.md) must allow the `iotsitewise:BatchPutAssetPropertyValue` permission to target asset properties\. The following example policy uses the `iotsitewise:assetHierarchyPath` condition key to grant access to a target root asset and its children\. You can remove the `Condition` from the policy to allow access to all of your AWS IoT SiteWise assets or specify ARNs of individual assets\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
               "Effect": "Allow",
               "Action": "iotsitewise:BatchPutAssetPropertyValue",
               "Resource": "*",
               "Condition": {
                   "StringLike": {
                       "iotsitewise:assetHierarchyPath": [
                           "/root node asset ID",
                           "/root node asset ID/*"
                       ]
                   }
               }
          }
      ]
  }
  ```

  <a name="wildcards-grant-granular-conditional-access"></a>You can grant granular or conditional access to resources, for example, by using a wildcard `*` naming scheme\. For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

  For important security information, see [ BatchPutAssetPropertyValue authorization](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/security_iam_service-with-iam.html#security_iam_service-with-iam-id-based-policies-batchputassetpropertyvalue-action) in the *AWS IoT SiteWise User Guide*\.

### Exporting to AWS IoT SiteWise<a name="export-streams-to-sitewise"></a>

To create a stream that exports to AWS IoT SiteWise, your Lambda functions [create a stream](work-with-streams.md#streammanagerclient-create-message-stream) with an export definition that includes one or more `IoTSiteWiseConfig` objects\. This object defines export settings, such as the batch size, batch interval, and priority\.

When your Lambda functions receive asset property data from devices, they append messages that contain the data to the target stream\. Messages are JSON\-serialized `PutAssetPropertyValueEntry` objects that contain property values for one or more asset properties\. For more information, see [Append message](work-with-streams.md#streammanagerclient-append-message-sitewise) for AWS IoT SiteWise export destinations\.

**Note**  
<a name="BatchPutAssetPropertyValue-data-reqs"></a>When you send data to AWS IoT SiteWise, your data must meet the requirements of the `BatchPutAssetPropertyValue` action\. For more information, see [BatchPutAssetPropertyValue](https://docs.aws.amazon.com/iot-sitewise/latest/APIReference/API_BatchPutAssetPropertyValue.html) in the *AWS IoT SiteWise API Reference*\.

Then, stream manager exports the data based on the batch settings and priority defined in the stream's export configurations\.

 

You can adjust your stream manager settings and Lambda function logic to design your export strategy\. For example:
+ For near real time exports, set low batch size and interval settings and append the data to the stream when it's received\.
+ To optimize batching, mitigate bandwidth constraints, or minimize cost, your Lambda functions can pool the timestamp\-quality\-value \(TQV\) data points received for a single asset property before appending the data to the stream\. One strategy is to batch entries for up to 10 different property\-asset combinations, or property aliases, in one message instead of sending more than one entry for the same property\. This helps stream manager to remain within [AWS IoT SiteWise quotas](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/quotas.html)\.

 

## Amazon S3 objects<a name="export-to-s3"></a>

Stream manager supports automatic exports to Amazon S3\. <a name="s3-export-destination"></a>You can use Amazon S3 to store and retrieve large amounts of data\. For more information, see [What is Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html) in the *Amazon Simple Storage Service Developer Guide*\.

In the AWS IoT Greengrass Core SDK, your Lambda functions use the `S3ExportTaskExecutorConfig` to define the export configuration for this destination type\. For more information, see the SDK reference for your target language:
+ [S3ExportTaskExecutorConfig](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.S3ExportTaskExecutorConfig) in the Python SDK
+ [S3ExportTaskExecutorConfig](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/export/S3ExportTaskExecutorConfig.html) in the Java SDK
+ [S3ExportTaskExecutorConfig](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.S3ExportTaskExecutorConfig.html) in the Node\.js SDK

### Requirements<a name="export-to-s3-reqs"></a>

This export destination has the following requirements:
+ Target Amazon S3 buckets must be in the same AWS account as the Greengrass group\.
+ If the [default containerization](lambda-group-config.md#lambda-containerization-groupsettings) for the Greengrass group is **Greengrass container**, you must set the [STREAM\_MANAGER\_READ\_ONLY\_DIRS](configure-stream-manager.md#stream-manager-read-only-directories) parameter to use an input file directory that's under `/tmp` or isn't on the root file system\.
+ If a Lambda function running in **Greengrass container** mode writes input files to the input file directory, you must create a local volume resource for the directory and mount the directory to the container with write permissions\. This ensures that the files are written to the root file system and visible outside the container\. For more information, see [Access local resources with Lambda functions and connectors](access-local-resources.md)\.
+ The [Greengrass group role](group-role.md) must allow the following permissions to the target buckets\. For example:

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "s3:PutObject",
                  "s3:UploadPart",
                  "s3:InitiateMultipartUpload",
                  "s3:CompleteMultipartUpload",
                  "s3:AbortMultipartUpload",
                  "s3:ListMultipartUploads"
              ],
              "Resource": [
                  "arn:aws:s3:::bucket-1-name/*",
                  "arn:aws:s3:::bucket-2-name/*"
              ]
          }
      ]
  }
  ```

  <a name="wildcards-grant-granular-conditional-access"></a>You can grant granular or conditional access to resources, for example, by using a wildcard `*` naming scheme\. For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

### Exporting to Amazon S3<a name="export-streams-to-s3"></a>

To create a stream that exports to Amazon S3, your Lambda functions use the `S3ExportTaskExecutorConfig` object to configure the export policy\. The policy defines export settings, such as the multipart upload threshold and priority\. For Amazon S3 exports, stream manager uploads data that it reads from local files on the core device\. To initiate an upload, your Lambda functions append an export task to the target stream\. The export task contains information about the input file and target Amazon S3 object\. Stream manager executes tasks in the sequence that they are appended to the stream\.

**Note**  
<a name="bucket-not-key-must-exist"></a>The target bucket must already exist in your AWS account\. If an object for the specified key doesn't exist, stream manager creates the object for you\.

 This high\-level workflow is shown in the following diagram\.

![\[Diagram of the stream manager workflow for Amazon S3 exports.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/stream-manager-s3.png)

Stream manager uses the multipart upload threshold property, [minimum part size](configure-stream-manager.md#stream-manager-minimum-part-size) setting, and size of the input file to determine how to upload data\. The multipart upload threshold must be greater or equal to the minimum part size\. If you want to upload data in parallel, you can create multiple streams\.

The keys that specify your target Amazon S3 objects can include valid [Java DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html) strings in `!{timestamp:value}` placeholders\. You can use these timestamp placeholders to partition data in Amazon S3 based on the time that the input file data was uploaded\. For example, the following key name resolves to a value such as `my-key/2020/12/31/data.txt`\.

```
my-key/!{timestamp:YYYY}/!{timestamp:MM}/!{timestamp:dd}/data.txt
```

**Note**  
If you want to monitor the export status for a stream, first create a status stream and then configure the export stream to use it\. For more information, see [Monitor export tasks](#monitor-export-status-s3)\.

#### Manage input data<a name="manage-s3-input-data"></a>

You can author code that IoT applications use to manage the lifecycle of the input data\. The following example workflow shows how you might use Lambda functions to manage this data\.

1. A local process receives data from devices or peripherals, and then writes the data to files in a directory on the core device\. These are the input files for stream manager\.
**Note**  
To determine if you must configure access to the input file directory, see the [STREAM\_MANAGER\_READ\_ONLY\_DIRS](configure-stream-manager.md#stream-manager-read-only-directories) parameter\.  
The process that stream manager runs in inherits all of the file system permissions of the [default access identity](lambda-group-config.md#lambda-access-identity-groupsettings) for the group\. Stream manager must have permission to access the input files\. You can use the `chmod(1)` command to change the permission of the files, if necessary\.

1. A Lambda function scans the directory and [appends an export task](work-with-streams.md#streammanagerclient-append-message-export-task) to the target stream when a new file is created\. The task is a JSON\-serialized `S3ExportTaskDefinition` object that specifies the URL of the input file, the target Amazon S3 bucket and key, and optional user metadata\.

1. Stream manager reads the input file and exports the data to Amazon S3 in the order of appended tasks\. <a name="bucket-not-key-must-exist"></a>The target bucket must already exist in your AWS account\. If an object for the specified key doesn't exist, stream manager creates the object for you\.

1. The Lambda function [reads messages](work-with-streams.md#streammanagerclient-read-messages) from a status stream to monitor the export status\. After export tasks are completed, the Lambda function can delete the corresponding input files\. For more information, see [Monitor export tasks](#monitor-export-status-s3)\.

### Monitor export tasks<a name="monitor-export-status-s3"></a>

You can author code that IoT applications use to monitor the status of your Amazon S3 exports\. Your Lambda functions must create a status stream and then configure the export stream to write status updates to the status stream\. A single status stream can receive status updates from multiple streams that export to Amazon S3\.

First, [create a stream](work-with-streams.md#streammanagerclient-create-message-stream) to use as the status stream\. You can configure the size and retention policies for the stream to control the lifespan of the status messages\. For example:
+ Set `Persistence` to `Memory` if you don't want to store the status messages\.
+ Set `StrategyOnFull` to `OverwriteOldestData` so that new status messages are not lost\.

Then, create or update the export stream to use the status stream\. Specifically, set the status configuration property of the stream’s `S3ExportTaskExecutorConfig` export configuration\. This tells stream manager to write status messages about the export tasks to the status stream\. In the `StatusConfig` object, specify the name of the status stream and the level of verbosity\. The following supported values range from least verbose \(`ERROR`\) to most verbose \(`TRACE`\)\. The default is `INFO`\.
+ `ERROR`
+ `WARN`
+ `INFO`
+ `DEBUG`
+ `TRACE`

 

The following example workflow shows how Lambda functions might use a status stream to monitor export status\.

1. As described in the previous workflow, a Lambda function [appends an export task](work-with-streams.md#streammanagerclient-append-message-export-task) to a stream that's configured to write status messages about export tasks to a status stream\. The append operation return a sequence number that represents the task ID\.

1. A Lambda function [reads messages](work-with-streams.md#streammanagerclient-read-messages) sequentially from the status stream, and then filters the messages based on the stream name and task ID or based on an export task property from the message context\. For example, the Lambda function can filter by the input file URL of the export task, which is represented by the `S3ExportTaskDefinition` object in the message context\.

   The following status codes indicate that an export task has reached a completed state:
   + `Success`\. The upload was completed successfully\.
   + `Failure`\. Stream manager encountered an error, for example, the specified bucket does not exist\. After resolving the issue, you can append the export task to the stream again\.
   + `Canceled`\. The task was aborted because the stream or export definition was deleted, or the time\-to\-live \(TTL\) period of the task expired\.
**Note**  
The task might also have a status of `InProgress` or `Warning`\. Stream manager issues warnings when an event returns an error that doesn't affect the execution of the task\. For example, a failure to clean up an aborted partial upload returns a warning\.

1. After export tasks are completed, the Lambda function can delete the corresponding input files\.

The following example shows how a Lambda function might read and process status messages\.

------
#### [ Python ]

```
import time
from greengrasssdk.stream_manager import (
    ReadMessagesOptions,
    Status,
    StatusConfig,
    StatusLevel,
    StatusMessage,
    StreamManagerClient,
)
from greengrasssdk.stream_manager.util import Util

client = StreamManagerClient()
 
try:
    # Read the statuses from the export status stream
    is_file_uploaded_to_s3 = False
    while not is_file_uploaded_to_s3:
        try:
            messages_list = client.read_messages(
                "StatusStreamName", ReadMessagesOptions(min_message_count=1, read_timeout_millis=1000)
            )
            for message in messages_list:
                # Deserialize the status message first.
                status_message = Util.deserialize_json_bytes_to_obj(message.payload, StatusMessage)

                # Check the status of the status message. If the status is "Success",
                # the file was successfully uploaded to S3.
                # If the status was either "Failure" or "Cancelled", the server was unable to upload the file to S3.
                # We will print the message for why the upload to S3 failed from the status message.
                # If the status was "InProgress", the status indicates that the server has started uploading
                # the S3 task.
                if status_message.status == Status.Success:
                    logger.info("Successfully uploaded file at path " + file_url + " to S3.")
                    is_file_uploaded_to_s3 = True
                elif status_message.status == Status.Failure or status_message.status == Status.Canceled:
                    logger.info(
                        "Unable to upload file at path " + file_url + " to S3. Message: " + status_message.message
                    )
                    is_file_uploaded_to_s3 = True
            time.sleep(5)
        except StreamManagerException:
            logger.exception("Exception while running")
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

Python SDK reference: [read\_messages](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.read_messages) \| [StatusMessage](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.StatusMessage)

------
#### [ Java ]

```
import com.amazonaws.greengrass.streammanager.client.StreamManagerClient;
import com.amazonaws.greengrass.streammanager.client.utils.ValidateAndSerialize;
import com.amazonaws.greengrass.streammanager.model.ReadMessagesOptions;
import com.amazonaws.greengrass.streammanager.model.Status;
import com.amazonaws.greengrass.streammanager.model.StatusConfig;
import com.amazonaws.greengrass.streammanager.model.StatusLevel;
import com.amazonaws.greengrass.streammanager.model.StatusMessage;

try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    try {
        boolean isS3UploadComplete = false;
        while (!isS3UploadComplete) {
            try {
                // Read the statuses from the export status stream
                List<Message> messages = client.readMessages("StatusStreamName",
                    new ReadMessagesOptions().withMinMessageCount(1L).withReadTimeoutMillis(1000L));
                for (Message message : messages) {
                    // Deserialize the status message first.
                    StatusMessage statusMessage = ValidateAndSerialize.deserializeJsonBytesToObj(message.getPayload(), StatusMessage.class);
                    // Check the status of the status message. If the status is "Success", the file was successfully uploaded to S3.
                    // If the status was either "Failure" or "Canceled", the server was unable to upload the file to S3.
                    // We will print the message for why the upload to S3 failed from the status message.
                    // If the status was "InProgress", the status indicates that the server has started uploading the S3 task.
                    if (Status.Success.equals(statusMessage.getStatus())) {
                        System.out.println("Successfully uploaded file at path " + FILE_URL + " to S3.");
                        isS3UploadComplete = true;
                     } else if (Status.Failure.equals(statusMessage.getStatus()) || Status.Canceled.equals(statusMessage.getStatus())) {
                        System.out.println(String.format("Unable to upload file at path %s to S3. Message %s",
                            statusMessage.getStatusContext().getS3ExportTaskDefinition().getInputUrl(),
                            statusMessage.getMessage()));
                        sS3UploadComplete = true;
                    }
                }
            } catch (StreamManagerException ignored) {
            } finally {
                // Sleep for sometime for the S3 upload task to complete before trying to read the status message.
                Thread.sleep(5000);
            }
        } catch (e) {
        // Properly handle errors.
    }
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

Java SDK reference: [readMessages](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#readMessages-java.lang.String-com.amazonaws.greengrass.streammanager.model.ReadMessagesOptions-) \| [StatusMessage](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/StatusMessage.html)

------
#### [ Node\.js ]

```
const {
    StreamManagerClient, ReadMessagesOptions,
    Status, StatusConfig, StatusLevel, StatusMessage,
    util,
} = require('aws-greengrass-core-sdk').StreamManager;

const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
        let isS3UploadComplete = false;
        while (!isS3UploadComplete) {
            try {
                // Read the statuses from the export status stream
                const messages = await c.readMessages("StatusStreamName",
                    new ReadMessagesOptions()
                        .withMinMessageCount(1)
                        .withReadTimeoutMillis(1000));

                messages.forEach((message) => {
                    // Deserialize the status message first.
                    const statusMessage = util.deserializeJsonBytesToObj(message.payload, StatusMessage);
                    // Check the status of the status message. If the status is 'Success', the file was successfully uploaded to S3.
                    // If the status was either 'Failure' or 'Cancelled', the server was unable to upload the file to S3.
                    // We will print the message for why the upload to S3 failed from the status message.
                    // If the status was "InProgress", the status indicates that the server has started uploading the S3 task.
                    if (statusMessage.status === Status.Success) {
                        console.log(`Successfully uploaded file at path ${FILE_URL} to S3.`);
                        isS3UploadComplete = true;
                    } else if (statusMessage.status === Status.Failure || statusMessage.status === Status.Canceled) {
                        console.log(`Unable to upload file at path ${FILE_URL} to S3. Message: ${statusMessage.message}`);
                        isS3UploadComplete = true;
                    }
                });
                // Sleep for sometime for the S3 upload task to complete before trying to read the status message.
                await new Promise((r) => setTimeout(r, 5000));
            } catch (e) {
                // Ignored
            }
    } catch (e) {
        // Properly handle errors.
    }
});
client.onError((err) => {
    // Properly handle connection errors.
    // This is called only when the connection to the StreamManager server fails.
});
```

Node\.js SDK reference: [readMessages](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#readMessages) \| [StatusMessage](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StatusMessage.html)

------