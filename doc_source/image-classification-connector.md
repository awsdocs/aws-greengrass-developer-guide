# ML Image Classification connector<a name="image-classification-connector"></a>

The ML Image Classification [connectors](connectors.md) provide a machine learning \(ML\) inference service that runs on the AWS IoT Greengrass core\. This local inference service performs image classification using a model trained by the Amazon SageMaker image classification algorithm\.

User\-defined Lambda functions use the AWS IoT Greengrass Machine Learning SDK to submit inference requests to the local inference service\. The service runs inference locally and returns probabilities that the input image belongs to specific categories\.

AWS IoT Greengrass provides the following versions of this connector, which is available for multiple platforms\.

------
#### [ Version 2 ]


| Connector | Description and ARN | 
| --- | --- | 
| ML Image Classification Aarch64 JTX2 |  Image classification inference service for NVIDIA Jetson TX2\. Supports GPU acceleration\. **ARN:** `arn:aws:greengrass:region::/connectors/ImageClassificationAarch64JTX2/versions/2` | 
| ML Image Classification x86\_64 |  Image classification inference service for x86\_64 platforms\. **ARN:** `arn:aws:greengrass:region::/connectors/ImageClassificationx86-64/versions/2` | 
| ML Image Classification ARMv7 |  Image classification inference service for ARMv7 platforms\. **ARN:** `arn:aws:greengrass:region::/connectors/ImageClassificationARMv7/versions/2` | 

------
#### [ Version 1 ]


| Connector | Description and ARN | 
| --- | --- | 
| ML Image Classification Aarch64 JTX2 |  Image classification inference service for NVIDIA Jetson TX2\. Supports GPU acceleration\. **ARN:** `arn:aws:greengrass:region::/connectors/ImageClassificationAarch64JTX2/versions/1` | 
| ML Image Classification x86\_64 |  Image classification inference service for x86\_64 platforms\. **ARN:** `arn:aws:greengrass:region::/connectors/ImageClassificationx86-64/versions/1` | 
| ML Image Classification Armv7 |  Image classification inference service for Armv7 platforms\. **ARN:** `arn:aws:greengrass:region::/connectors/ImageClassificationARMv7/versions/1` | 

------

