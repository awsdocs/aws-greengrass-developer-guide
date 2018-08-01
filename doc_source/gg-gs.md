# Getting Started with AWS Greengrass<a name="gg-gs"></a>

This tutorial includes six modules, each designed to show you AWS Greengrass basics and help you get started in as few steps as possible\. This tutorial covers:
+ The AWS Greengrass programming model\.
+  Fundamental concepts, such as AWS Greengrass cores, groups, and subscriptions\.
+ The deployment process for running AWS Lambda functions at the edge\.

## Requirements<a name="gg-requirements"></a>

To complete this tutorial, you will need the following:
+ A Mac, Windows PC, or UNIX\-like system\.
+ An Amazon Web Services \(AWS\) account\. If you don’t have an AWS account, see [Create an AWS Account](#create-aws-account)\.
+ The use of an AWS [region](https://en.wikipedia.org/wiki/Amazon_Web_Services#Availability_and_topology) that supports AWS Greengrass\. For the list of supported regions for AWS Greengrass, see [AWS Regions and Endpoints](http://docs.aws.amazon.com/general/latest/gr/rande.html#greengrass_region) in the *AWS General Reference*\.
**Important**  
Make note of your region to ensure that it is consistently used throughout this tutorial – inadvertently switching regions midway through the tutorial would be problematic\. Note that the last exercise in this tutorial assumes the US East \(N\. Virgina\) region, so you may want to only use the US East \(N\. Virgina\) region, as possible\.
+ A [Raspberry Pi 3 Model B](https://www.amazon.com/Raspberry-Model-1-2GHz-64-bit-quad-core/dp/B01CD5VC92/ref=sr_1_3?ie=UTF8&qid=1510013943&sr=8-3&keywords=Raspberry+Pi+Model+3&dpID=51wEoDfvlIL&preST=_SX300_QL70_&dpSrc=srch) with a 8 GB microSD card, or an Amazon EC2 instance\. Because AWS Greengrass is intended to be used with physical hardware, we recommend that you use a Raspberry Pi\. 
**Note**  
If the model of your Raspberry Pi is unknown, you can run the following command:  

  ```
  cat /proc/cpuinfo
  ```
Near the bottom of the listing, note the value of the `Revision` attribute\. You can determine the model of your Pi by using this value along with the table at [Which Pi have I got?](https://elinux.org/RPi_HardwareHistory#Which_Pi_have_I_got.3F) For example, if the value of `Revision` is `a02082`, then from the table we see that the Pi is a 3 Model B\. Additionally, the architecture of your Pi must be `armv71` or greater\. To determine the architecture of your Raspberry Pi, run the following command:  

  ```
  uname -m
  ```
The result must be greater than or equal to `armv71`\.
+ Basic familiarity with Python 2\.7\.

Although this tutorial focuses on running AWS Greengrass on a Raspberry Pi or an Amazon EC2 instance, other platforms are supported\. For more information, see [Supported Platforms and Requirements](what-is-gg.md#gg-platforms)\.

## Create an AWS Account<a name="create-aws-account"></a>

If you don't have an AWS account, follow these steps:

1. Open the [AWS home page](http://aws.amazon.com/), and choose **Create an AWS Account**\. 
**Note**  
If you've signed in to AWS recently, you might see **Sign In to the Console** instead\.

1. Follow the online instructions\. Part of the sign\-up procedure involves receiving a phone call and entering a PIN using your phone keypad\.
**Important**  
Ensure that your account has administrative privileges before proceeding\.