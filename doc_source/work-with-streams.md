# Use StreamManagerClient to work with streams<a name="work-with-streams"></a>

User\-defined Lambda functions running on the AWS IoT Greengrass core can use the `StreamManagerClient` object in the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks) to create and interact with streams in stream manager\. When a Lambda function creates a stream, it defines the AWS Cloud destinations, prioritization, and other export and data retention policies for the stream\. If an export destination is defined, stream manager exports the stream automatically\.

**Note**  
Typically, clients of stream manager are user\-defined Lambda functions\. If your business case requires it, you can allow non\-Lambda processes running on the Greengrass core \(for example, a Docker container\) to interact with stream manager\. For more information, see [Client authentication](stream-manager.md#stream-manager-security-client-authentication)\.

The snippets in this topic show you how clients use `StreamManagerClient` to work with streams\. For implementation details about the methods and their arguments, use the links to the SDK reference\. For tutorials that use a complete Python Lambda function, see [Export data streams to the AWS cloud \(console\)](stream-manager-console.md) or [Export data streams to the AWS cloud \(CLI\)](stream-manager-cli.md)\.

You should instantiate `StreamManagerClient` outside of the function handler\. If instantiated in the handler, the function creates a `client` and connection to stream manager every time that it's invoked\.

**Note**  
If you do instantiate `StreamManagerClient` in the handler, you must explicitly call the `close()` method when the `client` completes its work\. Otherwise, the `client` keeps the connection open and another thread running until the script exits\.

`StreamManagerClient` supports the following operations:
+ [Create message stream](#streammanagerclient-create-message-stream)
+ [Append message](#streammanagerclient-append-message)
+ [Read messages](#streammanagerclient-read-messages)
+ [List streams](#streammanagerclient-list-streams)
+ [Describe message stream](#streammanagerclient-describe-message-stream)
+ [Delete message stream](#streammanagerclient-delete-message-stream)

## Create message stream<a name="streammanagerclient-create-message-stream"></a>

To create a stream, a user\-defined Lambda function calls the create method and passes in a `MessageStreamDefinition` object\. `MessageStreamDefinition` includes the unique name for the stream and defines how stream manager should handle new data when the maximum stream size is reached\. You can use `MessageStreamDefinition` and its data types \(such as `ExportDefinition`, `StrategyOnFull`, and `Persistence`\) to define other stream properties\. These include:
+ The target AWS IoT Analytics channels and Kinesis data streams\. Stream manager exports the stream to target destinations automatically\. These AWS Cloud resources are created and maintained by the customer\.
+ Export priority\. Stream manager exports higher priority streams before lower priority streams\.
+ Maximum batch size and batch interval\. Stream manager exports messages when either condition is met\.
+ Time\-to\-live \(TTL\)\. The amount of time to guarantee that the stream data is available for processing\. You should make sure that the data can be consumed within this time period\. This is not a deletion policy\. The data might not be deleted immediately after TTL period\.
+ Stream persistence\. Choose to save streams to the file system to persist data across core restarts or save streams in memory\.

For more information about `MessageStreamDefinition`, see the SDK reference for your target language: [Python](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.MessageStreamDefinition), [Java](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/MessageStreamDefinition.html), or [Node\.js](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.MessageStreamDefinition.html)\.

**Note**  
`StreamManagerClient` also provides a target you can use to export streams to an HTTP server\. This target is intended for testing purposes only\. This target is not stable and is not supported for use in production environments\.

The number of streams that you create depends on your hardware capabilities and business case\. One strategy is to create a stream for each target channel in AWS IoT Analytics or Kinesis data stream \(though you can define multiple targets for a stream\)\. A stream has a durable lifespan\. After a stream is created, your Lambda functions can just read and write to it\. However, you can't change a stream definition after it's created\. If you want to make changes, you must delete the stream and then recreate it\. When you delete a stream, all the stored data for the stream is deleted from the disk\.

The following snippet creates a stream named `StreamName`\. It defines stream properties in the `MessageStreamDefinition` and supporting data types\.

------
#### [ Python ]

```
client = StreamManagerClient()
 
try:
    client.create_message_stream(MessageStreamDefinition(
        name="StreamName",  # Required.
        max_size=268435456,  # Default is 256 MB.
        stream_segment_size=16777216,  # Default is 16 MB.
        time_to_live_millis=None,  # By default, no TTL is enabled.
        strategy_on_full=StrategyOnFull.OverwriteOldestData,  # Required.
        persistence=Persistence.File,  # Default is File.
        flush_on_write=False,  # Default is false.
        export_definition=ExportDefinition(  # Optional. Choose where/how the stream is exported to the AWS Cloud.
            kinesis=None,
            iot_analytics=None
        )
    ))
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

SDK reference: [create\_message\_stream](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.create_message_stream) \| [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.MessageStreamDefinition)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    client.createMessageStream(
            new MessageStreamDefinition()
                    .withName("StreamName") // Required.
                    .withMaxSize(268435456L)  // Default is 256 MB.
                    .withStreamSegmentSize(16777216L)  // Default is 16 MB.
                    .withTimeToLiveMillis(null)  // By default, no TTL is enabled.
                    .withStrategyOnFull(StrategyOnFull.OverwriteOldestData)  // Required.
                    .withPersistence(Persistence.File)  // Default is File.
                    .withFlushOnWrite(false)  // Default is false.
                    .withExportDefinition(  // Optional. Choose where/how the stream is exported to the AWS Cloud.
                            new ExportDefinition()
                                    .withKinesis(null)
                                    .withIotAnalytics(null)
                    )
 
    );
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

SDK reference: [createMessageStream](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#createMessageStream-com.amazonaws.greengrass.streammanager.model.MessageStreamDefinition-) \| [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/MessageStreamDefinition.html)

------
#### [ Node\.js ]

```
const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
        await client.createMessageStream(
            new MessageStreamDefinition()
                .withName("StreamName") // Required.
                .withMaxSize(268435456)  // Default is 256 MB.
                .withStreamSegmentSize(16777216)  // Default is 16 MB.
                .withTimeToLiveMillis(null)  // By default, no TTL is enabled.
                .withStrategyOnFull(StrategyOnFull.OverwriteOldestData)  // Required.
                .withPersistence(Persistence.File)  // Default is File.
                .withFlushOnWrite(false)  // Default is false.
                .withExportDefinition(  // Optional. Choose where/how the stream is exported to the AWS Cloud.
                    new ExportDefinition()
                        .withKinesis(null)
                        .withIotAnalytics(null)
                )
        );
    } catch (e) {
        // Properly handle errors.
    }
});
client.onError((err) => {
    // Properly handle connection errors.
    // This is called only when the connection to the StreamManager server fails.
});
```

SDK reference: [createMessageStream](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#createMessageStream) \| [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.MessageStreamDefinition.html)

------

## Append message<a name="streammanagerclient-append-message"></a>

The following snippet appends a message to the stream named `StreamName`\.

------
#### [ Python ]

```
client = StreamManagerClient()
 
try:
    sequence_number = client.append_message(stream_name="StreamName", data=b'Arbitrary bytes data')
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

SDK reference: [append\_message](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.append_message)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    long sequenceNumber = client.appendMessage("StreamName", "Arbitrary byte array".getBytes());
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

SDK reference: [appendMessage](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#appendMessage-java.lang.String-byte:A-)

------
#### [ Node\.js ]

```
const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
        const sequenceNumber = await client.appendMessage("StreamName", Buffer.from("Arbitrary byte array"));
    } catch (e) {
        // Properly handle errors.
    }
});
client.onError((err) => {
    // Properly handle connection errors.
    // This is called only when the connection to the StreamManager server fails.
});
```

SDK reference: [appendMessage](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#appendMessage)

------

## Read messages<a name="streammanagerclient-read-messages"></a>

The following snippet reads messages from the stream named `StreamName`\. The read method takes an optional `ReadMessagesOptions` object that specifies the sequence number to start reading from, the minimum and maximum numbers to read, and a timeout for reading messages\.

------
#### [ Python ]

```
client = StreamManagerClient()
 
try:
    message_list = client.read_messages(
        stream_name="StreamName",
        # By default, if no options are specified, it tries to read one message from the beginning of the stream.
        options=ReadMessagesOptions(
            desired_start_sequence_number=100,
            # Try to read from sequence number 100 or greater. By default, this is 0.
            min_message_count=10,
            # Try to read 10 messages. If 10 messages are not available, then NotEnoughMessagesException is raised. By default, this is 1.
            max_message_count=100,  # Accept up to 100 messages. By default this is 1.
            read_timeout_millis=5000
            # Try to wait at most 5 seconds for the min_messsage_count to be fulfilled. By default, this is 0, which immediately returns the messages or an exception.
        )
    )
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

SDK reference: [read\_messages](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.read_messages) \| [ReadMessagesOptions](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.ReadMessagesOptions)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    List<Message> messages = client.readMessages("StreamName",
            // By default, if no options are specified, it tries to read one message from the beginning of the stream.
            new ReadMessagesOptions()
                    // Try to read from sequence number 100 or greater. By default this is 0.
                    .withDesiredStartSequenceNumber(100L)
                    // Try to read 10 messages. If 10 messages are not available, then NotEnoughMessagesException is raised. By default, this is 1.
                    .withMinMessageCount(10L)
                    // Accept up to 100 messages. By default this is 1.
                    .withMaxMessageCount(100L)
                    // Try to wait at most 5 seconds for the min_messsage_count to be fulfilled. By default, this is 0, which immediately returns the messages or an exception.
                    .withReadTimeoutMillis(Duration.ofSeconds(5L).toMillis())
    );
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

SDK reference: [readMessages](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#readMessages-java.lang.String-com.amazonaws.greengrass.streammanager.model.ReadMessagesOptions-) \| [ReadMessagesOptions](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/ReadMessagesOptions.html)

------
#### [ Node\.js ]

```
const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
        const messages = await client.readMessages("StreamName",
            // By default, if no options are specified, it tries to read one message from the beginning of the stream.
            new ReadMessagesOptions()
                // Try to read from sequence number 100 or greater. By default this is 0.
                .withDesiredStartSequenceNumber(100)
                // Try to read 10 messages. If 10 messages are not available, then NotEnoughMessagesException is thrown. By default, this is 1.
                .withMinMessageCount(10)
                // Accept up to 100 messages. By default this is 1.
                .withMaxMessageCount(100)
                // Try to wait at most 5 seconds for the minMessageCount to be fulfilled. By default, this is 0, which immediately returns the messages or an exception.
                .withReadTimeoutMillis(5 * 1000)
        );
    } catch (e) {
        // Properly handle errors.
    }
});
client.onError((err) => {
    // Properly handle connection errors.
    // This is called only when the connection to the StreamManager server fails.
});
```

SDK reference: [readMessages](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#readMessages) \| [ReadMessagesOptions](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.ReadMessagesOptions.html)

------

## List streams<a name="streammanagerclient-list-streams"></a>

The following snippet gets a list of the streams \(by name\) in stream manager\.

------
#### [ Python ]

```
client = StreamManagerClient()
 
try:
    stream_names = client.list_streams()
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

SDK reference: [list\_streams](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.list_streams)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    List<String> streamNames = client.listStreams();
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

SDK reference: [listStreams](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#listStreams--)

------
#### [ Node\.js ]

```
const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
        const streams = await client.listStreams();
    } catch (e) {
        // Properly handle errors.
    }
});
client.onError((err) => {
    // Properly handle connection errors.
    // This is called only when the connection to the StreamManager server fails.
});
```

SDK reference: [listStreams](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#listStreams)

------

## Describe message stream<a name="streammanagerclient-describe-message-stream"></a>

The following snippet gets metadata about the stream named `StreamName`, including the stream's definition, size, and exporter statuses\.

------
#### [ Python ]

```
client = StreamManagerClient()
 
try:
    stream_description = client.describe_message_stream(stream_name="StreamName")
    if stream_description.export_statuses[0].error_message:
        # The last export of export destination 0 failed with some error
        # Here is the last sequence number that was successfully exported
        stream_description.export_statuses[0].last_exported_sequence_number
 
    if (stream_description.storage_status.newest_sequence_number >
            stream_description.export_statuses[0].last_exported_sequence_number):
        pass
        # The end of the stream is ahead of the last exported sequence number
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

SDK reference: [describe\_message\_stream](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.describe_message_stream)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    MessageStreamInfo description = client.describeMessageStream("StreamName");
    String lastErrorMessage = description.getExportStatuses().get(0).getErrorMessage();
    if (lastErrorMessage != null && !lastErrorMessage.equals("")) {
        // The last export of export destination 0 failed with some error.
        // Here is the last sequence number that was successfully exported.
        description.getExportStatuses().get(0).getLastExportedSequenceNumber();
    }
 
    if (description.getStorageStatus().getNewestSequenceNumber() >
            description.getExportStatuses().get(0).getLastExportedSequenceNumber()) {
        // The end of the stream is ahead of the last exported sequence number.
    }
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

SDK reference: [describeMessageStream](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#describeMessageStream-java.lang.String-)

------
#### [ Node\.js ]

```
const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
        const description = await client.describeMessageStream("StreamName");
        const lastErrorMessage = description.exportStatuses[0].errorMessage;
        if (lastErrorMessage) {
            // The last export of export destination 0 failed with some error.
            // Here is the last sequence number that was successfully exported.
            description.exportStatuses[0].lastExportedSequenceNumber;
        }
 
        if (description.storageStatus.newestSequenceNumber >
            description.exportStatuses[0].lastExportedSequenceNumber) {
            // The end of the stream is ahead of the last exported sequence number.
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

SDK reference: [describeMessageStream](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#describeMessageStream)

------

## Delete message stream<a name="streammanagerclient-delete-message-stream"></a>

The following snippet deletes the stream named `StreamName`\. When you delete a stream, all of the stored data for the stream is deleted from the disk\.

------
#### [ Python ]

```
client = StreamManagerClient()
 
try:
    client.delete_message_stream(stream_name="StreamName")
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

SDK reference: [deleteMessageStream](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.delete_message_stream)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    client.deleteMessageStream("StreamName");
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

SDK reference: [delete\_message\_stream](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#deleteMessageStream-java.lang.String-)

------
#### [ Node\.js ]

```
const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
        await client.deleteMessageStream("StreamName");
    } catch (e) {
        // Properly handle errors.
    }
});
client.onError((err) => {
    // Properly handle connection errors.
    // This is called only when the connection to the StreamManager server fails.
});
```

SDK reference: [deleteMessageStream](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#deleteMessageStream)

------

## See also<a name="work-with-streams-see-also"></a>
+ [Manage data streams on the AWS IoT Greengrass core](stream-manager.md)
+ [Configure AWS IoT Greengrass stream manager](configure-stream-manager.md)
+ [Export data streams to the AWS cloud \(console\)](stream-manager-console.md)
+ [Export data streams to the AWS cloud \(CLI\)](stream-manager-cli.md)
+ `StreamManagerClient` in the AWS IoT Greengrass Core SDK reference:
  + [Python](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html)
  + [Java](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html)
  + [Node\.js](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html)