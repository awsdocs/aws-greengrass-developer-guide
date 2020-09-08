# ML Feedback connector<a name="ml-feedback-connector"></a>

The ML Feedback connector makes it easier to access your machine learning \(ML\) model data for model retraining and analysis\. The connector:
+ Uploads input data \(samples\) used by your ML model to Amazon S3\. Model input can be in any format, such as images, JSON, or audio\. After samples are uploaded to the cloud, you can use them to retrain the model to improve the accuracy and precision of its predictions\. For example, you can use [SageMaker Ground Truth](https://docs.aws.amazon.com/sagemaker/latest/dg/sms.html) to label your samples and [SageMaker](https://docs.aws.amazon.com/sagemaker/latest/dg/whatis.html) to retrain the model\.
+ Publishes the prediction results from the model as MQTT messages\. This lets you monitor and analyze the inference quality of your model in real time\. You can also store prediction results and use them to analyze trends over time\.
+ Publishes metrics about sample uploads and sample data to Amazon CloudWatch\.

To configure this connector, you describe your supported *feedback configurations* in JSON format\. A feedback configuration defines properties such as the destination Amazon S3 bucket, content type, and [sampling strategy](#ml-feedback-connector-sampling-strategies)\. \(A sampling strategy is used to determine which samples to upload\.\)

You can use the ML Feedback connector in the following scenarios:
+ With user\-defined Lambda functions\. Your local inference Lambda functions use the AWS IoT Greengrass Machine Learning SDK to invoke this connector and pass in the target feedback configuration, model input, and model output \(prediction results\)\. For an example, see [Usage Example](#ml-feedback-connector-usage)\.
+ With the [ML Image Classification connector](image-classification-connector.md) \(v2\)\. To use this connector with the ML Image Classification connector, configure the `MLFeedbackConnectorConfigId` parameter for the ML Image Classification connector\.
+ With the [ML Object Detection connector](obj-detection-connector.md)\. To use this connector with the ML Object Detection connector, configure the `MLFeedbackConnectorConfigId` parameter for the ML Object Detection connector\.

**ARN**: `arn:aws:greengrass:region::/connectors/MLFeedback/versions/1`

## Requirements<a name="ml-feedback-connector-req"></a>

This connector has the following requirements:
+ AWS IoT Greengrass Core Software v1\.9\.3 or later\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ One or more Amazon S3 buckets\. The number of buckets you use depends on your sampling strategy\.
+ The [Greengrass group role](group-role.md) configured to allow the `s3:PutObject` action on objects in the destination Amazon S3 bucket, as shown in the following example IAM policy\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": "s3:PutObject",
              "Resource": [
                  "arn:aws:s3:::bucket-name/*"
              ]
          }
      ]
  }
  ```

  The policy should include all destination buckets as resources\. You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\.

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.
+ The [CloudWatch Metrics connector](cloudwatch-metrics-connector.md) added to the Greengrass group and configured\. This is required only if you want to use the metrics reporting feature\.
+ [AWS IoT Greengrass Machine Learning SDK](lambda-functions.md#lambda-sdks-ml) v1\.1\.0 is required to interact with this connector\.

## Parameters<a name="ml-feedback-connector-param"></a>

`FeedbackConfigurationMap`  
A set of one or more feedback configurations that the connector can use to upload samples to Amazon S3\. A feedback configuration defines parameters such as the destination bucket, content type, and [sampling strategy](#ml-feedback-connector-sampling-strategies)\. When this connector is invoked, the calling Lambda function or connector specifies a target feedback configuration\.  
Display name in the AWS IoT console: **Feedback configuration map**  
Required: `true`  
Type: A well\-formed JSON string that defines the set of supported feedback configurations\. For an example, see [FeedbackConfigurationMap example](#ml-feedback-connector-feedbackconfigmap)\.    
  
The ID of a feedback configuration object has the following requirements\.    
  
The ID:  
+ Must be unique across configuration objects\.
+ Must begin with a letter or number\. Can contain lowercase and uppercase letters, numbers, and hyphens\.
+ Must be 2 \- 63 characters in length\.
Required: `true`  
Type: `string`  
Valid pattern: `^[a-zA-Z0-9][a-zA-Z0-9-]{1,62}$`  
Examples: `MyConfig0`, `config-a`, `12id`
The body of a feedback configuration object contains the following properties\.    
`s3-bucket-name`  
The name of the destination Amazon S3 bucket\.  
The group role must allow the `s3:PutObject` action on all destination buckets\. For more information, see [Requirements](#ml-feedback-connector-req)\.
Required: `true`  
Type: `string`  
Valid pattern: `^[a-z0-9\.\-]{3,63}$`  
`content-type`  
The content type of the samples to upload\. All content for an individual feedback configuration must be of the same type\.  
Required: `true`  
Type: `string`  
Examples: `image/jpeg`, `application/json`, `audio/ogg`  
`s3-prefix`  
The key prefix to use for uploaded samples\. A prefix is similar to a directory name\. It allows you to store similar data under the same directory in a bucket\. For more information, see [Object key and metadata](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingMetadata.html) in the *Amazon Simple Storage Service Developer Guide*\.  
Required: `false`  
Type: `string`  
`file-ext`  
The file extension to use for uploaded samples\. Must be a valid file extension for the content type\.  
Required: `false`  
Type: `string`  
Examples: `jpg`, `json`, `ogg`  
`sampling-strategy`  
The [sampling strategy](#ml-feedback-connector-sampling-strategies) to use to filter which samples to upload\. If omitted, the connector tries to upload all the samples that it receives\.  
Required: `false`  
Type: A well\-formed JSON string that contains the following properties\.    
`strategy-name`  
The name of the sampling strategy\.  
Required: `true`  
Type: `string`  
Valid values: `RANDOM_SAMPLING`, `LEAST_CONFIDENCE`, `MARGIN`, or `ENTROPY`  
`rate`  
The rate for the [Random](#ml-feedback-connector-sampling-strategies-random) sampling strategy\.  
Required: `true` if `strategy-name` is `RANDOM_SAMPLING`\.  
Type: `number`  
Valid values: `0.0 - 1.0`  
`threshold`  
The threshold for the [Least Confidence](#ml-feedback-connector-sampling-strategies-least-confidence), [Margin](#ml-feedback-connector-sampling-strategies-margin), or [Entropy](#ml-feedback-connector-sampling-strategies-entropy) sampling strategy\.  
Required: `true` if `strategy-name` is `LEAST_CONFIDENCE`, `MARGIN`, or `ENTROPY`\.  
Type: `number`  
Valid values:  
+ `0.0 - 1.0` for the `LEAST_CONFIDENCE` or `MARGIN` strategy\.
+ `0.0 - no limit` for the `ENTROPY` strategy\.

`RequestLimit`  
The maximum number of requests that the connector can process at a time\.  
You can use this parameter to restrict memory consumption by limiting the number of requests that the connector processes at the same time\. Requests that exceed this limit are ignored\.  
Display name in the AWS IoT console: **Request limit**  
Required: `false`  
Type: `string`  
Valid values: `0 - 999`  
Valid pattern: `^$|^[0-9]{1,3}$`

### Create Connector Example \(AWS CLI\)<a name="ml-feedback-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains the ML Feedback connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyMLFeedbackConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/MLFeedback/versions/1",
            "Parameters": {
                "FeedbackConfigurationMap": "{  \"RandomSamplingConfiguration\": {  \"s3-bucket-name\": \"my-aws-bucket-random-sampling\",  \"content-type\": \"image/png\",  \"file-ext\": \"png\",  \"sampling-strategy\": {  \"strategy-name\": \"RANDOM_SAMPLING\",  \"rate\": 0.5  } },  \"LeastConfidenceConfiguration\": {  \"s3-bucket-name\": \"my-aws-bucket-least-confidence-sampling\",  \"content-type\": \"image/png\",  \"file-ext\": \"png\",  \"sampling-strategy\": {  \"strategy-name\": \"LEAST_CONFIDENCE\",  \"threshold\": 0.4  } } }", 
                "RequestLimit": "10"
            }
        }
    ]
}'
```

