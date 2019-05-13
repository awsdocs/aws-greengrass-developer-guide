# Getting Started with AWS IoT Greengrass<a name="gg-gs"></a>

This tutorial includes six modules, each designed to show you AWS IoT Greengrass basics and help you get started in as few steps as possible\. This tutorial covers:
+ The AWS IoT Greengrass programming model\.
+  Fundamental concepts, such as AWS IoT Greengrass cores, groups, and subscriptions\.
+ The deployment process for running AWS Lambda functions at the edge\.

## Requirements<a name="gg-requirements"></a>

To complete this tutorial, you need the following:
+ A Mac, Windows PC, or UNIX\-like system\.
+ An Amazon Web Services \(AWS\) account\. If you don’t have one, see [Create an AWS Account](#create-aws-account)\.
+ The use of an AWS [Region](https://en.wikipedia.org/wiki/Amazon_Web_Services#Availability_and_topology) that supports AWS IoT Greengrass\. For the list of supported regions for AWS IoT Greengrass, see [AWS Regions and Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#greengrass_region) in the *AWS General Reference*\.
**Important**  
Make a note of your AWS Region to ensure that it is consistently used throughout this tutorial\. Inadvertently switching regions during the tutorial can create problems\. The last exercise in this tutorial assumes you are using the US East \(N\. Virgina\) Region\.
+ A Raspberry Pi 3 [Model B\+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) or [Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) with a 8 GB microSD card, or an Amazon EC2 instance\. Because AWS IoT Greengrass should ideally be used with physical hardware, we recommend that you use a Raspberry Pi\. The architecture of your Pi must be `armv7l` or later\.
**Note**  
Run the following command to get the model of your Rasberry Pi:  

  ```
  cat /proc/cpuinfo
  ```
Near the bottom of the listing, make a note of the value of the `Revision` attribute and then consult the [Which Pi have I got?](https://elinux.org/RPi_HardwareHistory#Which_Pi_have_I_got.3F) table\. For example, if the value of `Revision` is `a02082`, the table shows the Pi is a 3 Model B\.   
Run the following command to determine the architecture of your Raspberry Pi:  

  ```
  uname -m
  ```
The result must be greater than or equal to `armv71`\.
+ Basic familiarity with Python 2\.7\.

Although this tutorial provides steps for running AWS IoT Greengrass on a Raspberry Pi , you can use other platforms\. For more information, see [Supported Platforms and Requirements](what-is-gg.md#gg-platforms)\.

## Create an AWS Account<a name="create-aws-account"></a>

If you don't have an AWS account, follow these steps:

1. Open the [AWS home page](https://aws.amazon.com/), and choose **Create an AWS Account**\. 
**Note**  
If you've signed in to AWS recently, you might see **Sign In to the Console** instead\.

1. Follow the online instructions\. Part of the sign\-up procedure involves receiving a phone call and entering a PIN using your phone keypad\.
**Important**  
Before you begin, make sure that your account has administrative privileges\.