For information about version changes, see the [Changelog](#image-classification-connector-changelog)\.

## Requirements<a name="image-classification-connector-req"></a>

These connectors have the following requirements:

------
#### [ Version 2 ]
+ AWS IoT Greengrass Core Software v1\.9\.3 or later\.
+ [Python](https://www.python.org/) version 3\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="req-image-classification-framework"></a>Dependencies for the Apache MXNet framework installed on the core device\. For more information, see [Installing MXNet dependencies on the AWS IoT Greengrass core](#image-classification-connector-config)\.
+ <a name="req-image-classification-resource"></a>An [ML resource](ml-inference.md#ml-resources) in the Greengrass group that references an Amazon SageMaker model source\. This model must be trained by the Amazon SageMaker image classification algorithm\. For more information, see [Image classification algorithm](https://docs.aws.amazon.com/sagemaker/latest/dg/image-classification.html) in the *Amazon SageMaker Developer Guide*\.
+ <a name="req-image-classification-feedback"></a>The [ML Feedback connector](ml-feedback-connector.md) added to the Greengrass group and configured\. This is required only if you want to use the connector to upload model input data and publish predictions to an MQTT topic\.
+ <a name="req-image-classification-policy"></a>The [Greengrass group role](group-role.md) configured to allow the `sagemaker:DescribeTrainingJob` action on the target training job, as shown in the following example IAM policy\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "sagemaker:DescribeTrainingJob"
              ],
              "Resource": "arn:aws:sagemaker:region:account-id:training-job:training-job-name"
          }
      ]
  }
  ```

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

  You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\. If you change the target training job in the future, make sure to update the group role\.
+ [AWS IoT Greengrass Machine Learning SDK](lambda-functions.md#lambda-sdks-ml) v1\.1\.0 is required to interact with this connector\.

------
#### [ Version 1 ]
+ AWS IoT Greengrass Core Software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ <a name="req-image-classification-framework"></a>Dependencies for the Apache MXNet framework installed on the core device\. For more information, see [Installing MXNet dependencies on the AWS IoT Greengrass core](#image-classification-connector-config)\.
+ <a name="req-image-classification-resource"></a>An [ML resource](ml-inference.md#ml-resources) in the Greengrass group that references an Amazon SageMaker model source\. This model must be trained by the Amazon SageMaker image classification algorithm\. For more information, see [Image classification algorithm](https://docs.aws.amazon.com/sagemaker/latest/dg/image-classification.html) in the *Amazon SageMaker Developer Guide*\.
+ <a name="req-image-classification-policy"></a>The [Greengrass group role](group-role.md) configured to allow the `sagemaker:DescribeTrainingJob` action on the target training job, as shown in the following example IAM policy\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "sagemaker:DescribeTrainingJob"
              ],
              "Resource": "arn:aws:sagemaker:region:account-id:training-job:training-job-name"
          }
      ]
  }
  ```

  <a name="set-up-group-role"></a>For the group role requirement, you must configure the role to grant the required permissions and make sure the role has been added to the group\. For more information, see [Managing the Greengrass group role \(console\)](group-role.md#manage-group-role-console) or [Managing the Greengrass group role \(CLI\)](group-role.md#manage-group-role-cli)\.

  You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\. If you change the target training job in the future, make sure to update the group role\.
+ [AWS IoT Greengrass Machine Learning SDK](lambda-functions.md#lambda-sdks-ml) v1\.0\.0 or later is required to interact with this connector\.

------

## Connector Parameters<a name="image-classification-connector-param"></a>

These connectors provide the following parameters\.

------
#### [ Version 2 ]

`MLModelDestinationPath`  <a name="param-image-classification-mdlpath"></a>
The absolute local path of the ML resource inside the Lambda environment\. This is the destination path that's specified for the ML resource\.  
If you created the ML resource in the console, this is the local path\.
Display name in the AWS IoT console: **Model destination path**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`MLModelResourceId`  <a name="param-image-classification-mdlresourceid"></a>
The ID of the ML resource that references the source model\.  
Display name in the AWS IoT console: **SageMaker job ARN resource**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9:_-]+`

`MLModelSageMakerJobArn`  <a name="param-image-classification-mdljobarn"></a>
The ARN of the Amazon SageMaker training job that represents the Amazon SageMaker model source\. The model must be trained by the Amazon SageMaker image classification algorithm\.  
Display name in the AWS IoT console: **SageMaker job ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `^arn:aws:sagemaker:[a-zA-Z0-9-]+:[0-9]+:training-job/[a-zA-Z0-9][a-zA-Z0-9-]+$`

`LocalInferenceServiceName`  <a name="param-image-classification-svcname"></a>
The name for the local inference service\. User\-defined Lambda functions invoke the service by passing the name to the `invoke_inference_service` function of the AWS IoT Greengrass Machine Learning SDK\. For an example, see [Usage Example](#image-classification-connector-usage)\.  
Display name in the AWS IoT console: **Local inference service name**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9][a-zA-Z0-9-]{1,62}`

`LocalInferenceServiceTimeoutSeconds`  <a name="param-image-classification-svctimeout"></a>
The amount of time \(in seconds\) before the inference request is terminated\. The minimum value is 1\.  
Display name in the AWS IoT console: **Timeout \(second\)**  
Required: `true`  
Type: `string`  
Valid pattern: `[1-9][0-9]*`

`LocalInferenceServiceMemoryLimitKB`  <a name="param-image-classification-svcmemorylimit"></a>
The amount of memory \(in KB\) that the service has access to\. The minimum value is 1\.  
Display name in the AWS IoT console: **Memory limit \(KB\)**  
Required: `true`  
Type: `string`  
Valid pattern: `[1-9][0-9]*`

`GPUAcceleration`  <a name="param-image-classification-gpuacceleration"></a>
The CPU or GPU \(accelerated\) computing context\. This property applies to the ML Image Classification Aarch64 JTX2 connector only\.  
Display name in the AWS IoT console: **GPU acceleration**  
Required: `true`  
Type: `string`  
Valid values: `CPU` or `GPU`

`MLFeedbackConnectorConfigId`  <a name="param-image-classification-feedbackconfigid"></a>
The ID of the feedback configuration to use to upload model input data\. This must match the ID of a feedback configuration defined for the [ML Feedback connector](ml-feedback-connector.md)\.  
This parameter is required only if you want to use the ML Feedback connector to upload model input data and publish predictions to an MQTT topic\.  
Display name in the AWS IoT console: **ML Feedback connector configuration ID**  
Required: `false`  
Type: `string`  
Valid pattern: `^$|^[a-zA-Z0-9][a-zA-Z0-9-]{1,62}$`

------
#### [ Version 1 ]

`MLModelDestinationPath`  <a name="param-image-classification-mdlpath"></a>
The absolute local path of the ML resource inside the Lambda environment\. This is the destination path that's specified for the ML resource\.  
If you created the ML resource in the console, this is the local path\.
Display name in the AWS IoT console: **Model destination path**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`MLModelResourceId`  <a name="param-image-classification-mdlresourceid"></a>
The ID of the ML resource that references the source model\.  
Display name in the AWS IoT console: **SageMaker job ARN resource**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9:_-]+`

`MLModelSageMakerJobArn`  <a name="param-image-classification-mdljobarn"></a>
The ARN of the Amazon SageMaker training job that represents the Amazon SageMaker model source\. The model must be trained by the Amazon SageMaker image classification algorithm\.  
Display name in the AWS IoT console: **SageMaker job ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `^arn:aws:sagemaker:[a-zA-Z0-9-]+:[0-9]+:training-job/[a-zA-Z0-9][a-zA-Z0-9-]+$`

`LocalInferenceServiceName`  <a name="param-image-classification-svcname"></a>
The name for the local inference service\. User\-defined Lambda functions invoke the service by passing the name to the `invoke_inference_service` function of the AWS IoT Greengrass Machine Learning SDK\. For an example, see [Usage Example](#image-classification-connector-usage)\.  
Display name in the AWS IoT console: **Local inference service name**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9][a-zA-Z0-9-]{1,62}`

