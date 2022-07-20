--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Download required files<a name="file-download"></a>

1. If you haven't already done so, install the AWS IoT Device SDK for Python\. For instructions, see step 1 in [Install the AWS IoT Device SDK for Python](IoT-SDK.md)\.

   This SDK is used by client devices to communicate with AWS IoT and with AWS IoT Greengrass core devices\.

1. From the [ TrafficLight](https://github.com/aws/aws-greengrass-core-sdk-python/tree/master/examples/TrafficLight) examples folder on GitHub, download the `lightController.py` and `trafficLight.py` files to your computer\. Save them in the folder that contains the GG\_Switch and GG\_TrafficLight client device certificates and keys\.

   The `lightController.py` script corresponds to the GG\_Switch client device, and the `trafficLight.py` script corresponds to the GG\_TrafficLight client device\.   
![\[Screenshot of files including the two Python scripts and the device certificates and keys.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-082.png)
**Note**  
The example Python files are stored in the AWS IoT Greengrass Core SDK for Python repository for convenience, but they don't use the AWS IoT Greengrass Core SDK\.