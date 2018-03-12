# AWS Greengrass Developer Guide

-----
*****Copyright &copy; 2018 Amazon Web Services, Inc. and/or its affiliates. All rights reserved.*****

-----
Amazon's trademarks and trade dress may not be used in 
     connection with any product or service that is not Amazon's, 
     in any manner that is likely to cause confusion among customers, 
     or in any manner that disparages or discredits Amazon. All other 
     trademarks not owned by Amazon are the property of their respective
     owners, who may or may not be affiliated with, connected to, or 
     sponsored by Amazon.

-----
## Contents
+ [What Is AWS Greengrass?](what-is-gg.md)
+ [Getting Started with AWS Greengrass](gg-gs.md)
   + [Module 1: Environment Setup for Greengrass](module1.md)
   + [Module 2: Installing the Greengrass Core Software](module2.md)
      + [Configure AWS Greengrass on AWS IoT](gg-config.md)
      + [Start AWS Greengrass on the Core Device](gg-device-start.md)
   + [Module 3 (Part 1): Lambda Functions on AWS Greengrass](module3-I.md)
      + [Create and Package a Lambda Function](create-lambda.md)
      + [Configure the Lambda Function for AWS Greengrass](config-lambda.md)
      + [Deploy Cloud Configurations to an AWS Greengrass Core Device](configs-core.md)
      + [Verify the Lambda Function Is Running on the Device](lambda-check.md)
   + [Module 3 (Part 2): Lambda Functions on AWS Greengrass](module3-II.md)
      + [Create and Package the Lambda Function](package.md)
      + [Configure Long-Lived Lambda Functions for AWS Greengrass](long-lived.md)
      + [Test Long-Lived Lambda Functions](long-testing.md)
      + [Test On-Demand Lambda Functions](on-demand.md)
   + [Module 4: Interacting with Devices in an AWS Greengrass Group](module4.md)
      + [Create AWS IoT Devices in an AWS Greengrass Group](device-group.md)
      + [Configure Subscriptions](config-subs.md)
      + [Install the AWS IoT Device SDK for Python](IoT-SDK.md)
      + [Test Communications](test-comms.md)
   + [Module 5: Interacting with Device Shadows](module5.md)
      + [Configure Devices and Subscriptions](config-dev-subs.md)
      + [Download Required Files](file-download.md)
      + [Test Communications (Device Syncs Disabled)](comms-disabled.md)
      + [Test Communications (Device Syncs Enabled)](comms-enabled.md)
   + [Module 6: Accessing AWS Cloud Services](module6.md)
      + [Configure IAM Roles](config-iam-roles.md)
      + [Create and Configure the Lambda Function](create-config-lambda.md)
      + [Configure Subscriptions](config_subs.md)
      + [Test Communications](comms-test.md)
+ [OTA Updates of AWS Greengrass Core Software](core-ota-update.md)
+ [Reset Deployments](reset-deployments-scenario.md)
+ [Access Local Resources with Lambda Functions](access-local-resources.md)
   + [How to Configure Local Resource Access Using the AWS Command Line Interface](lra-cli.md)
   + [How to Configure Local Resource Access Using the AWS Management Console](lra-console.md)
+ [Greengrass Discovery RESTful API](gg-discover-api.md)
+ [Use Greengrass OPC-UA to Communicate with Industrial Equipment](opcua.md)
+ [AWS Greengrass Security](gg-sec.md)
+ [Monitoring with AWS Greengrass Logs](greengrass-logs-overview.md)
+ [Troubleshooting AWS Greengrass Applications](gg-troubleshooting.md)