`LocalInferenceServiceTimeoutSeconds`  <a name="param-image-classification-svctimeout"></a>
The amount of time \(in seconds\) before the inference request is terminated\. The minimum value is 1\.  
Display name in the AWS IoT console: **Timeout \(second\)**  
Required: `true`  
Type: `string`  
Valid pattern: `[1-9][0-9]*`

`LocalInferenceServiceMemoryLimitKB`  <a name="param-image-classification-svcmemorylimit"></a>
The amount of memory \(in KB\) that the service has access to\. The minimum value is 1\.  
Display name in the AWS IoT console: **Memory limit \(KB\)**  
Required: `true`  
Type: `string`  
Valid pattern: `[1-9][0-9]*`

`GPUAcceleration`  <a name="param-image-classification-gpuacceleration"></a>
The CPU or GPU \(accelerated\) computing context\. This property applies to the ML Image Classification Aarch64 JTX2 connector only\.  
Display name in the AWS IoT console: **GPU acceleration**  
Required: `true`  
Type: `string`  
Valid values: `CPU` or `GPU`

------

### Create Connector Example \(AWS CLI\)<a name="image-classification-connector-create"></a>

The following CLI commands create a `ConnectorDefinition` with an initial version that contains an ML Image Classification connector\.

**Example: CPU Instance**  
This example creates an instance of the ML Image Classification Armv7l connector\.  

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyImageClassificationConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ImageClassificationARMv7/versions/2",
            "Parameters": {
                "MLModelDestinationPath": "/path-to-model",
                "MLModelResourceId": "my-ml-resource",
                "MLModelSageMakerJobArn": "arn:aws:sagemaker:us-west-2:123456789012:training-job:MyImageClassifier",
                "LocalInferenceServiceName": "imageClassification",
                "LocalInferenceServiceTimeoutSeconds": "10",
                "LocalInferenceServiceMemoryLimitKB": "500000",
                "MLFeedbackConnectorConfigId": "MyConfig0"
            }
        }
    ]
}'
```

**Example: GPU Instance**  
This example creates an instance of the ML Image Classification Aarch64 JTX2 connector, which supports GPU acceleration on an NVIDIA Jetson TX2 board\.  

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyImageClassificationConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ImageClassificationAarch64JTX2/versions/2",
            "Parameters": {
                "MLModelDestinationPath": "/path-to-model",
                "MLModelResourceId": "my-ml-resource",
                "MLModelSageMakerJobArn": "arn:aws:sagemaker:us-west-2:123456789012:training-job:MyImageClassifier",
                "LocalInferenceServiceName": "imageClassification",
                "LocalInferenceServiceTimeoutSeconds": "10",
                "LocalInferenceServiceMemoryLimitKB": "500000",
                "GPUAcceleration": "GPU",
                "MLFeedbackConnectorConfigId": "MyConfig0"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in these connectors have a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="image-classification-connector-data-input"></a>

 These connectors accept an image file as input\. Input image files must be in `jpeg` or `png` format\. For more information, see [Usage Example](#image-classification-connector-usage)\. 

These connectors don't accept MQTT messages as input data\.

## Output data<a name="image-classification-connector-data-output"></a>

These connectors return a formatted prediction for the object identified in the input image:

```
[0.3,0.1,0.04,...]
```

The prediction contains a list of values that correspond with the categories used in the training dataset during model training\. Each value represents the probability that the image falls under the corresponding category\. The category with the highest probability is the dominant prediction\.

These connectors don't publish MQTT messages as output data\.

## Usage Example<a name="image-classification-connector-usage"></a>

The following example Lambda function uses the [AWS IoT Greengrass Machine Learning SDK](lambda-functions.md#lambda-sdks-ml) to interact with an ML Image Classification connector\.

**Note**  
 You can download the SDK from the [AWS IoT Greengrass Machine Learning SDK](what-is-gg.md#gg-ml-sdk-download) downloads page\.

The example initializes an SDK client and synchronously calls the SDK's `invoke_inference_service` function to invoke the local inference service\. It passes in the algorithm type, service name, image type, and image content\. Then, the example parses the service response to get the probability results \(predictions\)\.

------
#### [ Python 3\.7 ]

```
import logging
from threading import Timer

