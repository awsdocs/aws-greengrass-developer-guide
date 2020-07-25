# Getting started with AWS IoT Greengrass<a name="gg-gs"></a>

This Getting Started tutorial includes several modules designed to show you AWS IoT Greengrass basics and help you get started using AWS IoT Greengrass\. This tutorial covers fundamental concepts, such as:
+ Configuring AWS IoT Greengrass cores and groups\.
+ The deployment process for running AWS Lambda functions at the edge\.
+ Connecting AWS IoT devices to the AWS IoT Greengrass core\.
+ Creating subscriptions to allow MQTT communication between local Lambda functions, devices, and AWS IoT\.

## Choose how to get started with AWS IoT Greengrass<a name="gg-getting-started"></a>

You can choose how to use this tutorial to set up your core device:
+ Run [Greengrass device setup](quick-start.md) on your core device, which takes you from installing AWS IoT Greengrass dependencies to testing a Hello World Lambda function in minutes\. This script reproduces the steps in Module 1 through Module 3\-1\.

   

   \- or \-

   
+ Walk through the steps in Module 1 through Module 3\-1 to examine Greengrass requirements and processes more closely\. These steps set up your core device, create and configure a Greengrass group that contains a Hello World Lambda function, and deploy your Greengrass group\. Typically, this takes an hour or two to complete\.

![\[Getting Started modules\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/getting-started-modules.png)

**Quick Start**  
[Greengrass device setup](quick-start.md) configures your core device and Greengrass resources\. The script:  
+ Installs AWS IoT Greengrass dependencies\.
+ Downloads the root CA certificate and core device certificate and keys\.
+ Downloads, installs, and configures the AWS IoT Greengrass Core software on your device\.
+ Starts the Greengrass daemon process on the core device\.
+ Creates or updates the [Greengrass service role](service-role.md), if needed\.
+ Creates a Greengrass group and Greengrass core\.
+ \(Optional\) Creates a Hello World Lambda function, subscription, and local logging configuration\.
+ \(Optional\) Deploys the Greengrass group\.

**Modules 1 and 2**  
[Module 1](module1.md) and [Module 2](module2.md) describe how to set up your environment\. \(Or, use [Greengrass device setup](quick-start.md) to run these modules for you\.\)  
+ Configure your core device for Greengrass\.
+ Run the dependency checker script\.
+ Create a Greengrass group and Greengrass core\.
+ Download and install the latest AWS IoT Greengrass Core software from a tar\.gz file\.
+ Start the Greengrass daemon process on the core\.
AWS IoT Greengrass also provides other options for installing the AWS IoT Greengrass Core software, including `apt` installations on supported Debian platforms\. For more information, see [Install the AWS IoT Greengrass Core software](install-ggc.md)\.

**Modules 3\-1 and 3\-2**  
[Module 3\-1](module3-I.md) and [Module 3\-2](module3-II.md) describe how to use local Lambda functions\. \(Or, use [Greengrass device setup](quick-start.md) to run Module 3\-1 for you\.\)  
+ Create Hello World Lambda functions in AWS Lambda\.
+ Add Lambda functions to your Greengrass group\.
+ Create subscriptions that allow MQTT communication between the Lambda functions and AWS IoT\.
+ Configure local logging for Greengrass system components and Lambda functions\.
+ Deploy a Greengrass group that contains your Lambda functions and subscriptions\.
+ Send messages from local Lambda functions to AWS IoT\.
+ Invoke local Lambda functions from AWS IoT\.
+ Test on\-demand and long\-lived functions\.

**Modules 4 and 5**  
[Module 4](module4.md) shows how devices connect to the core and communicate with each other\.  
[Module 5](module5.md) shows how devices can use shadows to control state\.  
+ Register and provision AWS IoT devices \(represented by command\-line terminals\)\.
+ Install the AWS IoT Device SDK for Python\. This is used by devices to discover the Greengrass core\.
+ Add the devices to your Greengrass group\.
+ Create subscriptions that allow MQTT communication\.
+ Deploy a Greengrass group that contains your devices\.
+ Test device\-to\-device commumication\.
+ Test shadow state updates\.

**Module 6**  
[Module 6](module6.md) shows you how Lambda functions can access the AWS Cloud\.  
+ Create a Greengrass group role that allows access to Amazon DynamoDB resources\.
+ Add a Lambda function to your Greengrass group\. This function uses the AWS SDK for Python to interact with DynamoDB\.
+ Create subscriptions that allow MQTT communication\.
+ Test the interaction with DynamoDB\.

**Module 7**  
[Module 7](console-mod7.md) shows you how to configure a simulated hardware security module \(HSM\) for use with a Greengrass core\.  
This advanced module is provided only for experimentation and initial testing\. It is not for production use of any kind\.
+ Install and configure a software\-based HSM and private key\.
+ Configure the Greengrass core to use hardware security\.
+ Test the hardware security configuration\.

## Requirements<a name="gg-requirements"></a>

To complete this tutorial, you need the following:
+ A Mac, Windows PC, or UNIX\-like system\.
+ An Amazon Web Services \(AWS\) account\. If you don't have one, see [Create an AWS account](#create-aws-account)\.
+ The use of an AWS [Region](https://en.wikipedia.org/wiki/Amazon_Web_Services#Availability_and_topology) that supports AWS IoT Greengrass\. For the list of supported regions for AWS IoT Greengrass, see [AWS endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) in the *AWS General Reference*\.
**Note**  
Make a note of your AWS Region and make sure that it is consistently used throughout this tutorial\. If you switch AWS Regions during the tutorial, you might experience problems completing the steps\.
+ A Raspberry Pi 4 Model B, or Raspberry Pi 3 Model B/B\+, with a 8 GB microSD card, or an Amazon EC2 instance\. Because AWS IoT Greengrass should ideally be used with physical hardware, we recommend that you use a Raspberry Pi\.
**Note**  
Run the following command to get the model of your Raspberry Pi:  

  ```
  cat /proc/cpuinfo
  ```
Near the bottom of the listing, make a note of the value of the `Revision` attribute and then consult the [Which Pi have I got?](https://elinux.org/RPi_HardwareHistory#Which_Pi_have_I_got.3F) table\. For example, if the value of `Revision` is `a02082`, the table shows the Pi is a 3 Model B\.   
Run the following command to determine the architecture of your Raspberry Pi:  

  ```
  uname -m
  ```
For this tutorial, the result should be greater than or equal to `armv71`\.
+ Basic familiarity with Python\.

Although this tutorial is intended to run AWS IoT Greengrass on a Raspberry Pi, AWS IoT Greengrass also supports other platforms\. For more information, see [Supported platforms and requirements](what-is-gg.md#gg-platforms)\.

## Create an AWS account<a name="create-aws-account"></a>

If you don't have an AWS account, follow these steps to create and activate an AWS account:<a name="create-aws-account-steps"></a>

1. Open the [AWS home page](https://aws.amazon.com/), and choose **Create an AWS Account**\.
**Note**  
If you've signed in to AWS recently, you might see **Sign In to the Console** instead\.

1. Follow the online instructions\. Part of the sign\-up procedure includes registering a credit card, receiving a text message or phone call, and entering a PIN\.

   For more information, see [How do I create and activate a new Amazon Web Services account?](http://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)

**Important**  
For this tutorial, we assume that your IAM user account has administrator access permissions\.