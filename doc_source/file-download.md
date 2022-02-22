--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

--------

# Download required files<a name="file-download"></a>

1. If you haven't already done so, install the AWS IoT Device SDK for Python\. For instructions, see step 1 in [Install the AWS IoT Device SDK for Python](IoT-SDK.md)\.

   This SDK is used by AWS IoT devices to communicate with AWS IoT and with AWS IoT Greengrass core devices\.

1. From the [ TrafficLight](https://github.com/aws/aws-greengrass-core-sdk-python/tree/master/examples/TrafficLight) examples folder on GitHub, download the `lightController.py` and `trafficLight.py` files to your computer\. Save them in the folder that contains the GG\_Switch and GG\_TrafficLight device certificates and keys\.

   The `lightController.py` script corresponds to the GG\_Switch device, and the `trafficLight.py` script corresponds to the GG\_TrafficLight device\.   
![\[Screenshot of files including the two Python scripts and the device certificates and keys.\]](http://docs.aws.amazon.com/greengrass/v1/developerguide/images/gg-get-started-082.png)
**Note**  
The example Python files are stored in the AWS IoT Greengrass Core SDK for Python repository for convenience, but they don't use the AWS IoT Greengrass Core SDK\.