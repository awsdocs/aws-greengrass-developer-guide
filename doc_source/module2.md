# Module 2: Installing the Greengrass Core Software<a name="module2"></a>

This module shows you how to install the AWS IoT Greengrass core software on your device\. This tutorial provides instructions for setting up a Raspberry Pi, but you can use any supported device\. You can download the core software from the [AWS IoT Greengrass Core Software](what-is-gg.md#gg-core-download-tab) downloads\. This procedure includes steps for configuring and starting the core software on your device\.

The AWS IoT Greengrass core software provides the following functionality:
+ Deployment and local execution of connectors and Lambda functions\.
+ Secure, encrypted storage of local secrets and controlled access by connectors and Lambda functions\.
+ MQTT messaging over the local network between devices, connectors, and Lambda functions using managed subscriptions\.
+ MQTT messaging between AWS IoT and devices, connectors, and Lambda functions using managed subscriptions\.
+ Secure connections between devices and the cloud using device authentication and authorization\.
+ Local shadow synchronization of devices\. Shadows can be configured to sync with the cloud\.
+ Controlled access to local device and volume resources\.
+ Deployment of cloud\-trained machine learning models for running local inference\.
+ Automatic IP address detection that enables devices to discover the Greengrass core device\.
+ Central deployment of new or updated group configuration\. After the configuration data is downloaded, the core device is restarted automatically\.
+ Secure, over\-the\-air software updates of user\-defined Lambda functions\.

Before you begin, make sure that you have completed [Module 1](module1.md)\.

This module should take less than 30 minutes to complete\.