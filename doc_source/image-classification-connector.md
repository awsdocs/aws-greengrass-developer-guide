# Image Classification<a name="image-classification-connector"></a>

The Image Classification [connectors](connectors.md) provide a machine learning \(ML\) inference service that runs on the AWS IoT Greengrass core\. This local inference service performs image classification using a model trained by the Amazon SageMaker image classification algorithm\.

User\-defined Lambda functions use the AWS IoT Greengrass Machine Learning SDK to submit inference requests to the local inference service\. The service runs inference locally and returns probabilities that the input image belongs to specific categories\.

AWS IoT Greengrass provides Image Classification connectors for multiple platforms:


| Connector | Description and ARN | 
| --- | --- | 
| Image Classification Aarch64 JTX2 |  Image classification inference service for NVIDIA Jetson TX2\. Supports GPU acceleration\. **ARN:** `arn:aws:greengrass:region::/connectors/ImageClassificationAarch64JTX2/versions/1` | 
| Image Classification x86\_64 |  Image classification inference service for x86\_64 platforms\. **ARN:** `arn:aws:greengrass:region::/connectors/ImageClassificationx86-64/versions/1` | 
| Image Classification ARMv7 |  Image classification inference service for ARMv7 platforms\. **ARN:** `arn:aws:greengrass:region::/connectors/ImageClassificationARMv7/versions/1` | 

## Requirements<a name="image-classification-connector-req"></a>