import numpy as np

import greengrass_machine_learning_sdk as ml

# We assume the inference input image is provided as a local file
# to this inference client Lambda function.
with open('/test_img/test.jpg', 'rb') as f:
    content = bytearray(f.read())

client = ml.client('inference')

def infer():
    logging.info('invoking Greengrass ML Inference service')

    try:
        resp = client.invoke_inference_service(
            AlgoType='image-classification',
            ServiceName='imageClassification',
            ContentType='image/jpeg',
            Body=content
        )
    except ml.GreengrassInferenceException as e:
        logging.info('inference exception {}("{}")'.format(e.__class__.__name__, e))
        return
    except ml.GreengrassDependencyException as e:
        logging.info('dependency exception {}("{}")'.format(e.__class__.__name__, e))
        return

    logging.info('resp: {}'.format(resp))
    predictions = resp['Body'].read().decode("utf-8")
    logging.info('predictions: {}'.format(predictions))
    
    # The connector output is in the format: [0.3,0.1,0.04,...]
    # Remove the '[' and ']' at the beginning and end.
    predictions = predictions[1:-1]
    count = len(predictions.split(','))
    predictions_arr = np.fromstring(predictions, count=count, sep=',')

    # Perform business logic that relies on the predictions_arr, which is an array
    # of probabilities.
    
    # Schedule the infer() function to run again in one second.
    Timer(1, infer).start()
    return

infer()

def function_handler(event, context):
    return
```

------
#### [ Python 2\.7 ]

```
import logging
from threading import Timer

import numpy as np

import greengrass_machine_learning_sdk as ml

# We assume the inference input image is provided as a local file
# to this inference client Lambda function.
with open('/test_img/test.jpg', 'rb') as f:
    content = f.read()

client = ml.client('inference')

def infer():
    logging.info('invoking Greengrass ML Inference service')

    try:
        resp = client.invoke_inference_service(
            AlgoType='image-classification',
            ServiceName='imageClassification',
            ContentType='image/jpeg',
            Body=content
        )
    except ml.GreengrassInferenceException as e:
        logging.info('inference exception {}("{}")'.format(e.__class__.__name__, e))
        return
    except ml.GreengrassDependencyException as e:
        logging.info('dependency exception {}("{}")'.format(e.__class__.__name__, e))
        return

    logging.info('resp: {}'.format(resp))
    predictions = resp['Body'].read()
    logging.info('predictions: {}'.format(predictions))
    
    # The connector output is in the format: [0.3,0.1,0.04,...]
    # Remove the '[' and ']' at the beginning and end.
    predictions = predictions[1:-1]
    count = len(predictions.split(','))
    predictions_arr = np.fromstring(predictions, count=count, sep=',')

    # Perform business logic that relies on the predictions_arr, which is an array
    # of probabilities.
    
    # Schedule the infer() function to run again in one second.
    Timer(1, infer).start()
    return

