# Module 2: Installing the AWS IoT Greengrass Core Software<a name="module2"></a>

This module shows you how to install the AWS IoT Greengrass Core software on your chosen device\. This tutorial provides instructions for setting up a Raspberry Pi, but you can use any supported device\. You can download the AWS IoT Greengrass Core software from the [AWS IoT Greengrass Core Software](what-is-gg.md#gg-core-download-tab) downloads\. This procedure includes steps for configuring and starting the software on your device\.

**Tip**  
<a name="ggc-install-options"></a>AWS IoT Greengrass also provides other options for installing the AWS IoT Greengrass Core software\. For example, you can use [Greengrass device setup](quick-start.md) to configure your environment and install the latest version of the AWS IoT Greengrass Core software\. Or, on supported Debian platforms, you can use the [APT package manager](install-ggc.md#ggc-package-manager) to install or upgrade the AWS IoT Greengrass Core software\. For more information, see [Install the AWS IoT Greengrass Core Software](install-ggc.md)\.

The AWS IoT Greengrass Core software provides the following functionality:<a name="ggc-software-features"></a>
+ Deployment and local execution of connectors and Lambda functions\.
+ Process data streams locally with automatic exports to the AWS Cloud\.
+ MQTT messaging over the local network between devices, connectors, and Lambda functions using managed subscriptions\.
+ MQTT messaging between AWS IoT and devices, connectors, and Lambda functions using managed subscriptions\.
+ Secure connections between devices and the cloud using device authentication and authorization\.
+ Local shadow synchronization of devices\. Shadows can be configured to sync with the cloud\.
+ Controlled access to local device and volume resources\.
+ Deployment of cloud\-trained machine learning models for running local inference\.
+ Automatic IP address detection that enables devices to discover the Greengrass core device\.
+ Central deployment of new or updated group configuration\. After the configuration data is downloaded, the core device is restarted automatically\.
+ Secure, over\-the\-air \(OTA\) software updates of user\-defined Lambda functions\.
+ Secure, encrypted storage of local secrets and controlled access by connectors and Lambda functions\.

Before you begin, make sure that you have completed [Module 1](module1.md)\.

This module should take less than 30 minutes to complete\.

**Topics**
+ [Configure AWS IoT Greengrass on AWS IoT](gg-config.md)
+ [Start AWS IoT Greengrass on the Core Device](gg-device-start.md)