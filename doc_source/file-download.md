# Download required files<a name="file-download"></a>

1. If you haven't already done so, install the AWS IoT Device SDK for Python\. For instructions, see step 1 in [Install the AWS IoT Device SDK for Python](IoT-SDK.md)\.

   This SDK is used by AWS IoT devices to communicate with AWS IoT and with AWS IoT Greengrass core devices\.

1. From the [ TrafficLight](https://github.com/aws/aws-greengrass-core-sdk-python/tree/master/examples/TrafficLight) examples folder on GitHub, download the `lightController.py` and `trafficLight.py` files to your computer\. Save them in the folder that contains the GG\_Switch and GG\_TrafficLight device certificates and keys\.

   The `lightController.py` script corresponds to the GG\_Switch device, and the `trafficLight.py` script corresponds to the GG\_TrafficLight device\.   
![\[Screenshot of files including the two Python scripts and the device certificates and keys.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-082.png)
**Note**  
The example Python files are stored in the AWS IoT Greengrass Core SDK for Python repository for convenience, but they don't use the AWS IoT Greengrass Core SDK\.