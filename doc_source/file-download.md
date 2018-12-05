# Download Required Files<a name="file-download"></a>

1. If you haven’t already done so, install the [AWS IoT Device SDK for Python](https://github.com/aws/aws-iot-device-sdk-python)\. Follow the instructions in the `README` file\. This SDK is used by all AWS IoT devices to communicate with the AWS IoT cloud and AWS IoT Greengrass cores\.

1. From the [AWS IoT Greengrass samples](https://github.com/aws-samples/aws-greengrass-samples/tree/master/traffic-light-example-python) repository on GitHub, download the `lightController.py` and `trafficLight.py` files to your computer and move them to the folder that contains the GG\_Switch and GG\_TrafficLight device certificates:  
![\[Screenshot of files including the two Python scripts and the device certificates and keys.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-082.png)

   The `lightController.py` script corresponds to the GG\_Switch device, and the `trafficLight.py` script corresponds to the GG\_TrafficLight device\. 