infer()

def function_handler(event, context):
    return
```

------

The `invoke_inference_service` function in the AWS IoT Greengrass Machine Learning SDK accepts the following arguments\.


| Argument | Description | 
| --- | --- | 
| `AlgoType` | The name of the algorithm type to use for inference\. Currently, only `image-classification` is supported\. Required: `true` Type: `string` Valid values: `image-classification` | 
| `ServiceName` | The name of the local inference service\. Use the name that you specified for the `LocalInferenceServiceName` parameter when you configured the connector\. Required: `true` Type: `string` | 
| `ContentType` | The mime type of the input image\. Required: `true` Type: `string` Valid values: `image/jpeg, image/png` | 
| `Body` | The content of the input image file\. Required: `true` Type: `binary` | 

## Installing MXNet dependencies on the AWS IoT Greengrass core<a name="image-classification-connector-config"></a>

To use an ML Image Classification connector, you must install the dependencies for the Apache MXNet framework on the core device\. The connectors use the framework to serve the ML model\.

**Note**  
These connectors are bundled with a precompiled MXNet library, so you don't need to install the MXNet framework on the core device\. 

AWS IoT Greengrass provides scripts to install the dependencies for the following common platforms and devices \(or to use as a reference for installing them\)\. If you're using a different platform or device, see the [MXNet documentation](https://mxnet.apache.org/) for your configuration\.

Before installing the MXNet dependencies, make sure that the required [system libraries](#image-classification-connector-logging) \(with the specified minimum versions\) are present on the device\.

------
#### [ NVIDIA Jetson TX2 ]

1. Install CUDA Toolkit 9\.0 and cuDNN 7\.0\. You can follow the instructions in [Setting up other devices](setup-filter.other.md) in the Getting Started tutorial\.

1. Enable universe repositories so the connector can install community\-maintained open software\. For more information, see [ Repositories/Ubuntu](https://help.ubuntu.com/community/Repositories/Ubuntu) in the Ubuntu documentation\.

   1. Open the `/etc/apt/sources.list` file\.

   1. Make sure that the following lines are uncommented\.

      ```
      deb http://ports.ubuntu.com/ubuntu-ports/ xenial universe
      deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial universe
      deb http://ports.ubuntu.com/ubuntu-ports/ xenial-updates universe
      deb-src http://ports.ubuntu.com/ubuntu-ports/ xenial-updates universe
      ```

1. Save a copy of the following installation script to a file named `nvidiajtx2.sh` on the core device\.

------
#### [ Python 3\.7 ]

   ```
   #!/bin/bash
   set -e
   
   echo "Installing dependencies on the system..."
   echo 'Assuming that universe repos are enabled and checking dependencies...'
   apt-get -y update
   apt-get -y dist-upgrade
   apt-get install -y liblapack3 libopenblas-dev liblapack-dev libatlas-base-dev
   apt-get install -y python3.7 python3.7-dev
   
   python3.7 -m pip install --upgrade pip
   python3.7 -m pip install numpy==1.15.0
   python3.7 -m pip install opencv-python || echo 'Error: Unable to install OpenCV with pip on this platform. Try building the latest OpenCV from source (https://github.com/opencv/opencv).'
   
   echo 'Dependency installation/upgrade complete.'
   ```

**Note**  
<a name="opencv-build-from-source"></a>If [OpenCV](https://github.com/opencv/opencv) does not install successfully using this script, you can try building from source\. For more information, see [ Installation in Linux](https://docs.opencv.org/4.1.0/d7/d9f/tutorial_linux_install.html) in the OpenCV documentation, or refer to other online resources for your platform\.

------
#### [ Python 2\.7 ]

   ```
   #!/bin/bash
   set -e
   
   echo "Installing dependencies on the system..."
   echo 'Assuming that universe repos are enabled and checking dependencies...'
   apt-get -y update
   apt-get -y dist-upgrade
   apt-get install -y liblapack3 libopenblas-dev liblapack-dev libatlas-base-dev python-dev
   
   echo 'Install latest pip...'
   wget https://bootstrap.pypa.io/get-pip.py
   python get-pip.py
   rm get-pip.py
   
   pip install numpy==1.15.0 scipy
   
   echo 'Dependency installation/upgrade complete.'
   ```

------

1. From the directory where you saved the file, run the following command:

   ```
   sudo nvidiajtx2.sh
   ```

------
#### [ x86\_64 \(Ubuntu or Amazon Linux\)  ]

1. Save a copy of the following installation script to a file named `x86_64.sh` on the core device\.

------
#### [ Python 3\.7 ]

   ```
   #!/bin/bash
   set -e
   
   echo "Installing dependencies on the system..."
   
   release=$(awk -F= '/^NAME/{print $2}' /etc/os-release)
   
   if [ "$release" == '"Ubuntu"' ]; then
     # Ubuntu. Supports EC2 and DeepLens. DeepLens has all the dependencies installed, so
     # this is mostly to prepare dependencies on Ubuntu EC2 instance.
     apt-get -y update
     apt-get -y dist-upgrade
   
     apt-get install -y libgfortran3 libsm6 libxext6 libxrender1
     apt-get install -y python3.7 python3.7-dev
   elif [ "$release" == '"Amazon Linux"' ]; then
     # Amazon Linux. Expect python to be installed already
     yum -y update
     yum -y upgrade
   
     yum install -y compat-gcc-48-libgfortran libSM libXrender libXext
   else
     echo "OS Release not supported: $release"
     exit 1
   fi
   
   python3.7 -m pip install --upgrade pip
   python3.7 -m pip install numpy==1.15.0
   python3.7 -m pip install opencv-python || echo 'Error: Unable to install OpenCV with pip on this platform. Try building the latest OpenCV from source (https://github.com/opencv/opencv).'
   
   echo 'Dependency installation/upgrade complete.'
   ```

**Note**  
<a name="opencv-build-from-source"></a>If [OpenCV](https://github.com/opencv/opencv) does not install successfully using this script, you can try building from source\. For more information, see [ Installation in Linux](https://docs.opencv.org/4.1.0/d7/d9f/tutorial_linux_install.html) in the OpenCV documentation, or refer to other online resources for your platform\.

------
#### [ Python 2\.7 ]

   ```
   #!/bin/bash
   set -e
   
   echo "Installing dependencies on the system..."
   
   release=$(awk -F= '/^NAME/{print $2}' /etc/os-release)
   
   if [ "$release" == '"Ubuntu"' ]; then
     # Ubuntu. Supports EC2 and DeepLens. DeepLens has all the dependencies installed, so
     # this is mostly to prepare dependencies on Ubuntu EC2 instance.
     apt-get -y update
     apt-get -y dist-upgrade
   
     apt-get install -y libgfortran3 libsm6 libxext6 libxrender1 python-dev python-pip
   elif [ "$release" == '"Amazon Linux"' ]; then
     # Amazon Linux. Expect python to be installed already
     yum -y update
     yum -y upgrade
   
     yum install -y compat-gcc-48-libgfortran libSM libXrender libXext python-pip
   else
     echo "OS Release not supported: $release"
     exit 1
   fi
   
   pip install numpy==1.15.0 scipy opencv-python
   
   echo 'Dependency installation/upgrade complete.'
   ```

------

1. From the directory where you saved the file, run the following command:

   ```
   sudo x86_64.sh
   ```

------
#### [ Armv7 \(Raspberry Pi\) ]

1. Save a copy of the following installation script to a file named `armv7l.sh` on the core device\.

------
#### [ Python 3\.7 ]

   ```
   #!/bin/bash
   set -e
   
   echo "Installing dependencies on the system..."
   
   apt-get update
   apt-get -y upgrade
   
   apt-get install -y liblapack3 libopenblas-dev liblapack-dev
   apt-get install -y python3.7 python3.7-dev
   
   python3.7 -m pip install --upgrade pip
   python3.7 -m pip install numpy==1.15.0
   python3.7 -m pip install opencv-python || echo 'Error: Unable to install OpenCV with pip on this platform. Try building the latest OpenCV from source (https://github.com/opencv/opencv).'
   
   echo 'Dependency installation/upgrade complete.'
   ```

**Note**  
<a name="opencv-build-from-source"></a>If [OpenCV](https://github.com/opencv/opencv) does not install successfully using this script, you can try building from source\. For more information, see [ Installation in Linux](https://docs.opencv.org/4.1.0/d7/d9f/tutorial_linux_install.html) in the OpenCV documentation, or refer to other online resources for your platform\.

------
#### [ Python 2\.7 ]

   ```
   #!/bin/bash
   set -e
   
   echo "Installing dependencies on the system..."
   
   apt-get update
   apt-get -y upgrade
   
   apt-get install -y liblapack3 libopenblas-dev liblapack-dev python-dev
   
   # python-opencv depends on python-numpy. The latest version in the APT repository is python-numpy-1.8.2
   # This script installs python-numpy first so that python-opencv can be installed, and then install the latest
   # numpy-1.15.x with pip
   apt-get install -y python-numpy python-opencv
   dpkg --remove --force-depends python-numpy
   
   echo 'Install latest pip...'
   wget https://bootstrap.pypa.io/get-pip.py
   python get-pip.py
   rm get-pip.py
   
   pip install --upgrade numpy==1.15.0 picamera scipy
   
   echo 'Dependency installation/upgrade complete.'
   ```

------

1. From the directory where you saved the file, run the following command:

   ```
   sudo bash armv7l.sh
   ```
**Note**  
On a Raspberry Pi, using `pip` to install machine learning dependencies is a memory\-intensive operation that can cause the device to run out of memory and become unresponsive\. As a workaround, you can temporarily increase the swap size:  
In `/etc/dphys-swapfile`, increase the value of the `CONF_SWAPSIZE` variable and then run the following command to restart `dphys-swapfile`\.  

   ```
   /etc/init.d/dphys-swapfile restart
   ```

------

## Logging and troubleshooting<a name="image-classification-connector-logging"></a>

Depending on your group settings, event and error logs are written to CloudWatch Logs, the local file system, or both\. Logs from this connector use the prefix `LocalInferenceServiceName`\. If the connector behaves unexpectedly, check the connector's logs\. These usually contain useful debugging information, such as a missing ML library dependency or the cause of a connector startup failure\.

If the AWS IoT Greengrass group is configured to write local logs, the connector writes log files to `greengrass-root/ggc/var/log/user/region/aws/`\. For more information about Greengrass logging, see [Monitoring with AWS IoT Greengrass logs](greengrass-logs-overview.md)\.

Use the following information to help troubleshoot issues with the ML Image Classification connectors\.

**Required system libraries**

The following tabs list the system libraries required for each ML Image Classification connector\.

------
#### [ ML Image Classification Aarch64 JTX2 ]


| Library | Minimum version | 
| --- | --- | 
| ld\-linux\-aarch64\.so\.1 | GLIBC\_2\.17 | 
| libc\.so\.6 | GLIBC\_2\.17 | 
| libcublas\.so\.9\.0 | not applicable | 
| libcudart\.so\.9\.0 | not applicable | 
| libcudnn\.so\.7 | not applicable | 
| libcufft\.so\.9\.0 | not applicable | 
| libcurand\.so\.9\.0 | not applicable | 
| libcusolver\.so\.9\.0 | not applicable | 
| libgcc\_s\.so\.1 | GCC\_4\.2\.0 | 
| libgomp\.so\.1 | GOMP\_4\.0, OMP\_1\.0 | 
| libm\.so\.6 | GLIBC\_2\.23 | 
| libpthread\.so\.0 | GLIBC\_2\.17 | 
| librt\.so\.1 | GLIBC\_2\.17 | 
| libstdc\+\+\.so\.6 | GLIBCXX\_3\.4\.21, CXXABI\_1\.3\.8 | 

------
#### [ ML Image Classification x86\_64 ]


| Library | Minimum version | 
| --- | --- | 
| ld\-linux\-x86\-64\.so\.2 | GCC\_4\.0\.0 | 
| libc\.so\.6 | GLIBC\_2\.4 | 
| libgfortran\.so\.3 | GFORTRAN\_1\.0 | 
| libm\.so\.6 | GLIBC\_2\.23 | 
| libpthread\.so\.0 | GLIBC\_2\.2\.5 | 
| librt\.so\.1 | GLIBC\_2\.2\.5 | 
| libstdc\+\+\.so\.6 | CXXABI\_1\.3\.8, GLIBCXX\_3\.4\.21 | 

------
#### [ ML Image Classification Armv7 ]


| Library | Minimum version | 
| --- | --- | 
| ld\-linux\-armhf\.so\.3 | GLIBC\_2\.4 | 
| libc\.so\.6 | GLIBC\_2\.7 | 
| libgcc\_s\.so\.1 | GCC\_4\.0\.0 | 
| libgfortran\.so\.3 | GFORTRAN\_1\.0 | 
| libm\.so\.6 | GLIBC\_2\.4 | 
| libpthread\.so\.0 | GLIBC\_2\.4 | 
| librt\.so\.1 | GLIBC\_2\.4 | 
| libstdc\+\+\.so\.6 | CXXABI\_1\.3\.8, CXXABI\_ARM\_1\.3\.3, GLIBCXX\_3\.4\.20 | 

------

**Issues**


| Symptom | Solution | 
| --- | --- | 
|  On a Raspberry Pi, the following error message is logged and you are not using the camera: `Failed to initialize libdc1394`   |  Run the following command to disable the driver: <pre>sudo ln /dev/null /dev/raw1394</pre> This operation is ephemeral and the symbolic link will disappear after rebooting\. Consult the manual of your OS distribution to learn how to automatically create the link up on reboot\.  | 

## Licenses<a name="image-classification-connector-license"></a>

The ML Image Classification connectors includes the following third\-party software/licensing:<a name="boto-3-licenses"></a>
+ [AWS SDK for Python \(Boto3\)](https://pypi.org/project/boto3/)/Apache License 2\.0
+ [botocore](https://pypi.org/project/botocore/)/Apache License 2\.0
+ [dateutil](https://pypi.org/project/python-dateutil/1.4/)/PSF License
+ [docutils](https://pypi.org/project/docutils/)/BSD License, GNU General Public License \(GPL\), Python Software Foundation License, Public Domain
+ [jmespath](https://pypi.org/project/jmespath/)/MIT License
+ [s3transfer](https://pypi.org/project/s3transfer/)/Apache License 2\.0
+ [urllib3](https://pypi.org/project/urllib3/)/MIT License
+ [Deep Neural Network Library \(DNNL\)](https://github.com/intel/mkl-dnn)/Apache License 2\.0
+ [OpenMP\* Runtime Library](https://www.openmprtl.org/)/See [Intel OpenMP Runtime Library licensing](#openmp-license)\.
+ [mxnet](https://pypi.org/project/mxnet/)/Apache License 2\.0
+ <a name="six-license"></a>[six](https://github.com/benjaminp/six)/MIT

**Intel OpenMP Runtime Library licensing**\. The Intel® OpenMP\* runtime is dual\-licensed, with a commercial \(COM\) license as part of the Intel® Parallel Studio XE Suite products, and a BSD open source \(OSS\) license\. For more information, see [Licensing](https://www.openmprtl.org/faq/10) in the Intel® OpenMP\* Runtime Library documentation\.

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## Changelog<a name="image-classification-connector-changelog"></a>

The following table describes the changes in each version of the connector\.


| Version | Changes | 
| --- | --- | 
| 2 | Added the `MLFeedbackConnectorConfigId` parameter to support the use of the [ML Feedback connector](ml-feedback-connector.md) to upload model input data, publish predictions to an MQTT topic, and publish metrics to Amazon CloudWatch\.  | 
| 1 | Initial release\.  | 

<a name="one-conn-version"></a>A Greengrass group can contain only one version of the connector at a time\. For information about upgrading a connector version, see [Upgrading connector versions](connectors.md#upgrade-connector-versions)\.

## See also<a name="image-classification-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)
+ [Perform machine learning inference](ml-inference.md)
+ [Image classification algorithm](https://docs.aws.amazon.com/sagemaker/latest/dg/image-classification.html) in the *Amazon SageMaker Developer Guide*