### FeedbackConfigurationMap example<a name="ml-feedback-connector-feedbackconfigmap"></a>

The following is an expanded example value for the `FeedbackConfigurationMap` parameter\. This example includes several feedback configurations that use different sampling strategies\.

```
{
    "ConfigID1": {
        "s3-bucket-name": "my-aws-bucket-random-sampling",
        "content-type": "image/png",
        "file-ext": "png",
        "sampling-strategy": {
            "strategy-name": "RANDOM_SAMPLING",
            "rate": 0.5
        }
    },
    "ConfigID2": {
        "s3-bucket-name": "my-aws-bucket-margin-sampling",
        "content-type": "image/png",
        "file-ext": "png",
        "sampling-strategy": {
            "strategy-name": "MARGIN",
            "threshold": 0.4
        }
    },
    "ConfigID3": {
        "s3-bucket-name": "my-aws-bucket-least-confidence-sampling",
        "content-type": "image/png",
        "file-ext": "png",
        "sampling-strategy": {
            "strategy-name": "LEAST_CONFIDENCE",
            "threshold": 0.4
        }
    },
    "ConfigID4": {
        "s3-bucket-name": "my-aws-bucket-entropy-sampling",
        "content-type": "image/png",
        "file-ext": "png",
        "sampling-strategy": {
            "strategy-name": "ENTROPY",
            "threshold": 2
        }
    },
    "ConfigID5": {
        "s3-bucket-name": "my-aws-bucket-no-sampling",
        "s3-prefix": "DeviceA",
        "content-type": "application/json"
    }
}
```

