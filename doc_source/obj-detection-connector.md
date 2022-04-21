--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# ML Object Detection connector<a name="obj-detection-connector"></a>

**Warning**  <a name="connectors-extended-life-phase-warning"></a>
This connector has moved into the *extended life phase*, and AWS IoT Greengrass won't release updates that provide features, enhancements to existing features, security patches, or bug fixes\. For more information, see [AWS IoT Greengrass Version 1 maintenance policy](maintenance-policy.md)\.

The ML Object Detection [connectors](connectors.md) provide a machine learning \(ML\) inference service that runs on the AWS IoT Greengrass core\. This local inference service performs object detection using an object detection model compiled by the SageMaker Neo deep learning compiler\. Two types of object detection models are supported: Single Shot Multibox Detector \(SSD\) and You Only Look Once \(YOLO\) v3\. For more information, see [Object Detection Model Requirements](#obj-detection-connector-req-model)\.

 User\-defined Lambda functions use the AWS IoT Greengrass Machine Learning SDK to submit inference requests to the local inference service\. The service performs local inference on an input image and returns a list of predictions for each object detected in the image\. Each prediction contains an object category, a prediction confidence score, and pixel coordinates that specify a bounding box around the predicted object\. 

AWS IoT Greengrass provides ML Object Detection connectors for multiple platforms:


| Connector | Description and ARN | 
| --- | --- | 
| ML Object Detection Aarch64 JTX2 |  Object detection inference service for NVIDIA Jetson TX2\. Supports GPU acceleration\.  **ARN:** `arn:aws:greengrass:region::/connectors/ObjectDetectionAarch64JTX2/versions/1`   | 
| ML Object Detection x86\_64 |  Object detection inference service for x86\_64 platforms\.  **ARN:** `arn:aws:greengrass:region::/connectors/ObjectDetectionx86-64/versions/1`   | 
| ML Object Detection ARMv7 |   Object detection inference service for ARMv7 platforms\.   **ARN:** `arn:aws:greengrass:region::/connectors/ObjectDetectionARMv7/versions/1`   | 

## Requirements<a name="obj-detection-connector-req"></a>

These connectors have the following requirements:
+ AWS IoT Greengrass Core Software v1\.9\.3 or later\.
+ <a name="conn-req-py-3.7-and-3.8"></a>[Python](https://www.python.org/) version 3\.7 or 3\.8 installed on the core device and added to the PATH environment variable\.
**Note**  <a name="use-runtime-py3.8"></a>
To use Python 3\.8, run the following command to create a symbolic link from the the default Python 3\.7 installation folder to the installed Python 3\.8 binaries\.  

  ```
  sudo ln -s path-to-python-3.8/python3.8 /usr/bin/python3.7
  ```
This configures your device to meet the Python requirement for AWS IoT Greengrass\.
+ Dependencies for the SageMaker Neo deep learning runtime installed on the core device\. For more information, see [Installing Neo deep learning runtime dependencies on the AWS IoT Greengrass core](#obj-detection-connector-config)\.
+ An [ML resource](ml-inference.md#ml-resources) in the Greengrass group\. The ML resource must reference an Amazon S3 bucket that contains an object detection model\. For more information, see [Amazon S3 model sources](ml-inference.md#s3-ml-resources)\.
**Note**  
The model must be a Single Shot Multibox Detector or You Only Look Once v3 object detection model type\. It must be compiled using the SageMaker Neo deep learning compiler\. For more information, see [Object Detection Model Requirements](#obj-detection-connector-req-model)\.
+ <a name="req-image-classification-feedback"></a>The [ML Feedback connector](ml-feedback-connector.md) added to the Greengrass group and configured\. This is required only if you want to use the connector to upload model input data and publish predictions to an MQTT topic\.
+ [AWS IoT Greengrass Machine Learning SDK](lambda-functions.md#lambda-sdks-ml) v1\.1\.0 is required to interact with this connector\.

### Object detection model requirements<a name="obj-detection-connector-req-model"></a>

The ML Object Detection connectors support Single Shot multibox Detector \(SSD\) and You Only Look Once \(YOLO\) v3 object detection model types\. You can use the object detection components provided by [GluonCV](https://gluon-cv.mxnet.io) to train the model with your own dataset\. Or, you can use pre\-trained models from the GluonCV Model Zoo:
+ [Pre\-trained SSD model](https://gluon-cv.mxnet.io/build/examples_detection/demo_ssd.html)
+ [Pre\-trained YOLO v3 model](https://gluon-cv.mxnet.io/build/examples_detection/demo_yolo.html)

Your object detection model must be trained with 512 x 512 input images\. The pre\-trained models from the GluonCV Model Zoo already meet this requirement\.

Trained object detection models must be compiled with the SageMaker Neo deep learning compiler\. When compiling, make sure the target hardware matches the hardware of your Greengrass core device\. For more information, see [ SageMaker Neo](https://docs.aws.amazon.com/sagemaker/latest/dg/neo.html) in the *Amazon SageMaker Developer Guide*\.

The compiled model must be added as an ML resource \([Amazon S3 model source](ml-inference.md#s3-ml-resources)\) to the same Greengrass group as the connector\.

## Connector Parameters<a name="obj-detection-connector-param"></a>

These connectors provide the following parameters\.

`MLModelDestinationPath`  
The absolute path to the the Amazon S3 bucket that contains the Neo\-compatible ML model\. This is the destination path that's specified for the ML model resource\.  
Display name in the AWS IoT console: **Model destination path**  
Required: `true`  
Type: `string`  
Valid pattern: `.+`

`MLModelResourceId`  
The ID of the ML resource that references the source model\.  
Display name in the AWS IoT console: **Greengrass group ML resource**  
Required: `true`  
Type: `S3MachineLearningModelResource`  
Valid pattern: `^[a-zA-Z0-9:_-]+$`

`LocalInferenceServiceName`  
The name for the local inference service\. User\-defined Lambda functions invoke the service by passing the name to the `invoke_inference_service` function of the AWS IoT Greengrass Machine Learning SDK\. For an example, see [Usage Example](#obj-detection-connector-usage)\.  
Display name in the AWS IoT console: **Local inference service name**  
Required: `true`  
Type: `string`  
Valid pattern: `^[a-zA-Z0-9][a-zA-Z0-9-]{1,62}$`

`LocalInferenceServiceTimeoutSeconds`  
The time \(in seconds\) before the inference request is terminated\. The minimum value is 1\. The default value is 10\.  
Display name in the AWS IoT console: **Timeout \(second\)**  
Required: `true`  
Type: `string`  
Valid pattern: `^[1-9][0-9]*$`

`LocalInferenceServiceMemoryLimitKB`  
The amount of memory \(in KB\) that the service has access to\. The minimum value is 1\.  
Display name in the AWS IoT console: **Memory limit**  
Required: `true`  
Type: `string`  
Valid pattern: `^[1-9][0-9]*$`

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

### Create Connector Example \(AWS CLI\)<a name="obj-detection-connector-create"></a>

The following CLI command creates a `ConnectorDefinition` with an initial version that contains an ML Object Detection connector\. This example creates an instance of the ML Object Detection ARMv7l connector\.

```
aws greengrass create-connector-definition --name MyGreengrassConnectors --initial-version '{
    "Connectors": [
        {
            "Id": "MyObjectDetectionConnector",
            "ConnectorArn": "arn:aws:greengrass:region::/connectors/ObjectDetectionARMv7/versions/1",
            "Parameters": {
                "MLModelDestinationPath": "/path-to-model",
                "MLModelResourceId": "my-ml-resource",
                "LocalInferenceServiceName": "objectDetection",
                "LocalInferenceServiceTimeoutSeconds": "10",
                "LocalInferenceServiceMemoryLimitKB": "500000",
                "MLFeedbackConnectorConfigId" : "object-detector-random-sampling"
            }
        }
    ]
}'
```

**Note**  
The Lambda function in these connectors have a [long\-lived](lambda-functions.md#lambda-lifecycle) lifecycle\.

In the AWS IoT Greengrass console, you can add a connector from the group's **Connectors** page\. For more information, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

## Input data<a name="obj-detection-connector-data-input"></a>

 These connectors accept an image file as input\. Input image files must be in `jpeg` or `png` format\. For more information, see [Usage Example](#obj-detection-connector-usage)\. 

These connectors don't accept MQTT messages as input data\.

## Output data<a name="obj-detection-connector-data-output"></a>

 These connectors return a formatted list of prediction results for the identified objects in the input image: 

```
     {
         "prediction": [
             [
                 14,
                 0.9384938478469849,
                 0.37763649225234985,
                 0.5110225081443787,
                 0.6697432398796082,
                 0.8544386029243469
             ],
             [
                 14,
                 0.8859519958496094,
                 0,
                 0.43536216020584106,
                 0.3314110040664673,
                 0.9538808465003967
             ],
             [
                 12,
                 0.04128098487854004,
                 0.5976729989051819,
                 0.5747185945510864,
                 0.704264223575592,
                 0.857937216758728
             ],
             ...
         ]
     }
```

Each prediction in the list is contained in square brackets and contains six values:
+  The first value represents the predicted object category for the identified object\. Object categories and their corresponding values are determined when training your object detection machine learning model in the Neo deep learning compiler\.
+ The second value is the confidence score for the object category prediction\. This represents the probability that the prediction was correct\. 
+ The last four values correspond to pixel dimensions that represent a bounding box around the predicted object in the image\.

These connectors don't publish MQTT messages as output data\.

## Usage Example<a name="obj-detection-connector-usage"></a>

The following example Lambda function uses the [AWS IoT Greengrass Machine Learning SDK](lambda-functions.md#lambda-sdks-ml) to interact with an ML Object Detection connector\.

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
    content = bytearray(f.read())

client = ml.client('inference')

def infer():
    logging.info('invoking Greengrass ML Inference service')

    try:
        resp = client.invoke_inference_service(
            AlgoType='object-detection',
            ServiceName='objectDetection',
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
    predictions = eval(predictions) 

    # Perform business logic that relies on the predictions.
    
    # Schedule the infer() function to run again in ten second.
    Timer(10, infer).start()
    return

infer()

def function_handler(event, context):
    return
```

The `invoke_inference_service` function in the AWS IoT Greengrass Machine Learning SDK accepts the following arguments\.


| Argument | Description | 
| --- | --- | 
| `AlgoType` | The name of the algorithm type to use for inference\. Currently, only `object-detection` is supported\. Required: `true` Type: `string` Valid values: `object-detection` | 
| `ServiceName` | The name of the local inference service\. Use the name that you specified for the `LocalInferenceServiceName` parameter when you configured the connector\. Required: `true` Type: `string` | 
| `ContentType` | The mime type of the input image\. Required: `true` Type: `string` Valid values: `image/jpeg, image/png` | 
| `Body` | The content of the input image file\. Required: `true` Type: `binary` | 

## Installing Neo deep learning runtime dependencies on the AWS IoT Greengrass core<a name="obj-detection-connector-config"></a>

The ML Object Detection connectors are bundled with the SageMaker Neo deep learning runtime \(DLR\)\. The connectors use the runtime to serve the ML model\. To use these connectors, you must install the dependencies for the DLR on your core device\. 

Before you install the DLR dependencies, make sure that the required [system libraries](#obj-detection-connector-logging) \(with the specified minimum versions\) are present on the device\.

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

1. From the directory where you saved the file, run the following command:

   ```
   sudo bash armv7l.sh
   ```
**Note**  
On a Raspberry Pi, using `pip` to install machine learning dependencies is a memory\-intensive operation that can cause the device to run out of memory and become unresponsive\. As a workaround, you can temporarily increase the swap size\. In `/etc/dphys-swapfile`, increase the value of the `CONF_SWAPSIZE` variable and then run the following command to restart `dphys-swapfile`\.  

   ```
   /etc/init.d/dphys-swapfile restart
   ```

------

## Logging and troubleshooting<a name="obj-detection-connector-logging"></a>

Depending on your group settings, event and error logs are written to CloudWatch Logs, the local file system, or both\. Logs from this connector use the prefix `LocalInferenceServiceName`\. If the connector behaves unexpectedly, check the connector's logs\. These usually contain useful debugging information, such as a missing ML library dependency or the cause of a connector startup failure\.

If the AWS IoT Greengrass group is configured to write local logs, the connector writes log files to `greengrass-root/ggc/var/log/user/region/aws/`\. For more information about Greengrass logging, see [Monitoring with AWS IoT Greengrass logs](greengrass-logs-overview.md)\.

Use the following information to help troubleshoot issues with the ML Object Detection connectors\.

**Required system libraries**

The following tabs list the system libraries required for each ML Object Detection connector\.

------
#### [ ML Object Detection Aarch64 JTX2 ]


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
| libnvinfer\.so\.4 | not applicable | 
| libnvrm\_gpu\.so | not applicable | 
| libnvrm\.so | not applicable | 
| libnvidia\-fatbinaryloader\.so\.28\.2\.1 | not applicable | 
| libnvos\.so | not applicable | 
| libpthread\.so\.0 | GLIBC\_2\.17 | 
| librt\.so\.1 | GLIBC\_2\.17 | 
| libstdc\+\+\.so\.6 | GLIBCXX\_3\.4\.21, CXXABI\_1\.3\.8 | 

------
#### [ ML Object Detection x86\_64 ]


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
#### [ ML Object Detection ARMv7 ]


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
|  On a Raspberry Pi, the following error message is logged and you are not using the camera: `Failed to initialize libdc1394`   |  Run the following command to disable the driver: <pre>sudo ln /dev/null /dev/raw1394</pre> This operation is ephemeral\. The symbolic link disappears after you reboot\. Consult the manual of your OS distribution to learn how to create the link automatically upon reboot\.  | 

## Licenses<a name="obj-detection-connector-license"></a>

The ML Object Detection connectors include the following third\-party software/licensing:<a name="boto-3-licenses"></a>
+ [AWS SDK for Python \(Boto3\)](https://pypi.org/project/boto3/)/Apache License 2\.0
+ [botocore](https://pypi.org/project/botocore/)/Apache License 2\.0
+ [dateutil](https://pypi.org/project/python-dateutil/1.4/)/PSF License
+ [docutils](https://pypi.org/project/docutils/)/BSD License, GNU General Public License \(GPL\), Python Software Foundation License, Public Domain
+ [jmespath](https://pypi.org/project/jmespath/)/MIT License
+ [s3transfer](https://pypi.org/project/s3transfer/)/Apache License 2\.0
+ [urllib3](https://pypi.org/project/urllib3/)/MIT License
+ [Deep Learning Runtime](https://github.com/neo-ai/neo-ai-dlr)/Apache License 2\.0
+ <a name="six-license"></a>[six](https://github.com/benjaminp/six)/MIT

This connector is released under the [Greengrass Core Software License Agreement](https://greengrass-release-license.s3.us-west-2.amazonaws.com/greengrass-license-v1.pdf)\.

## See also<a name="obj-detection-connector-see-also"></a>
+ [Integrate with services and protocols using Greengrass connectors](connectors.md)
+ [Getting started with Greengrass connectors \(console\)](connectors-console.md)
+ [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)
+ [Perform machine learning inference](ml-inference.md)
+ [Object detection algorithm](https://docs.aws.amazon.com/sagemaker/latest/dg/object-detection.html) in the *Amazon SageMaker Developer Guide*