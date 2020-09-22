# Manage data streams on the AWS IoT Greengrass core<a name="stream-manager"></a>

AWS IoT Greengrass stream manager makes it easier and more reliable to transfer high\-volume IoT data to the AWS Cloud\. Stream manager processes data streams locally and exports them to the AWS Cloud automatically\. This feature integrates with common edge scenarios, such as machine learning \(ML\) inference, where data is processed and analyzed locally before being exported to the AWS Cloud or local storage destinations\.

Stream manager simplifies application development\. Your IoT applications can use a standardized mechanism to process high\-volume streams and manage local data retention policies instead of building custom stream management functionality\. IoT applications can read and write to streams\. They can define policies for storage type, size, and data retention on a per\-stream basis to control how stream manager processes and exports streams\.

Stream manager is designed to work in environments with intermittent or limited connectivity\. You can define bandwidth use, timeout behavior, and how stream data is handled when the core is connected or disconnected\. For critical data, you can set priorities to control the order in which streams are exported to the AWS Cloud\.

You can configure automatic exports to the AWS Cloud for storage or further processing and analysis\. Stream manager supports exporting to the following AWS Cloud destinations\.<a name="supported-export-destinations"></a>
+ Channels in AWS IoT Analytics\. <a name="ita-export-destination"></a>AWS IoT Analytics lets you perform advanced analysis on your data to help make business decisions and improve machine learning models\. For more information, see [What is AWS IoT Analytics?](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html) in the *AWS IoT Analytics User Guide*\.
+ Streams in Kinesis Data Streams\. <a name="aks-export-destination"></a>Kinesis Data Streams is commonly used to aggregate high\-volume data and load it into a data warehouse or map\-reduce cluster\. For more information, see [What is Amazon Kinesis Data Streams?](https://docs.aws.amazon.com/streams/latest/dev/what-is-this-service.html) in the *Amazon Kinesis Developer Guide*\.
+ Asset properties in AWS IoT SiteWise\. <a name="itsw-export-destination"></a>AWS IoT SiteWise lets you collect, organize, and analyze data from industrial equipment at scale\. For more information, see [What is AWS IoT SiteWise?](https://docs.aws.amazon.com/iot-sitewise/latest/userguide/what-is-sitewise.html) in the *AWS IoT SiteWise User Guide*\.
+ Objects in Amazon S3\. <a name="s3-export-destination"></a>You can use Amazon S3 to store and retrieve large amounts of data\. For more information, see [What is Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html) in the *Amazon Simple Storage Service Developer Guide*\.

## Stream management workflow<a name="stream-manager-workflow"></a>

Your IoT applications interact with stream manager through the AWS IoT Greengrass Core SDK\. In a simple workflow, a user\-defined Lambda function running on the Greengrass core consumes IoT data, such as time\-series temperature and pressure metrics\. The Lambda function might filter or compress the data and then call the AWS IoT Greengrass Core SDK to write the data to a stream in stream manager\. Stream manager can export the stream to the AWS Cloud automatically, based on the policies defined for the stream\. User\-defined Lambda functions can also send data directly to local databases or storage repositories\.

Your IoT applications can include multiple user\-defined Lambda functions that read or write to streams\. These local Lambda functions can read and write to streams to filter, aggregate, and analyze data locally\. This makes it possible to respond quickly to local events and extract valuable information before the data is transferred from the core to cloud or local destinations\.

An example workflow is shown in the following diagram\.

![\[Diagram of the stream manager workflow.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/stream-manager-architecture.png)

To use stream manager, start by configuring stream manager parameters to define group\-level runtime settings that apply to all streams on the Greengrass core\. These customizable settings allow you to control how stream manager stores, processes, and exports streams based on your business need and environment constraints\. For more information, see [Configure AWS IoT Greengrass stream manager](configure-stream-manager.md)\.

After you configure stream manager, you can create and deploy your IoT applications\. These are typically user\-defined Lambda functions that use `StreamManagerClient` in the AWS IoT Greengrass Core SDK to create and interact with streams\. During stream creation, the Lambda function defines per\-stream policies, such as export destinations, priority, and persistence\. For more information, including code snippets for `StreamManagerClient` operations, see [Use StreamManagerClient to work with streams](work-with-streams.md)\.

For tutorials that configure a simple workflow, see [Export data streams to the AWS Cloud \(console\)](stream-manager-console.md) or [Export data streams to the AWS Cloud \(CLI\)](stream-manager-cli.md)\.

## Requirements<a name="stream-manager-requirements"></a>

The following requirements apply for using stream manager:
+ You must use AWS IoT Greengrass Core software v1\.10 or later, with stream manager enabled\. For more information, see [Configure AWS IoT Greengrass stream manager](configure-stream-manager.md)\.
**Note**  <a name="stream-manager-not-supported-openwrt"></a>
Stream manager is not supported on OpenWrt distributions\.
+ The Java 8 runtime \(JDK 8\) must be installed on the core\.<a name="install-java8-runtime-general"></a>
  + For Debian\-based distributions \(including Raspbian\) or Ubuntu\-based distributions, run the following command:

    ```
    sudo apt install openjdk-8-jdk
    ```
  + For Red Hat\-based distributions \(including Amazon Linux\), run the following command:

    ```
    sudo yum install java-1.8.0-openjdk
    ```

    For more information, see [ How to download and install prebuilt OpenJDK packages](https://openjdk.java.net/install/) in the OpenJDK documentation\.

   
+ Stream manager requires a minimum of 70 MB RAM in addition to your base AWS IoT Greengrass Core software\. Your total memory requirement depends on your workload\.

   
+ User\-defined Lambda functions must use the [AWS IoT Greengrass Core SDK](lambda-functions.md#lambda-sdks-core) to interact with stream manager\. The AWS IoT Greengrass Core SDK is available in several languages, but only the following versions support stream manager operations:<a name="streammanagerclient-sdk-versions"></a>
  + Java SDK \(v1\.4\.0 or later\)
  + Python SDK \(v1\.5\.0 or later\)
  + Node\.js SDK \(v1\.6\.0 or later\)

  Download the version of the SDK that corresponds to your Lambda function runtime and include it in your Lambda function deployment package\.
**Note**  
The AWS IoT Greengrass Core SDK for Python requires Python 3\.7 or later and has other package dependencies\. For more information, see [Create a Lambda function deployment package \(console\)](stream-manager-console.md#stream-manager-console-create-deployment-package) or [Create a Lambda function deployment package \(CLI\)](stream-manager-cli.md#stream-manager-cli-create-deployment-package)\.
+ If you define AWS Cloud export destinations for a stream, you must create your export targets and grant access permissions in the Greengrass group role\. Depending on the destination, other requirements might also apply\. For more information, see:<a name="export-destinations-links"></a>
  + [AWS IoT Analytics channels](stream-export-configurations.md#export-to-iot-analytics)
  + [Amazon Kinesis data streams](stream-export-configurations.md#export-to-kinesis)
  + [AWS IoT SiteWise asset properties](stream-export-configurations.md#export-to-iot-sitewise)
  + [Amazon S3 objects](stream-export-configurations.md#export-to-s3)

  You are responsible for maintaining these AWS Cloud resources\.

## Data security<a name="stream-manager-security"></a>

When you use stream manager, be aware of the following security considerations\.

### Local data security<a name="stream-manager-security-stream-data"></a>

AWS IoT Greengrass does not encrypt stream data at rest or in transit locally between components on the core device\.
+ **Data at rest**\. Stream data is stored locally in a storage directory on the Greengrass core\. For data security, AWS IoT Greengrass relies on Unix file permissions and full\-disk encryption, if enabled\. You can use the optional [STREAM\_MANAGER\_STORE\_ROOT\_DIR](configure-stream-manager.md#STREAM_MANAGER_STORE_ROOT_DIR) parameter to specify the storage directory\. If you change this parameter later to use a different storage directory, AWS IoT Greengrass does not delete the previous storage directory or its contents\.

   
+ **Data in transit locally**\. AWS IoT Greengrass does not encrypt stream data in local transit on the core between data sources, Lambda functions, the AWS IoT Greengrass Core SDK, and stream manager\.

   
+ **Data in transit to the AWS Cloud**\. Data streams exported by stream manager to the AWS Cloud use standard AWS service client encryption with Transport Layer Security \(TLS\)\.

For more information, see [Data encryption](data-encryption.md)\.

### Client authentication<a name="stream-manager-security-client-authentication"></a>

Stream manager clients use the AWS IoT Greengrass Core SDK to communicate with stream manager\. When client authentication is enabled, only Lambda functions in the Greengrass group can interact with streams in stream manager\. When client authentication is disabled, any process running on the Greengrass core \(such as [Docker containers](docker-app-connector.md)\) can interact with streams in stream manager\. You should disable authentication only if your business case requires it\.

You use the [STREAM\_MANAGER\_AUTHENTICATE\_CLIENT](configure-stream-manager.md#STREAM_MANAGER_AUTHENTICATE_CLIENT) parameter to set the client authentication mode\. You can configure this parameter from the console or AWS IoT Greengrass API\. Changes take effect after the group is deployed\.


****  

|   | Enabled | Disabled | 
| --- | --- | --- | 
| Parameter value | `true` \(default and recommended\) | `false` | 
| Allowed clients | User\-defined Lambda functions in the Greengrass group | User\-defined Lambda functions in the Greengrass group Other processes running on the Greengrass core device | 

## See also<a name="stream-manager-see-also"></a>
+ [Configure AWS IoT Greengrass stream manager](configure-stream-manager.md)
+ [Use StreamManagerClient to work with streams](work-with-streams.md)
+ [Export configurations for supported AWS Cloud destinations](stream-export-configurations.md)
+ [Export data streams to the AWS Cloud \(console\)](stream-manager-console.md)
+ [Export data streams to the AWS Cloud \(CLI\)](stream-manager-cli.md)