### Sampling strategies<a name="ml-feedback-connector-sampling-strategies"></a>

The connector supports four sampling strategies that determine whether to upload samples that are passed to the connector\. Samples are discrete instances of data that a model uses for a prediction\. You can use sampling strategies to filter for the samples that are most likely to improve model accuracy\.

`RANDOM_SAMPLING`  <a name="ml-feedback-connector-sampling-strategies-random"></a>
Randomly uploads samples based on the supplied rate\. It uploads a sample if a randomly generated value is less than the rate\. The higher the rate, the more samples are uploaded\.  
This strategy disregards any model prediction that is supplied\.

`LEAST_CONFIDENCE`  <a name="ml-feedback-connector-sampling-strategies-least-confidence"></a>
Uploads samples whose maximum confidence probability falls below the supplied threshold\.    
Example scenario:  
Threshold: `.6`  
Model prediction: `[.2, .2, .4, .2]`  
Maximum confidence probability: `.4`  
Result:  
Use the sample because maximum confidence probability \(`.4`\) <= threshold \(`.6`\)\.

`MARGIN`  <a name="ml-feedback-connector-sampling-strategies-margin"></a>
Uploads samples if the margin between the top two confidence probabilities falls within the supplied threshold\. The margin is the difference between the top two probabilities\.    
Example scenario:  
Threshold: `.02`  
Model prediction: `[.3, .35, .34, .01]`  
Top two confidence probabilities: `[.35, .34]`  
Margin: `.01` \(`.35 - .34`\)  
Result:  
Use the sample because margin \(`.01`\) <= threshold \(`.02`\)\.

`ENTROPY`  <a name="ml-feedback-connector-sampling-strategies-entropy"></a>
Uploads samples whose entropy is greater than the supplied threshold\. Uses the model prediction's normalized entropy\.    
Example scenario:  
Threshold: `0.75`  
Model prediction: `[.5, .25, .25]`  
Entropy for prediction: `1.03972`  
Result:  
Use sample because entropy \(`1.03972`\) > threshold \(`0.75`\)\.

## Input data<a name="ml-feedback-connector-data-input"></a>

