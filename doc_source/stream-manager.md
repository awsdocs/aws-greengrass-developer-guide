# Manage data streams on the AWS IoT Greengrass core<a name="stream-manager"></a>

AWS IoT Greengrass stream manager makes it easier and more reliable to transfer high\-volume IoT data to the AWS Cloud\. Stream manager processes data streams locally and exports them to the AWS Cloud automatically\. This feature integrates with common edge scenarios, such as machine learning \(ML\) inference, where data is processed and analyzed locally before being exported to the AWS Cloud or local storage destinations\.

Stream manager simplifies application development\. Your IoT applications can use a standardized mechanism to process high\-volume streams and manage local data retention policies instead of building custom stream management functionality\. IoT applications can read and write to streams\. They can define policies for storage type, size, and data retention on a per\-stream basis to control how stream manager processes and exports streams\.

Stream manager is designed to work in environments with intermittent or limited connectivity\. You can define bandwidth use, timeout behavior, and how stream data is handled when the core is connected or disconnected\. For critical data, you can set priorities to control the order in which streams are exported to the AWS Cloud\.

You can configure automatic exports to AWS IoT Analytics and Amazon Kinesis Data Streams for further processing and analysis in the AWS Cloud\. With AWS IoT Analytics, you can perform advanced analysis on your data to help make business decisions and improve machine learning models\. Kinesis Data Streams is commonly used to aggregate high\-volume data and load it into a data warehouse or map\-reduce cluster\. For more information, see [What is AWS IoT Analytics?](https://docs.aws.amazon.com/iotanalytics/latest/userguide/welcome.html) in the *AWS IoT Analytics User Guide* and [What is Amazon Kinesis Data Streams?](https://docs.aws.amazon.com/streams/latest/dev/what-is-this-service.html) in the *Amazon Kinesis Developer Guide*\.

## Stream management workflow<a name="stream-manager-workflow"></a>

Your IoT applications interact with stream manager through the AWS IoT Greengrass Core SDK\. In a simple workflow, a user\-defined Lambda function running on the AWS IoT Greengrass core consumes IoT data, such as time\-series temperature and pressure metrics\. The Lambda function might filter or compress the data and then call the AWS IoT Greengrass Core SDK to write the data to a stream in stream manager\. Stream manager can export the stream to the AWS Cloud automatically, based on the policies defined for the stream\. User\-defined Lambda functions can also send data directly to local databases or storage repositories\.

Your IoT applications can include multiple user\-defined Lambda functions that read or write to streams\. These local Lambda functions can read and write to streams to filter, aggregate, and analyze data locally\. This makes it possible to respond quickly to local events and extract valuable information before the data is transferred from the core to cloud or local destinations\.

An example workflow is shown in the following diagram\.

![\[Diagram of the stream manager workflow.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/stream-manager-architecture.png)

For tutorials that show you how to create a simple workflow, see [Export data streams to the AWS cloud \(console\)](stream-manager-console.md) or [Export data streams to the AWS cloud \(CLI\)](stream-manager-cli.md)\.

Customizable settings allow you to control how stream manager stores, processes, and exports streams based on business need and environment constraints\. You can configure stream manager parameters to define group\-level runtime settings that apply to all streams on the AWS IoT Greengrass core\. These settings take effect after you deploy the Greengrass group\. For more information, see [Configure AWS IoT Greengrass stream manager](configure-stream-manager.md)\.

Your user\-defined Lambda functions use `StreamManagerClient` in the AWS IoT Greengrass Core SDK to create and interact with streams\. When a stream is created, the Lambda function defines stream parameters, such as destinations, priority, and persistence\. For more information, including example Lambda function code, see [Use StreamManagerClient to work with streams](work-with-streams.md)\.

## Requirements<a name="stream-manager-requirements"></a>

The following requirements apply for the Greengrass stream manager:
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
  + Java SDK \(v1\.4\.0\)
  + Python SDK \(v1\.5\.0\)
  + Node\.js SDK \(v1\.6\.0\)

  You download the version of the SDK that corresponds to your Lambda function runtime and include it in your Lambda function deployment package\.
**Note**  
The AWS IoT Greengrass Core SDK for Python requires Python 3\.7 or later and has other package dependencies\. For more information, see [Create a Lambda function deployment package \(console\)](stream-manager-console.md#stream-manager-console-create-deployment-package) or [Create a Lambda function deployment package \(CLI\)](stream-manager-cli.md#stream-manager-cli-create-deployment-package)\.
+ If you define export destinations for a stream, you must create your export targets and grant permissions to access them in the Greengrass [group role](group-role.md)\. The following targets are supported:
  + Channels in AWS IoT Analytics in the same AWS Region as the Greengrass group\. To allow exports to AWS IoT Analytics, the group role must allow the `iotanalytics:BatchPutMessage` permission to target channels\. For example:

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
  + Streams in Amazon Kinesis Data Streams in the same AWS Region as the Greengrass group\. To allow exports to Kinesis Data Streams, the group role must allow the `kinesis:PutRecords` permission to target data streams\. For example:

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

  You can grant granular or conditional access to resources \(for example, by using a wildcard `*` naming scheme\)\. For more information, see [Adding and removing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

## Data security<a name="stream-manager-security"></a>

When you use stream manager, be aware of the following security considerations\.

### Local data security<a name="stream-manager-security-stream-data"></a>

AWS IoT Greengrass does not encrypt stream data at rest or in transit locally between components on the core device\.
+ **Data at rest**\. Stream data is stored locally in a storage directory\. For data security, AWS IoT Greengrass relies on Unix file permissions and full\-disk encryption, if enabled\. You can use the optional [STREAM\_MANAGER\_STORE\_ROOT\_DIR](configure-stream-manager.md#STREAM_MANAGER_STORE_ROOT_DIR) parameter to specify the storage directory\. If you change this parameter later to use a different storage directory, AWS IoT Greengrass does not delete the previous storage directory or its contents\.

   
+ **Data in transit locally**\. AWS IoT Greengrass does not encrypt stream data in local transit between data sources, Lambda functions, the AWS IoT Greengrass Core SDK, and stream manager\.

   
+ **Data in transit to the AWS Cloud**\. Data streams exported by stream manager to the AWS Cloud use standard AWS service client encryption with Transport Layer Security \(TLS\)\.

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
+ [Export data streams to the AWS cloud \(console\)](stream-manager-console.md)
+ [Export data streams to the AWS cloud \(CLI\)](stream-manager-cli.md)