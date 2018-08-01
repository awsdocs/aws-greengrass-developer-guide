# Run Lambda Functions on the AWS Greengrass Core<a name="lambda-functions"></a>

AWS Greengrass provides a containerized Lambda runtime environment for user\-defined code\. Lambda functions that are deployed to an AWS Greengrass core run in the core's local Lambda runtime\. Local Lambda functions can be triggered by local events, messages from the cloud, and other sources, which brings local compute functionality to connected devices\. For example, you can use Greengrass Lambda functions to filter device data before transmitting the data to the cloud\.

To deploy a Lambda function to a core, you add the function to a Greengrass group \(by referencing the existing Lambda function\), configure group\-specific settings for the function, and then deploy the group\. If the function accesses AWS services, you also must add any required permissions to the group role\.

## SDKs for Greengrass Lambda Functions<a name="lambda-sdks"></a>

AWS provides two SDKs that are used by Greengrass Lambda functions\.

**AWS Greengrass Core SDK**  
Enables local Lambda functions to interact with local services on the AWS Greengrass core\. This SDK is required by all Greengrass Lambda functions\. SDK versions are available for common programming languages and platforms\.  
The following table lists each supported language or platform and the versions of AWS Greengrass core software that it can run on\.    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lambda-functions.html)
+ Download the AWS Greengrass Core SDK for Python, Java, or Node\.js from the **Software** page of the AWS IoT console\.
+ Download the [AWS Greengrass Core SDK for C](https://github.com/aws/aws-greengrass-core-sdk-c) from GitHub\. This SDK is used by [Lambda executables](#lambda-executables)\.

**AWS SDK**  
Enables local Lambda functions to make direct calls to AWS services, such as Amazon S3, DynamoDB, AWS IoT, and AWS Greengrass\. To use the AWS SDK in a Greengrass Lambda function, you must include it in your deployment package\. When you use the AWS Greengrass Core SDK and the AWS SDK in the same package, make sure that your Lambda functions use the correct namespaces\. Greengrass Lambda functions can't communicate with cloud services when the core is offline\.  
+ Download the AWS SDK from the [ Getting Started Resource Center](https://aws.amazon.com/getting-started/tools-sdks/)\.

For more information about creating a deployment package, see [Create and Package a Lambda Function](create-lambda.md) in the Getting Started tutorial or [Creating a Deployment Package](http://docs.aws.amazon.com/lambda/latest/dg/deployment-package-v2.html) in the *AWS Lambda Developer Guide*\.

### Migrating Cloud\-Based Lambda Functions<a name="lambda-migrate-sdks"></a>

The AWS Greengrass Core SDK follows the AWS SDK programming model, which makes it easy to port Lambda functions that are developed for the cloud to Lambda functions that run on an AWS Greengrass core\.

For example, the following Python Lambda function uses the AWS SDK for Python to publish a message to the topic */some/topic* in the cloud:

```
import boto3
        
client = boto3.client('iot-data')
response = client.publish(
	topic = "/some/topic",
	qos = 0,
	payload = "Some payload".encode()
)
```

To port the function for an AWS Greengrass core, in the `import` statement and `client` initialization, change the `boto3` module name to `greengrasssdk`, as shown in the following example:

```
import greengrasssdk
                        
client = greengrasssdk.client('iot-data')
response = client.publish(
    topic = '/some/topic',
    qos = 0,
    payload = 'Some payload'.encode()
)
```

**Note**  
The AWS Greengrass Core SDK supports sending MQTT messages with QoS = 0 only\.

This makes it possible for you to test your Lambda functions in the cloud and then migrate them to AWS Greengrass with minimal effort\. Lambda executables don't run in the cloud, so you can't use the AWS SDK to test them in the cloud before deployment\.

## Reference Lambda Functions by Alias or Version<a name="lambda-versions-aliases"></a>

Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\. Aliases resolve to version numbers during group deployment\. When you use aliases, the resolved version is updated to the version that the alias is pointing to at the time of deployment\.

AWS Greengrass doesn't support Lambda aliases for **$LATEST** versions\. **$LATEST** versions aren't bound to immutable, published function versions and can be changed at any time, which is counter to the AWS Greengrass principle of version immutability\.

A common practice for keeping your Greengrass Lambda functions updated with code changes is to use an alias named **PRODUCTION** in your Greengrass group and subscriptions\. As you promote new versions of your Lambda function into production, point the alias to the latest stable version and then redeploy the group\. You can also use this method to roll back to a previous version\.

## Group\-Specific Configuration for Greengrass Lambda Functions<a name="lambda-group-config"></a>

AWS Greengrass provides cloud\-based management of Greengrass Lambda functions\. While a function's code and dependencies are managed using AWS Lambda, AWS Greengrass supports the following group\-specific configuration settings:

**Memory limit**  
The memory allocation for the function\. The default is 16 MB\.

**Timeout**  
The amount of time before the function or request is terminated\. The default is 3 seconds\.

**Lifecycle**  
A Lambda function lifecycle can be *on\-demand* or *long\-lived*\. The default is on\-demand\.  
An on\-demand Lambda function starts in a new or reused container when invoked\. Requests to the function might be processed by any available container\. A long\-lived—or *pinned*—Lambda function starts automatically after AWS Greengrass starts and keeps running in its own container \(or sandbox\)\. All requests to the function are processed by the same container\. For more information, see [Lifecycle Configuration for Greengrass Lambda Functions](#lambda-lifecycle)\.

**Read access to /sys directory**  
Whether the function can access the host's /sys folder\. Use this when the function needs to read device information from /sys\. The default is false\.

**Input payload data type**  
The expected encoding type of the input payload for the function, either JSON or binary\. [Lambda executables](#lambda-executables) support only the binary encoding type \(not JSON\)\.  
Accepting binary input data can be useful for functions that interact with device data, because the restricted hardware capabilities of devices often make it difficult or impossible for them to construct a JSON data type\. Support for the binary encoding type is available starting in AWS Greengrass Core Software v1\.5\.0 and AWS Greengrass Core SDK v1\.1\.0\.

**Environment variables**  
Key\-value pairs that can dynamically pass settings to function code and libraries\. Local environment variables work the same way as [AWS Lambda function environment variables](http://docs.aws.amazon.com/lambda/latest/dg/env_variables.html), but are available in the core environment\.

**Resource access policies**  
A list of up to 10 [local resources](access-local-resources.md) and [machine learning resources](ml-inference.md) that the Lambda function is allowed to access, and the corresponding `read-only` or `read-write` permission\. In the console, these *affiliated* resources are listed on the function's **Resources** page\.

## Communication Flows for Greengrass Lambda Functions<a name="lambda-communication"></a>

Greengrass Lambda functions can interact with other members of the AWS Greengrass group, local services, and cloud services \(including AWS services\)\.

### MQTT Messages<a name="lambda-mqtt-messages"></a>

Greengrass Lambda functions can exchange MQTT messages with the following entities:
+ Devices in the group\.
+ Lambda functions in the group\.
+ AWS IoT\.
+ Local Device Shadow service\.

This communication flow uses a publish\-subscribe pattern that's controlled by subscriptions\. A subscription defines a message source, message target, and topic filter\. Messages that are published to a Lambda function target are passed to the function's registered handler\. Subscriptions enable more security and provide predictable interactions\. For more information, see [Greengrass Messaging Workflow](gg-sec.md#gg-msg-workflow)\.

**Note**  
Greengrass Lambda functions can exchange messages with devices, other functions, and local shadows when the core is offline, but messages to AWS IoT are queued\. For more information, see [MQTT Message Queue](gg-core.md#mqtt-message-queue)\.

### Other Communication Flows<a name="lambda-other-communication"></a>
+ To interact with local resources and machine learning models on a core device, Greengrass Lambda functions use platform\-specific operating system interfaces\. For example, you can use the `open` method in the [os](https://docs.python.org/2/library/os.html) module in Python 2\.7 functions\. To allow a function to access a resource, the function must be *affiliated* with the resource and granted `read-only` or `read-write` permission\. For more information, including AWS Greengrass core version availability, see [Access Local Resources with Lambda Functions](access-local-resources.md) and [Perform Machine Learning Inference](ml-inference.md)\.
+ Greengrass Lambda functions can use the AWS SDK to communicate with AWS services\. For more information, see [AWS SDK](#aws-sdk)\.
+ Greengrass Lambda functions can use third\-party interfaces to communicate with external cloud services, similar to cloud\-based Lambda functions\.

**Note**  
Greengrass Lambda functions can't communicate with AWS or other cloud services when the core is offline\.

## Lifecycle Configuration for Greengrass Lambda Functions<a name="lambda-lifecycle"></a>

The Greengrass Lambda function lifecycle determines when a function starts and how it creates and uses containers\. The lifecycle also determines how variables and preprocessing logic that are outside of the function handler are retained\.

AWS Greengrass supports the on\-demand \(default\) or long\-lived lifecycles:
+ **On\-demand** functions start when they are invoked and stop when there are no tasks left to execute\. An invocation of the function creates a separate container \(or sandbox\) to process invocations, unless an existing container is available for reuse\. Data that's sent to the function might be pulled by any of the containers\.

  Multiple invocations of an on\-demand function can run in parallel\.

  Variables and preprocessing logic that are defined outside of the function handler are not retained when new containers are created\.
+ **Long\-lived** \(or *pinned*\) functions start automatically when the AWS Greengrass core starts and run in a single container\. All data that's sent to the function is pulled by the same container\.

  Multiple invocations are queued until earlier invocations are executed\.

  Variables and preprocessing logic that are defined outside of the function handler are retained for every invocation of the handler\.

  Long\-lived Lambda functions are useful when you need to start doing work without any initial input\. For example, a long\-lived function can load and start processing an ML model to be ready for when the function starts receiving device data\.
**Note**  
Remember that long\-lived functions have timeouts that are associated with invocations of their handler\. If you want to execute indefinitely running code, you must start it outside the handler\. Make sure that there's no blocking code outside the handler that might prevent the function from completing its initialization\.

For more information about container reuse, see [Understanding Container Reuse in AWS Lambda](https://aws.amazon.com/blogs/compute/container-reuse-in-lambda/) on the AWS Compute Blog\.

## Lambda Executables<a name="lambda-executables"></a>

This feature is available for AWS Greengrass Core v1\.6\.0 only\.

A Lambda executable is a type of Greengrass Lambda function that you can use to run binary code in the core environment\. It lets you execute device\-specific functionality natively, and benefit from the smaller footprint of compiled code\. Lambda executables can be invoked by events, invoke other functions, and access local resources\.

Lambda executables support only the binary encoding type \(not JSON\), but otherwise you can manage them in your Greengrass group and deploy them like other Greengrass Lambda functions\. However, the process of creating Lambda executables is different from creating Python, Java, and Node\.js Lambda functions:
+ You can't use the AWS Lambda console to create \(or manage\) a Lambda executable\. You can create a Lambda executable only by using the AWS Lambda API\.
+ You upload the function code to AWS Lambda as a compiled executable that includes the [AWS Greengrass Core SDK for C](https://github.com/aws/aws-greengrass-core-sdk-c)\.
+ You specify the executable name as the function handler\.

Lambda executables must implement certain calls and programming patterns in their function code\. For example, the `main` method must:
+ Call `gg_global_init` to initialize Greengrass internal global variables\. This function must be called before creating any threads, and before calling any other AWS Greengrass Core SDK functions\.
+ Call `gg_runtime_start` to register the function handler with the Greengrass Lambda runtime\. This function must be called during initialization\. Calling this function causes the current thread to be used by the runtime\. The optional `GG_RT_OPT_ASYNC` parameter tells this function to not block, but instead to create a new thread for the runtime\. This function uses a `SIGTERM` handler\.

The following snippet is the `main` method from the [simple\_handler\.c](https://github.com/aws/aws-greengrass-core-sdk-c/blob/master/aws-greengrass-core-sdk-c-example/simple_handler.c) code example on GitHub\.

```
int main() {
    gg_error err = GGE_SUCCESS;

    err = gg_global_init(0);
    if(err) {
        gg_log(GG_LOG_ERROR, "gg_global_init failed %d", err);
        goto cleanup;
    }

    gg_runtime_start(handler, 0);

cleanup:
    return -1;
}
```

For more information about requirements, constraints, and other implementation details, see [AWS Greengrass Core SDK for C](https://github.com/aws/aws-greengrass-core-sdk-c)\.

### Create a Lambda Executable<a name="create-lambda-executable"></a>

After you compile your code along with the SDK, use the AWS Lambda API to create a Lambda function and upload your compiled executable\.

**Note**  
Your function must be compiled with a C89 compatible compiler\.

The following example uses the [create\-function](http://docs.aws.amazon.com/cli/latest/reference/lambda/create-function.html) CLI command to create a Lambda executable\. The command specifies:
+ The name of the executable for the handler\. This must be the exact name of your compiled executable\.
+ The path to the `.zip` file that contains the compiled executable\.
+ `arn:aws:greengrass:::runtime/function/executable` for the runtime\. This is the runtime for all Lambda executables\.

**Note**  
For `role`, you can specify the ARN of any Lambda execution role\. AWS Greengrass doesn't use this role but the parameter is required to create the function\. For more information about Lambda execution roles, see [AWS Lambda Permissions Model](http://docs.aws.amazon.com/lambda/latest/dg/intro-permission-model.html) in the *AWS Lambda Developer Guide*\.

```
aws lambda create-function \
--region aws-region \
--function-name function-name \
--handler executable-name \
--role role-arn \
--zip-file fileb://file-name.zip \
--runtime arn:aws:greengrass:::runtime/function/executable
```

Next, use the AWS Lambda API to publish a version and create an alias\.
+ Use [publish\-version](http://docs.aws.amazon.com/cli/latest/reference/lambda/publish-version.html) to publish a function version\.

  ```
  aws lambda publish-version \
  --function-name function-name \
  --region aws-region
  ```
+ Use [create\-alias](http://docs.aws.amazon.com/cli/latest/reference/lambda/create-alias.html) to create an alias the points to the version you just published\. We recommend that you reference Lambda functions by alias when you add them to a Greengrass group\.

  ```
  aws lambda create-alias \
  --function-name function-name \
  --name alias-name \
  --function-version version-number \
  --region aws-region
  ```

**Note**  
The AWS Lambda console doesn't display Lambda executables\. To update the function code, you must also use the Lambda API\.

Then, add the Lambda executable to a Greengrass group, configure it to accept binary input data in its group\-specific settings, and deploy the group\. You can do this in the AWS Greengrass console or by using the AWS Greengrass API\.