User\-defined Lambda functions use the `publish` function of the `feedback` client in the AWS IoT Greengrass Machine Learning SDK to invoke the connector\. For an example, see [Usage Example](#ml-feedback-connector-usage)\.

**Note**  
This connector doesn't accept MQTT messages as input data\.

The `publish` function takes the following arguments:

ConfigId  
The ID of the target feedback configuration\. This must match the ID of a feedback configuration defined in the [FeedbackConfigurationMap](#ml-feedback-connector-param) parameter for the ML Feedback connector\.  
Required: true  
Type: string

ModelInput  
The input data that was passed to a model for inference\. This input data is uploaded using the target configuration unless it is filtered out based on the sampling strategy\.  
Required: true  
Type: bytes

ModelPrediction  
The prediction results from the model\. The result type can be a dictionary or a list\. For example, the prediction results from the ML Image Classification connector is a list of probabilities \(such as `[0.25, 0.60, 0.15]`\)\. This data is published to the `/feedback/message/prediction` topic\.  
Required: true  
Type: dictionary or list of `float` values

Metadata  
Customer\-defined, application\-specific metadata that is attached to the uploaded sample and published to the `/feedback/message/prediction` topic\. The connector also inserts a `publish-ts` key with a timestamp value into the metadata\.  
Required: false  
Type: dictionary  
Example: `{"some-key": "some value"}`

## Output data<a name="ml-feedback-connector-data-output"></a>

This connector publishes data to three MQTT topics:
+ Status information from the connector on the `feedback/message/status` topic\.
+ Prediction results on the `feedback/message/prediction` topic\.
+ Metrics destined for CloudWatch on the `cloudwatch/metric/put` topic\.

<a name="connectors-input-output-subscriptions"></a>You must configure subscriptions to allow the connector to communicate on MQTT topics\. For more information, see [Inputs and outputs](connectors.md#connectors-inputs-outputs)\.

**Topic filter**: `feedback/message/status`  
Use this topic to monitor the status of sample uploads and dropped samples\. The connector publishes to this topic every time that it receives a request\.     
**Example output: Sample upload succeeded**  

```
{
  "response": {
    "status": "success",
    "s3_response": {
      "ResponseMetadata": {
        "HostId": "IOWQ4fDEXAMPLEQM+ey7N9WgVhSnQ6JEXAMPLEZb7hSQDASK+Jd1vEXAMPLEa3Km",
        "RetryAttempts": 1,
        "HTTPStatusCode": 200,
        "RequestId": "79104EXAMPLEB723",
        "HTTPHeaders": {
          "content-length": "0",
          "x-amz-id-2": "lbbqaDVFOhMlyU3gRvAX1ZIdg8P0WkGkCSSFsYFvSwLZk3j7QZhG5EXAMPLEdd4/pEXAMPLEUqU=",
          "server": "AmazonS3",
          "x-amz-expiration": "expiry-date=\"Wed, 17 Jul 2019 00:00:00 GMT\", rule-id=\"OGZjYWY3OTgtYWI2Zi00ZDllLWE4YmQtNzMyYzEXAMPLEoUw\"",
          "x-amz-request-id": "79104EXAMPLEB723",
          "etag": "\"b9c4f172e64458a5fd674EXAMPLE5628\"",
          "date": "Thu, 11 Jul 2019 00:12:50 GMT",
          "x-amz-server-side-encryption": "AES256"
        }
      },
      "bucket": "greengrass-feedback-connector-data-us-west-2",
      "ETag": "\"b9c4f172e64458a5fd674EXAMPLE5628\"",
      "Expiration": "expiry-date=\"Wed, 17 Jul 2019 00:00:00 GMT\", rule-id=\"OGZjYWY3OTgtYWI2Zi00ZDllLWE4YmQtNzMyYzEXAMPLEoUw\"",
      "key": "s3-key-prefix/UUID.file_ext",
      "ServerSideEncryption": "AES256"
    }
  },
  "id": "5aaa913f-97a3-48ac-5907-18cd96b89eeb"
}
```
The connector adds the `bucket` and `key` fields to the response from Amazon S3\. For more information about the Amazon S3 response, see [PUT object](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectPUT.html#RESTObjectPUT-responses) in the *Amazon Simple Storage Service API Reference*\.  
**Example output: Sample dropped because of the sampling strategy**  

```
{
  "response": {
    "status": "sample_dropped_by_strategy"
  },
  "id": "4bf5aeb0-d1e4-4362-5bb4-87c05de78ba3"
}
```  
**Example output: Sample upload failed**  
A failure status includes the error message as the `error_message` value and the exception class as the `error` value\.  

```
{
  "response": {
    "status": "fail",
    "error_message": "[RequestId: 4bf5aeb0-d1e4-4362-5bb4-87c05de78ba3] Failed to upload model input data due to exception. Model prediction will not be published. Exception type: NoSuchBucket, error: An error occurred (NoSuchBucket) when calling the PutObject operation: The specified bucket does not exist",
    "error": "NoSuchBucket"
  },
  "id": "4bf5aeb0-d1e4-4362-5bb4-87c05de78ba3"
}
```  
**Example output: Request throttled because of the request limit**  

```
{
  "response": {
    "status": "fail",
    "error_message": "Request limit has been reached (max request: 10 ). Dropping request.",
    "error": "Queue.Full"
  },
  "id": "4bf5aeb0-d1e4-4362-5bb4-87c05de78ba3"
}
```

**Topic filter**: `feedback/message/prediction`  
Use this topic to listen for predictions based on uploaded sample data\. This lets you analyze your model performance in real time\. Model predictions are published to this topic only if data is successfully uploaded to Amazon S3\. Messages published on this topic are in JSON format\. They contain the link to the uploaded data object, the model's prediction, and the metadata included in the request\.  
You can also store prediction results and use them to report and analyze trends over time\. Trends can provide valuable insights\. For example, a *decreasing accuracy over time* trend can help you to decide whether the model needs to be retrained\.    
**Example output**  

```
{
  "source-ref": "s3://greengrass-feedback-connector-data-us-west-2/s3-key-prefix/UUID.file_ext",
  "model-prediction": [
    0.5,
    0.2,
    0.2,
    0.1
  ],
  "config-id": "ConfigID2",
  "metadata": {
    "publish-ts": "2019-07-11 00:12:48.816752"
  }
}
```
You can configure the [IoT Analytics connector](iot-analytics-connector.md) to subscribe to this topic and send the information to AWS IoT Analytics for further or historical analysis\.

**Topic filter**: `cloudwatch/metric/put`  
This is the output topic used to publish metrics to CloudWatch\. This feature requires that you install and configure the [CloudWatch Metrics connector](cloudwatch-metrics-connector.md)\.  
Metrics include:  
+ The number of uploaded samples\.
+ The size of uploaded samples\.
+ The number of errors from uploads to Amazon S3\.
+ The number of dropped samples based on the sampling strategy\.
+ The number of throttled requests\.  
**Example output: Size of the data sample \(published before the actual upload\)**  

```
{
  "request": {
    "namespace": "GreengrassFeedbackConnector",
    "metricData": {
      "value": 47592,
      "unit": "Bytes",
      "metricName": "SampleSize"
    }
  }
}
```  
**Example output: Sample upload succeeded**  

```
{
  "request": {
    "namespace": "GreengrassFeedbackConnector",
    "metricData": {
      "value": 1,
      "unit": "Count",
      "metricName": "SampleUploadSuccess"
    }
  }
}
```  
**Example output: Sample upload succeeded and prediction result published**  

```
{
  "request": {
    "namespace": "GreengrassFeedbackConnector",
    "metricData": {
      "value": 1,
      "unit": "Count",
      "metricName": "SampleAndPredictionPublished"
    }
  }
}
```  
**Example output: Sample upload failed**  

```
{
  "request": {
    "namespace": "GreengrassFeedbackConnector",
    "metricData": {
      "value": 1,
      "unit": "Count",
      "metricName": "SampleUploadFailure"
    }
  }
}
```  
**Example output: Sample dropped because of the sampling strategy**  

```
{
  "request": {
    "namespace": "GreengrassFeedbackConnector",
    "metricData": {
      "value": 1,
      "unit": "Count",
      "metricName": "SampleNotUsed"
    }
  }
}
```  
**Example output: Request throttled because of the request limit**  

```
{
  "request": {
    "namespace": "GreengrassFeedbackConnector",
    "metricData": {
      "value": 1,
      "unit": "Count",
      "metricName": "ErrorRequestThrottled"
    }
  }
}
```

## Usage Example<a name="ml-feedback-connector-usage"></a>

The following example is a user\-defined Lambda function that uses the [AWS IoT Greengrass Machine Learning SDK](lambda-functions.md#lambda-sdks-ml) to send data to the ML Feedback connector\.

**Note**  
You can download the AWS IoT Greengrass Machine Learning SDK from the AWS IoT Greengrass [downloads page](what-is-gg.md#gg-ml-sdk-download)\.

```
import json
import logging
import os
import sys
import greengrass_machine_learning_sdk as ml

client = ml.client('feedback')

try:
    feedback_config_id = os.environ["FEEDBACK_CONFIG_ID"]
    model_input_data_dir = os.environ["MODEL_INPUT_DIR"]
    model_prediction_str = os.environ["MODEL_PREDICTIONS"]
    model_prediction = json.loads(model_prediction_str)
except Exception as e:
    logging.info("Failed to open environment variables. Failed with exception:{}".format(e))
    sys.exit(1)

try:
    with open(os.path.join(model_input_data_dir, os.listdir(model_input_data_dir)[0]), 'rb') as f:
        content = f.read()
except Exception as e:
    logging.info("Failed to open model input directory. Failed with exception:{}".format(e))
    sys.exit(1)    

def invoke_feedback_connector():
    logging.info("Invoking feedback connector.")
    try:
        client.publish(
            ConfigId=feedback_config_id,
            ModelInput=content,
            ModelPrediction=model_prediction
        )
    except Exception as e:
        logging.info("Exception raised when invoking feedback connector:{}".format(e))
        sys.exit(1)    

invoke_feedback_connector()

def function_handler(event, context):
    return
```

## Licenses<a name="ml-feedback-connector-license"></a>

The ML Feedback connector includes the following third\-party software/licensing:<a name="boto-3-licenses"></a>
+ [AWS SDK for Python \(Boto3\)](https://pypi.org/project/boto3/)/Apache License 2\.0
+ [botocore](https://pypi.org/project/botocore/)/Apache License 2\.0
+ [dateutil](https://pypi.org/project/python-dateutil/1.4/)/PSF License
+ [docutils](https://pypi.org/project/docutils/)/BSD License, GNU General Public License \(GPL\), Python Software Foundation License, Public Domain
+ [jmespath](https://pypi.org/project/jmespath/)/MIT License
+ [s3transfer](https://pypi.org/project/s3transfer/)/Apache License 2\.0
+ [urllib3](https://pypi.org/project/urllib3/)/MIT License
+ <a name="six-license"></a>[six](https://github.com/benjaminp/six)/MIT

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## See also<a name="ml-feedback-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)