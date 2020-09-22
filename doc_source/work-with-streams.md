# Use StreamManagerClient to work with streams<a name="work-with-streams"></a>

User\-defined Lambda functions running on the AWS IoT Greengrass core can use the `StreamManagerClient` object in the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks) to create streams in [stream manager](stream-manager.md) and then interact with the streams\. When a Lambda function creates a stream, it defines the AWS Cloud destinations, prioritization, and other export and data retention policies for the stream\. To send data to stream manager, Lambda functions append the data to the stream\. If an export destination is defined for the stream, stream manager exports the stream automatically\.

**Note**  
<a name="stream-manager-clients"></a>Typically, clients of stream manager are user\-defined Lambda functions\. If your business case requires it, you can also allow non\-Lambda processes running on the Greengrass core \(for example, a Docker container\) to interact with stream manager\. For more information, see [Client authentication](stream-manager.md#stream-manager-security-client-authentication)\.

The snippets in this topic show you how clients call `StreamManagerClient` methods to work with streams\. For implementation details about the methods and their arguments, use the links to the SDK reference listed after each snippet\. For tutorials that include a complete Python Lambda function, see [Export data streams to the AWS Cloud \(console\)](stream-manager-console.md) or [Export data streams to the AWS Cloud \(CLI\)](stream-manager-cli.md)\.

Your Lambda function should instantiate `StreamManagerClient` outside of the function handler\. If instantiated in the handler, the function creates a `client` and connection to stream manager every time that it's invoked\.

**Note**  
If you do instantiate `StreamManagerClient` in the handler, you must explicitly call the `close()` method when the `client` completes its work\. Otherwise, the `client` keeps the connection open and another thread running until the script exits\.

`StreamManagerClient` supports the following operations:
+ [Create message stream](#streammanagerclient-create-message-stream)
+ [Append message](#streammanagerclient-append-message)
+ [Read messages](#streammanagerclient-read-messages)
+ [List streams](#streammanagerclient-list-streams)
+ [Describe message stream](#streammanagerclient-describe-message-stream)
+ [Update message stream](#streammanagerclient-update-message-stream)
+ [Delete message stream](#streammanagerclient-delete-message-stream)

## Create message stream<a name="streammanagerclient-create-message-stream"></a>

To create a stream, a user\-defined Lambda function calls the create method and passes in a `MessageStreamDefinition` object\. This object specifies the unique name for the stream and defines how stream manager should handle new data when the maximum stream size is reached\. You can use `MessageStreamDefinition` and its data types \(such as `ExportDefinition`, `StrategyOnFull`, and `Persistence`\) to define other stream properties\. These include:
+ The target AWS IoT Analytics, Kinesis Data Streams, AWS IoT SiteWise, and Amazon S3 destinations for automatic exports\. For more information, see [Export configurations for supported AWS Cloud destinations](stream-export-configurations.md)\.
+ Export priority\. Stream manager exports higher priority streams before lower priority streams\.
+ Maximum batch size and batch interval for AWS IoT Analytics, Kinesis Data Streams, and AWS IoT SiteWise destinations\. Stream manager exports messages when either condition is met\.
+ Time\-to\-live \(TTL\)\. The amount of time to guarantee that the stream data is available for processing\. You should make sure that the data can be consumed within this time period\. This is not a deletion policy\. The data might not be deleted immediately after TTL period\.
+ Stream persistence\. Choose to save streams to the file system to persist data across core restarts or save streams in memory\.
+ Starting sequence number\. Specify the sequence number of the message to use as the starting message in the export\.

For more information about `MessageStreamDefinition`, see the SDK reference for your target language:
+ [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/MessageStreamDefinition.html) in the Java SDK
+ [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.MessageStreamDefinition.html) in the Node\.js SDK
+ [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.MessageStreamDefinition) in the Python SDK

**Note**  
<a name="streammanagerclient-http-config"></a>`StreamManagerClient` also provides a target destination you can use to export streams to an HTTP server\. This target is intended for testing purposes only\. It is not stable or supported for use in production environments\.

After a stream is created, your Lambda functions can [append messages](#streammanagerclient-append-message) to the stream to send data for export and [read messages](#streammanagerclient-append-message) from the stream for local processing\. The number of streams that you create depends on your hardware capabilities and business case\. One strategy is to create a stream for each target channel in AWS IoT Analytics or Kinesis data stream, though you can define multiple targets for a stream\. A stream has a durable lifespan\.

### Requirements<a name="streammanagerclient-create-message-stream-reqs"></a>

This operation has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core version: 1\.10\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.5\.0  \|  Java: 1\.4\.0  \|  Node\.js: 1\.6\.0

**Note**  
Creating streams with an AWS IoT SiteWise or Amazon S3 export destination has the following requirements:  
<a name="streammanagerclient-min-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core version: 1\.11\.0
<a name="streammanagerclient-min-sdk-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.6\.0  \|  Java: 1\.5\.0  \|  Node\.js: 1\.7\.0

### Examples<a name="streammanagerclient-create-message-stream-examples"></a>

The following snippet creates a stream named `StreamName`\. It defines stream properties in the `MessageStreamDefinition` and subordinate data types\.

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
            iot_analytics=None,
            iot_sitewise=None,
            s3=None
        )
    ))
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

Python SDK reference: [create\_message\_stream](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.create_message_stream) \| [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.MessageStreamDefinition)

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
                                    .withIotSiteWise(null)
                                    .withS3(null)
                    )
 
    );
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

Java SDK reference: [createMessageStream](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#createMessageStream-com.amazonaws.greengrass.streammanager.model.MessageStreamDefinition-) \| [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/MessageStreamDefinition.html)

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
                        .withIotSiteWise(null)
                        .withS3(null)
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

Node\.js SDK reference: [createMessageStream](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#createMessageStream) \| [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.MessageStreamDefinition.html)

------

For more information about configuring export destinations, see [Export configurations for supported AWS Cloud destinations](stream-export-configurations.md)\.

 

## Append message<a name="streammanagerclient-append-message"></a>

To send data to stream manager for export, your Lambda functions append the data to the target stream\. The export destination determines the data type to pass to this method\.

### Requirements<a name="streammanagerclient-append-message-reqs"></a>

This operation has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core version: 1\.10\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.5\.0  \|  Java: 1\.4\.0  \|  Node\.js: 1\.6\.0

**Note**  
Appending messages with an AWS IoT SiteWise or Amazon S3 export destination has the following requirements:  
<a name="streammanagerclient-min-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core version: 1\.11\.0
<a name="streammanagerclient-min-sdk-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.6\.0  \|  Java: 1\.5\.0  \|  Node\.js: 1\.7\.0

### Examples<a name="streammanagerclient-append-message-examples"></a>

#### AWS IoT Analytics or Kinesis Data Streams export destinations<a name="streammanagerclient-append-message-blob"></a>

The following snippet appends a message to the stream named `StreamName`\. For AWS IoT Analytics or Kinesis Data Streams destinations, your Lambda functions append a blob of data\.

This snippet has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core version: 1\.10\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.5\.0  \|  Java: 1\.4\.0  \|  Node\.js: 1\.6\.0

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

Python SDK reference: [append\_message](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.append_message)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    long sequenceNumber = client.appendMessage("StreamName", "Arbitrary byte array".getBytes());
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

Java SDK reference: [appendMessage](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#appendMessage-java.lang.String-byte:A-)

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

Node\.js SDK reference: [appendMessage](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#appendMessage)

------

#### AWS IoT SiteWise export destinations<a name="streammanagerclient-append-message-sitewise"></a>

The following snippet appends a message to the stream named `StreamName`\. For AWS IoT SiteWise destinations, your Lambda functions append a serialized `PutAssetPropertyValueEntry` object\. For more information, see [Exporting to AWS IoT SiteWise](stream-export-configurations.md#export-streams-to-sitewise)\.

**Note**  
<a name="BatchPutAssetPropertyValue-data-reqs"></a>When you send data to AWS IoT SiteWise, your data must meet the requirements of the `BatchPutAssetPropertyValue` action\. For more information, see [BatchPutAssetPropertyValue](https://docs.aws.amazon.com/iot-sitewise/latest/APIReference/API_BatchPutAssetPropertyValue.html) in the *AWS IoT SiteWise API Reference*\.

This snippet has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core version: 1\.11\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.6\.0  \|  Java: 1\.5\.0  \|  Node\.js: 1\.7\.0

------
#### [ Python ]

```
client = StreamManagerClient()
 
try:
    # SiteWise requires unique timestamps in all messages and also needs timestamps not earlier
    # than 10 minutes in the past. Add some randomness to time and offset.

    # Note: To create a new asset property data, you should use the classes defined in the
    # greengrasssdk.stream_manager module.

    time_in_nanos = TimeInNanos(
        time_in_seconds=calendar.timegm(time.gmtime()) - random.randint(0, 60), offset_in_nanos=random.randint(0, 10000)
    )
    variant = Variant(double_value=random.random())
    asset = [AssetPropertyValue(value=variant, quality=Quality.GOOD, timestamp=time_in_nanos)]
    putAssetPropertyValueEntry = PutAssetPropertyValueEntry(entry_id=str(uuid.uuid4()), property_alias="PropertyAlias", property_values=asset)
    sequence_number = client.append_message(stream_name="StreamName", Util.validate_and_serialize_to_json_bytes(putAssetPropertyValueEntry))
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

Python SDK reference: [append\_message](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.append_message) \| [PutAssetPropertyValueEntry](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.PutAssetPropertyValueEntry)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    Random rand = new Random();
    // Note: To create a new asset property data, you should use the classes defined in the
    // com.amazonaws.greengrass.streammanager.model.sitewise package.
    List<AssetPropertyValue> entries = new ArrayList<>() ;

    // IoTSiteWise requires unique timestamps in all messages and also needs timestamps not earlier
    // than 10 minutes in the past. Add some randomness to time and offset.
    final int maxTimeRandomness = 60;
    final int maxOffsetRandomness = 10000;
    double randomValue = rand.nextDouble();
    TimeInNanos timestamp = new TimeInNanos()
            .withTimeInSeconds(Instant.now().getEpochSecond() - rand.nextInt(maxTimeRandomness))
            .withOffsetInNanos((long) (rand.nextInt(maxOffsetRandomness)));
    AssetPropertyValue entry = new AssetPropertyValue()
            .withValue(new Variant().withDoubleValue(randomValue))
            .withQuality(Quality.GOOD)
            .withTimestamp(timestamp);
    entries.add(entry);

    PutAssetPropertyValueEntry putAssetPropertyValueEntry = new PutAssetPropertyValueEntry()
            .withEntryId(UUID.randomUUID().toString())
            .withPropertyAlias("PropertyAlias")
            .withPropertyValues(entries);
    long sequenceNumber = client.appendMessage("StreamName", ValidateAndSerialize.validateAndSerializeToJsonBytes(putAssetPropertyValueEntry));
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

Java SDK reference: [appendMessage](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#appendMessage-java.lang.String-byte:A-) \| [PutAssetPropertyValueEntry](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/PutAssetPropertyValueEntry.html)

------
#### [ Node\.js ]

```
const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
        const maxTimeRandomness = 60;
        const maxOffsetRandomness = 10000;
        const randomValue = Math.random();
        // Note: To create a new asset property data, you should use the classes defined in the
        // aws-greengrass-core-sdk StreamManager module.
        const timestamp = new TimeInNanos()
            .withTimeInSeconds(Math.round(Date.now() / 1000) - Math.floor(Math.random() - maxTimeRandomness))
            .withOffsetInNanos(Math.floor(Math.random() * maxOffsetRandomness));
        const entry = new AssetPropertyValue()
            .withValue(new Variant().withDoubleValue(randomValue))
            .withQuality(Quality.GOOD)
            .withTimestamp(timestamp);

        const putAssetPropertyValueEntry =  new PutAssetPropertyValueEntry()
            .withEntryId(`${ENTRY_ID_PREFIX}${i}`)
            .withPropertyAlias("PropertyAlias")
            .withPropertyValues([entry]);
        const sequenceNumber = await client.appendMessage("StreamName", util.validateAndSerializeToJsonBytes(putAssetPropertyValueEntry));
    } catch (e) {
        // Properly handle errors.
    }
});
client.onError((err) => {
    // Properly handle connection errors.
    // This is called only when the connection to the StreamManager server fails.
});
```

Node\.js SDK reference: [appendMessage](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#appendMessage) \| [PutAssetPropertyValueEntry](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.PutAssetPropertyValueEntry .html)

------

#### Amazon S3 export destinations<a name="streammanagerclient-append-message-export-task"></a>

The following snippet appends an export task to the stream named `StreamName`\. For Amazon S3 destinations, your Lambda functions append a serialized `S3ExportTaskDefinition` object that contains information about the source input file and target Amazon S3 object\. If the specified object doesn't exist, Stream Manager creates it for you\. For more information, see [Exporting to Amazon S3](stream-export-configurations.md#export-streams-to-s3)\.

This snippet has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core version: 1\.11\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.6\.0  \|  Java: 1\.5\.0  \|  Node\.js: 1\.7\.0

------
#### [ Python ]

```
client = StreamManagerClient()
 
try:
    # Append an Amazon S3 Task definition and print the sequence number.
    s3_export_task_definition = S3ExportTaskDefinition(input_url="URLToFile", bucket="BucketName", key="KeyName")
    sequence_number = client.append_message(stream_name="StreamName", Util.validate_and_serialize_to_json_bytes(s3_export_task_definition))
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

Python SDK reference: [append\_message](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.append_message) \| [S3ExportTaskDefinition](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.S3ExportTaskDefinition)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    // Append an Amazon S3 export task definition and print the sequence number.
    S3ExportTaskDefinition s3ExportTaskDefinition = new S3ExportTaskDefinition()
        .withBucket("BucketName")
        .withKey("KeyName")
        .withInputUrl("URLToFile");
    long sequenceNumber = client.appendMessage("StreamName", ValidateAndSerialize.validateAndSerializeToJsonBytes(s3ExportTaskDefinition));
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

Java SDK reference: [appendMessage](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#appendMessage-java.lang.String-byte:A-) \| [S3ExportTaskDefinition](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/S3ExportTaskDefinition.html)

------
#### [ Node\.js ]

```
const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
     // Append an Amazon S3 export task definition and print the sequence number.
     const taskDefinition = new S3ExportTaskDefinition()
        .withBucket("BucketName")
        .withKey("KeyName")
        .withInputUrl("URLToFile");
        const sequenceNumber = await client.appendMessage("StreamName", util.validateAndSerializeToJsonBytes(taskDefinition)));
    } catch (e) {
        // Properly handle errors.
    }
});
client.onError((err) => {
    // Properly handle connection errors.
    // This is called only when the connection to the StreamManager server fails.
});
```

Node\.js SDK reference: [appendMessage](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#appendMessage) \| [S3ExportTaskDefinition](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.S3ExportTaskDefinition .html)

------

 

## Read messages<a name="streammanagerclient-read-messages"></a>

Read messages from a stream\.

### Requirements<a name="streammanagerclient-read-messages-reqs"></a>

This operation has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core version: 1\.10\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.5\.0  \|  Java: 1\.4\.0  \|  Node\.js: 1\.6\.0

### Examples<a name="streammanagerclient-read-messages-examples"></a>

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

Python SDK reference: [read\_messages](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.read_messages) \| [ReadMessagesOptions](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.ReadMessagesOptions)

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

Java SDK reference: [readMessages](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#readMessages-java.lang.String-com.amazonaws.greengrass.streammanager.model.ReadMessagesOptions-) \| [ReadMessagesOptions](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/ReadMessagesOptions.html)

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

Node\.js SDK reference: [readMessages](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#readMessages) \| [ReadMessagesOptions](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.ReadMessagesOptions.html)

------

 

## List streams<a name="streammanagerclient-list-streams"></a>

Get the list of streams in stream manager\.

### Requirements<a name="streammanagerclient-list-streams-reqs"></a>

This operation has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core version: 1\.10\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.5\.0  \|  Java: 1\.4\.0  \|  Node\.js: 1\.6\.0

### Examples<a name="streammanagerclient-list-streams-examples"></a>

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

Python SDK reference: [list\_streams](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.list_streams)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    List<String> streamNames = client.listStreams();
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

Java SDK reference: [listStreams](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#listStreams--)

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

Node\.js SDK reference: [listStreams](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#listStreams)

------

 

## Describe message stream<a name="streammanagerclient-describe-message-stream"></a>

Get metadata about a stream, including the stream definition, size, and export status\.

### Requirements<a name="streammanagerclient-describe-message-stream-reqs"></a>

This operation has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core version: 1\.10\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.5\.0  \|  Java: 1\.4\.0  \|  Node\.js: 1\.6\.0

### Examples<a name="streammanagerclient-describe-message-stream-examples"></a>

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

Python SDK reference: [describe\_message\_stream](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.describe_message_stream)

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

Java SDK reference: [describeMessageStream](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#describeMessageStream-java.lang.String-)

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

Node\.js SDK reference: [describeMessageStream](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#describeMessageStream)

------

 

## Update message stream<a name="streammanagerclient-update-message-stream"></a>

Update properties of an existing stream\. You might want to update a stream if your requirements change after the stream was created\. For example:
+ Add a new [export configuration](stream-export-configurations.md) for an AWS Cloud destination\.
+ Increase the maximum size of a stream to change how data is exported or retained\. For example, the stream size in combination with your strategy on full settings might result in data being deleted or rejected before stream manager can process it\.
+ Pause and resume exports; for example, if export tasks are long running and you want to ration your upload data\.

Your Lambda functions follow this high\-level process to update a stream:

1. [Get the description of the stream\.](#streammanagerclient-describe-message-stream)

1. Update the target properties on the corresponding `MessageStreamDefinition` and subordinate objects\.

1. Pass in the updated `MessageStreamDefinition`\. Make sure to include the complete object definitions for the updated stream\. Undefined properties revert to the default values\.

   You can specify the sequence number of the message to use as the starting message in the export\.

### Requirements<a name="-streammanagerclient-update-message-streamreqs"></a>

This operation has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core version: 1\.11\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.11.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.6\.0  \|  Java: 1\.5\.0  \|  Node\.js: 1\.7\.0

### Examples<a name="streammanagerclient-update-message-stream-examples"></a>

The following snippet updates the stream named `StreamName`\. It updates multiple properties of a stream that exports to Kinesis Data Streams\.

------
#### [ Python ]

```
client = StreamManagerClient()
 
try:
    message_stream_info = client.describe_message_stream(STREAM_NAME)
    message_stream_info.definition.max_size=536870912
    message_stream_info.definition.stream_segment_size=33554432
    message_stream_info.definition.time_to_live_millis=3600000
    message_stream_info.definition.strategy_on_full=StrategyOnFull.RejectNewData
    message_stream_info.definition.persistence=Persistence.Memory
    message_stream_info.definition.flush_on_write=False
    message_stream_info.definition.export_definition.kinesis=
        [KinesisConfig(  
            # Updating Export definition to add a Kinesis Stream configuration.
            identifier=str(uuid.uuid4()), kinesis_stream_name=str(uuid.uuid4()))]
    client.update_message_stream(message_stream_info.definition)
except StreamManagerException:
    pass
    # Properly handle errors.
except ConnectionError or asyncio.TimeoutError:
    pass
    # Properly handle errors.
```

Python SDK reference: [updateMessageStream](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.update_message_stream) \| [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.data.html#greengrasssdk.stream_manager.data.MessageStreamDefinition)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    MessageStreamInfo messageStreamInfo = client.describeMessageStream(STREAM_NAME);
    // Update the message stream with new values.
    client.updateMessageStream(
        messageStreamInfo.getDefinition()
            .withStrategyOnFull(StrategyOnFull.RejectNewData) // Required. Updating Strategy on full to reject new data.
            // Max Size update should be greater than initial Max Size defined in Create Message Stream request
            .withMaxSize(536870912L) // Update Max Size to 512 MB.
            .withStreamSegmentSize(33554432L) // Update Segment Size to 32 MB.
            .withFlushOnWrite(true) // Update flush on write to true.
            .withPersistence(Persistence.Memory) // Update the persistence to Memory.
            .withTimeToLiveMillis(3600000L)  // Update TTL to 1 hour.
            .withExportDefinition(
                // Optional. Choose where/how the stream is exported to the AWS Cloud.
                messageStreamInfo.getDefinition().getExportDefinition().
                    // Updating Export definition to add a Kinesis Stream configuration.
                    .withKinesis(new ArrayList<KinesisConfig>() {{
                        add(new KinesisConfig()
                            .withIdentifier(EXPORT_IDENTIFIER)
                            .withKinesisStreamName("test"));
                        }})
            );
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

Java SDK reference: [update\_message\_stream](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#updateMessageStream-java.lang.String-) \| [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/model/MessageStreamDefinition.html)

------
#### [ Node\.js ]

```
const client = new StreamManagerClient();
client.onConnected(async () => {
    try {
        const messageStreamInfo = await c.describeMessageStream(STREAM_NAME);
        await client.updateMessageStream(
            messageStreamInfo.definition
                // Max Size update should be greater than initial Max Size defined in Create Message Stream request
                .withMaxSize(536870912)  // Default is 256 MB. Updating Max Size to 512 MB.
                .withStreamSegmentSize(33554432)  // Default is 16 MB. Updating Segment Size to 32 MB.
                .withTimeToLiveMillis(3600000)  // By default, no TTL is enabled. Update TTL to 1 hour.
                .withStrategyOnFull(StrategyOnFull.RejectNewData)  // Required. Updating Strategy on full to reject new data.
                .withPersistence(Persistence.Memory)  // Default is File. Update the persistence to Memory
                .withFlushOnWrite(true)  // Default is false. Updating to true.
                .withExportDefinition(  
                    // Optional. Choose where/how the stream is exported to the AWS Cloud.
                    messageStreamInfo.definition.exportDefinition
                        // Updating Export definition to add a Kinesis Stream configuration.
                        .withKinesis([new KinesisConfig().withIdentifier(uuidv4()).withKinesisStreamName(uuidv4())])
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

Node\.js SDK reference: [updateMessageStream](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#updateMessageStream) \| [MessageStreamDefinition](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.MessageStreamDefinition.html)

------

### Constraints for updating streams<a name="streammanagerclient-update-constraints"></a>

The following constraints apply when updating streams\. Unless noted in the following list, updates take effect immediately\.
+ You can't update a stream's persistence\. To change this behavior, [delete the stream](#streammanagerclient-delete-message-stream) and [create a stream](#streammanagerclient-create-message-stream) that defines the new persistence policy\.
+ You can update the maximum size of a stream only under the following conditions:
  + The maximum size must be greater or equal to the current size of the stream\. <a name="messagestreaminfo-describe-stream"></a>To find this information, [describe the stream](#streammanagerclient-describe-message-stream) and then check the storage status of the returned `MessageStreamInfo` object\.
  + The maximum size must be greater than or equal to the stream's segment size\.
+ You can update the stream segment size to a value less than the maximum size of the stream\. The updated setting applies to new segments\.
+ Updates to the time to live \(TTL\) property apply to new append operations\. If you decrease this value, stream manager might also delete existing segments that exceed the TTL\.
+ Updates to the strategy on full property apply to new append operations\. If you set the strategy to overwrite the oldest data, stream manager might also overwrite existing segments based on the new setting\.
+ Updates to the flush on write property apply to new messages\.
+ Updates to export configurations apply to new exports\. The update request must include all export configurations that you want to support\. Otherwise, stream manager deletes them\.
  + When you update an export configuration, specify the identifier of the target export configuration\.
  + To add an export configuration, specify a unique identifier for the new export configuration\.
  + To delete an export configuration, omit the export configuration\.
+ To [update](#streammanagerclient-update-message-stream) the starting sequence number of an export configuration in a stream, you must specify a value that's less than the latest sequence number\. <a name="messagestreaminfo-describe-stream"></a>To find this information, [describe the stream](#streammanagerclient-describe-message-stream) and then check the storage status of the returned `MessageStreamInfo` object\.

 

## Delete message stream<a name="streammanagerclient-delete-message-stream"></a>

Deletes a stream\. When you delete a stream, all of the stored data for the stream is deleted from the disk\.

### Requirements<a name="streammanagerclient-delete-message-stream-reqs"></a>

This operation has the following requirements:
+ <a name="streammanagerclient-min-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core version: 1\.10\.0
+ <a name="streammanagerclient-min-sdk-ggc-1.10.0"></a>Minimum AWS IoT Greengrass Core SDK version: Python: 1\.5\.0  \|  Java: 1\.4\.0  \|  Node\.js: 1\.6\.0

### Examples<a name="streammanagerclient-delete-message-stream-examples"></a>

The following snippet deletes the stream named `StreamName`\.

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

Python SDK reference: [deleteMessageStream](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html#greengrasssdk.stream_manager.streammanagerclient.StreamManagerClient.delete_message_stream)

------
#### [ Java ]

```
try (final StreamManagerClient client = GreengrassClientBuilder.streamManagerClient().build()) {
    client.deleteMessageStream("StreamName");
} catch (StreamManagerException e) {
    // Properly handle exception.
}
```

Java SDK reference: [delete\_message\_stream](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html#deleteMessageStream-java.lang.String-)

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

Node\.js SDK reference: [deleteMessageStream](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html#deleteMessageStream)

------

## See also<a name="work-with-streams-see-also"></a>
+ [Manage data streams on the AWS IoT Greengrass core](stream-manager.md)
+ [Configure AWS IoT Greengrass stream manager](configure-stream-manager.md)
+ [Export configurations for supported AWS Cloud destinations](stream-export-configurations.md)
+ [Export data streams to the AWS Cloud \(console\)](stream-manager-console.md)
+ [Export data streams to the AWS Cloud \(CLI\)](stream-manager-cli.md)
+ `StreamManagerClient` in the AWS IoT Greengrass Core SDK reference:
  + [Python](https://aws.github.io/aws-greengrass-core-sdk-python/_apidoc/greengrasssdk.stream_manager.streammanagerclient.html)
  + [Java](https://aws.github.io/aws-greengrass-core-sdk-java/com/amazonaws/greengrass/streammanager/client/StreamManagerClient.html)
  + [Node\.js](https://aws.github.io/aws-greengrass-core-sdk-js/aws-greengrass-core-sdk.StreamManager.StreamManagerClient.html)