These connectors have the following requirements:
+ AWS IoT Greengrass Core Software v1\.7 or later\.
+ [Python](https://www.python.org/) version 2\.7 installed on the core device and added to the PATH environment variable\.
+ Dependencies for the Apache MXNet framework installed on the core device\. For more information, see [Installing MXNet Dependencies on the AWS IoT Greengrass Core](#image-classification-connector-config)\.
+ An [ML resource](ml-inference.md#ml-resources) in the Greengrass group that references an Amazon SageMaker model source\. This model must be trained by the Amazon SageMaker image classification algorithm\. For more information, see [Image Classification Algorithm](https://docs.aws.amazon.com/sagemaker/latest/dg/image-classification.html) in the *Amazon SageMaker Developer Guide*\.
+ An IAM policy added to the Greengrass group role that allows the `sagemaker:DescribeTrainingJob` action on the target training job, as shown in the following example\.

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

  You can grant granular or conditional access to resources \(for example, by using a wildcard \* naming scheme\)\. If you change the target training job in the future, make sure to update the group role\. For more information, see [Adding and Removing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

## Connector Parameters<a name="image-classification-connector-param"></a>

These connectors provide the following parameters\.

`MLModelDestinationPath`  
The absolute local path of the ML resource inside the Lambda environment\. This is the destination path that's specified for the ML resource\.  
If you created the ML resource in the console, this is the local path\.
Display name in console: **Model destination path**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`MLModelResourceId`  
The ID of the ML resource that references the source model\.  
Display name in console: **SageMaker job ARN resource**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9:_-]+`

`MLModelSageMakerJobArn`  
The ARN of the Amazon SageMaker training job that represents the Amazon SageMaker model source\. The model must be trained by the Amazon SageMaker image classification algorithm\.  
Display name in console: **SageMaker job ARN**  
Required: `true`  
Type: `string`  
Valid pattern: `^arn:aws:sagemaker:[a-zA-Z0-9-]+:[0-9]+:training-job/[a-zA-Z0-9][a-zA-Z0-9-]+$`

`LocalInferenceServiceName`  
The name for the local inference service\. User\-defined Lambda functions invoke the service by passing the name to the `invoke_inference_service` function of the AWS IoT Greengrass Machine Learning SDK\. For an example, see [Usage Example](#image-classification-connector-usage)\.  
Display name in console: **Local inference service name**  
Required: `true`  
Type: `string`  
Valid pattern: `[a-zA-Z0-9][a-zA-Z0-9-]{1,62}`

`LocalInferenceServiceTimeoutSeconds`  
The amount of time \(in seconds\) before the inference request is terminated\. The minimum value is 1\.  
Display name in console: **Timeout \(second\)**  
Required: `true`  
Type: `string`  
Valid pattern: `[1-9][0-9]*`

`LocalInferenceServiceMemoryLimitKB`  
The amount of memory \(in KB\) that the service has access to\. The minimum value is 1\.  
Display name in console: **Memory limit \(KB\)**  
Required: `true`  
Type: `string`  
Valid pattern: `[1-9][0-9]*`

`GPUAcceleration`  
The CPU or GPU \(accelerated\) computing context\. This property applies to the Image Classification Aarch64 JTX2 connector only\.  
Display name in console: **GPU acceleration**  
Required: `true`  
Type: `string`  
Valid values: `CPU` or `GPU`

### Create Connector Example \(CLI\)<a name="image-classification-connector-create"></a>

The following CLI commands create a `ConnectorDefinition` with an initial version that contains an Image Classification connector\.

**Example: CPU Instance**  
This example creates an instance of the Image Classification ARMv7l connector\.  

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyImageClassificationConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ImageClassificationARMv7/versions/1",
            "Parameters": {
                "MLModelDestinationPath": "/path-to-model",
                "MLModelResourceId": "my-ml-resource",
                "MLModelSageMakerJobArn": "arn:aws:sagemaker:us-west-2:123456789012:training-job:MyImageClassifier",
                "LocalInferenceServiceName": "imageClassification",
                "LocalInferenceServiceTimeoutSeconds": "10",
                "LocalInferenceServiceMemoryLimitKB": "500000"
            }
        }
    ]
}'
```

**Example: GPU Instance**  
This example creates an instance of the Image Classification Aarch64 JTX2 connector, which supports GPU acceleration on an NVIDIA Jetson TX2 board\.  

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyImageClassificationConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ImageClassificationAarch64JTX2/versions/1",
            "Parameters": {
                "MLModelDestinationPath": "/path-to-model",
                "MLModelResourceId": "my-ml-resource",
                "MLModelSageMakerJobArn": "arn:aws:sagemaker:us-west-2:123456789012:training-job:MyImageClassifier",
                "LocalInferenceServiceName": "imageClassification",
                "LocalInferenceServiceTimeoutSeconds": "10",
                "LocalInferenceServiceMemoryLimitKB": "500000",
                "GPUAcceleration": "GPU"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in these connectors have a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)\.

## Input Data<a name="image-classification-connector-data-input"></a>

These connectors don't accept MQTT messages as input data\.

## Output Data<a name="image-classification-connector-data-output"></a>

These connectors don't publish MQTT messages as output data\.

## Usage Example<a name="image-classification-connector-usage"></a>

The following example Lambda function uses the [AWS IoT Greengrass Machine Learning SDK](lambda-functions.md#lambda-sdks-ml) to interact with an Image Classification connector\.

**Note**  
 You can download the SDK from the [AWS IoT Greengrass Machine Learning SDK](what-is-gg.md#gg-ml-sdk-download) downloads page\.

The example initializes an SDK client and synchronously calls the SDK's `invoke_inference_service` function to invoke the local inference service\. It passes in the algorithm type, service name, image type, and image content\. Then, the example parses the service response to get the probability results \(predictions\)\.

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

The `invoke_inference_service` function in the AWS IoT Greengrass Machine Learning SDK accepts the following arguments\.


| Argument | Description | 
| --- | --- | 
| `AlgoType` | The name of the algorithm type to use for inference\. Currently, only `image-classification` is supported\. Required: `true` Type: `string` Valid values: `image-classification` | 
| `ServiceName` | The name of the local inference service\. Use the name that you specified for the `LocalInferenceServiceName` parameter when you configured the connector\. Required: `true` Type: `string` | 
| `ContentType` | The mime type of the input image\. Required: `true` Type: `string` Valid values: `image/jpeg, image/png` | 
| `Body` | The content of the input image file\. Required: `true` Type: `binary` | 

## Installing MXNet Dependencies on the AWS IoT Greengrass Core<a name="image-classification-connector-config"></a>

To use an Image Classification connector, you must install the dependencies for the Apache MXNet framework on the core device\. The connectors use the framework to serve the ML model\.

**Note**  
These connectors are bundled with a precompiled MXNet library, so you don't need to install the MXNet framework on the core device\. 

AWS IoT Greengrass provides scripts to install the dependencies for the following common platforms and devices \(or to use as a reference for installing them\)\. If you're using a different platform or device, see the [MXNet documentation](https://mxnet.apache.org/) for your configuration\.

Before installing the MXNet dependencies, make sure that the required [system libraries](#image-classification-connector-logging) \(with the specified minimum versions\) are present on the device\.

------
#### [ NVIDIA Jetson TX2 ]

1. Install CUDA Toolkit 9\.0 and cuDNN 7\.0\. You can follow the instructions in [Setting Up Other Devices](module1.md#setup-filter.other) in the Getting Started tutorial\.

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

   ```
   #!/bin/bash
   set -e
   
   echo "Installing MXNet dependencies on the system..."
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

1. From the directory where you saved the file, run the following command:

   ```
   sudo nvidiajtx2.sh
   ```

------
#### [ x86\_64 \(Ubuntu or Amazon Linux\)  ]

1. Save a copy of the following installation script to a file named `x86_64.sh` on the core device\.

   ```
   #!/bin/bash
   set -e
   
   echo "Installing MXNet dependencies on the system..."
   
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

1. From the directory where you saved the file, run the following command:

   ```
   sudo x86_64.sh
   ```

------
#### [ ARMv7 \(Raspberry Pi\) ]

1. Save a copy of the following installation script to a file named `armv7l.sh` on the core device\.

   ```
   #!/bin/bash
   set -e
   
   echo "Installing MXNet dependencies on the system..."
   
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

1. From the directory where you saved the file, run the following command:

   ```
   sudo bash armv7l.sh
   ```
**Note**  
On a Raspberry Pi, installing `scipy` using `pip` is a memory\-intensive operation that can cause the device to run out of memory and become unresponsive\. As a workaround, you can temporarily increase the swap size:  
In `/etc/dphys-swapfile`, increase the value of the `CONF_SWAPSIZE` variable and then run the following command to restart `dphys-swapfile`\.  

   ```
   /etc/init.d/dphys-swapfile restart
   ```

------

## Logging and Troubleshooting<a name="image-classification-connector-logging"></a>

Depending on your group settings, event and error logs are written to CloudWatch Logs, the local file system, or both\. Logs from this connector use the prefix `LocalInferenceServiceName`\. If the connector behaves unexpectedly, check the connector's logs\. These usually contain useful debugging information, such as a missing ML library dependency or the cause of a connector startup failure\.

If the AWS IoT Greengrass group is configured to write local logs, the connector writes log files to `greengrass-root/ggc/var/log/user/region/aws/`\. For more information about Greengrass logging, see [Monitoring with AWS IoT Greengrass Logs](greengrass-logs-overview.md)\.

Use the following information to help troubleshoot issues with the Image Classification connectors\.

**Required system libraries**

The following tabs list the system libraries required for each Image Classification connector\.

------
#### [ Image Classification Aarch64 JTX2 ]


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
#### [ Image Classification x86\_64 ]


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
#### [ Image Classification ARMv7 ]


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

The Image Classification connectors includes the following third\-party software/licensing:
+ [AWS SDK for Python \(Boto 3\)](https://github.com/boto/boto3) / Apache 2\.0
+ [mxnet](https://pypi.org/project/mxnet/) / Apache 2\.0

This connector is released under the [Greengrass Core Software License Agreement](https://s3-us-west-2.amazonaws.com/greengrass-release-license/greengrass-license-v1.pdf)\.

## See Also<a name="image-classification-connector-see-also"></a>
+ [Integrate with Services and Protocols Using Greengrass Connectors](connectors.md)
+ [Getting Started with Greengrass Connectors \(Console\)](connectors-console.md)
+ [Getting Started with Greengrass Connectors \(CLI\)](connectors-cli.md)
+ [Perform Machine Learning Inference](ml-inference.md)
+ [Image Classification Algorithm](https://docs.aws.amazon.com/sagemaker/latest/dg/image-classification.html) in the *Amazon SageMaker Developer Guide*