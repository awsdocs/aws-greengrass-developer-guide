--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# AWS IoT Greengrass Version 1 maintenance policy<a name="maintenance-policy"></a>

Use this AWS IoT Greengrass V1 maintenance policy to understand the different levels of maintenance and updates for the AWS IoT Greengrass V1 service and the AWS IoT Greengrass Core software v1\.x\.

**Topics**
+ [AWS IoT Greengrass versioning scheme](#greengrass-versioning-scheme)
+ [Lifecycle phases for major versions of the AWS IoT Greengrass Core software](#greengrass-software-lifecycle-phases)
+ [Maintenance policy for AWS IoT Greengrass Core software](#greengrass-software-maintenance-policy)
+ [Deprecation schedule](#deprecation-schedule)
+ [Support policy for AWS Lambda functions on Greengrass core devices](#lambda-function-support-policy)
+ [Support policy for AWS IoT Device Tester for AWS IoT Greengrass V1](#idt-support-policy-summary)
+ [End of maintenance schedule](#end-of-maintenance-schedule)

## AWS IoT Greengrass versioning scheme<a name="greengrass-versioning-scheme"></a>

AWS IoT Greengrass uses [semantic versioning](https://semver.org/) for the AWS IoT Greengrass Core software\. Semantic versions follow a *major*\.*minor*\.*patch* number system\. The major version increments for functional and API changes that aren't backward\-compatible with previous major versions\. The minor version increments for releases that add new backward\-compatible functionality\. The patch version increments for security patches or bug fixes\. Since its first major release, v1\.0\.0, AWS IoT Greengrass has released 11 minor versions of the AWS IoT Greengrass Core software v1\.x, where v1\.11\.6 is the latest release\. We recommend that you update your AWS IoT Greengrass Core software to the latest available version to take advantage of new features, enhancements, and bug fixes\.

In December 2020, AWS IoT Greengrass released its first major version update\. This update included the AWS IoT Greengrass V2 service and version 2\.0\.3 of the AWS IoT Greengrass Core software\. For new applications, we strongly recommend that you use AWS IoT Greengrass Version 2 and the AWS IoT Greengrass Core software v2\.x\. Version 2 receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What is AWS IoT Greengrass V2?](https://docs.aws.amazon.com/greengrass/v2/developerguide/what-is-iot-greengrass.html)\.

## Lifecycle phases for major versions of the AWS IoT Greengrass Core software<a name="greengrass-software-lifecycle-phases"></a>

Each major version of the AWS IoT Greengrass Core software has the following three sequential lifecycle phases\. Each lifecycle phase provides different levels of maintenance over a period of time after the initial release date\.
+ **Release phase** – AWS IoT Greengrass may release the following updates:
  + Minor version updates that provide new features or enhancements to existing features
  + Patch version updates that provide security patches and bug fixes
+ **Maintenance phase** – AWS IoT Greengrass may release patch version updates that provide security patches and bug fixes\. AWS IoT Greengrass won't release new features or enhancements to existing features during the maintenance phase\.
+ **Extended life phase** – AWS IoT Greengrass won't release updates that provide features, enhancements to existing features, security patches, or bug fixes\. However, the AWS Cloud endpoints and API operations will remain available and operate according to the [AWS IoT Greengrass Service Level Agreement](http://aws.amazon.com/greengrass/sla)\. Devices that run the AWS IoT Greengrass Core software v1\.x can continue to connect to the AWS Cloud and operate\.

After the extended life phase ends for a major version of AWS IoT Greengrass, the AWS Cloud endpoints and API operations will be deprecated and no longer available\. Devices that run the AWS IoT Greengrass Core software v1\.x won't be able to connect to AWS Cloud services to operate\.

## Maintenance policy for AWS IoT Greengrass Core software<a name="greengrass-software-maintenance-policy"></a>

The AWS IoT Greengrass Core software v1\.x is currently in the maintenance phase until June 30, 2023\. On this date, the AWS IoT Greengrass Core software v1\.x enters the extended life phase, and it will remain in the extended life phase until further notice\.

The AWS IoT Greengrass Core software v2\.x is currently in the release phase, and it will remain in the release phase until further notice\. AWS IoT Greengrass continues to add new features and enhancements to the AWS IoT Greengrass Core software v2\.x\. For example, AWS IoT Greengrass released Windows support in v2\.5\.0 of the AWS IoT Greengrass Core software\. AWS IoT Greengrass releases security patches and bug fixes for all minor versions of AWS IoT Greengrass Core v2\.x for at least 1 year after the release date\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html)\.

### Maintenance phase schedule<a name="maintenance-phase-schedule"></a>

On June 30, 2023, the maintenance phase ends for the AWS IoT Greengrass Core software v1\.11\.x\. On March 31, 2022, the maintenance phase ended for the AWS IoT Greengrass Core software v1\.10\.x\. The maintenance phase ends for certain AWS IoT Greengrass Core software v1\.x artifacts and features earlier than these dates\. For more information, see [End of maintenance schedule](#end-of-maintenance-schedule)\. During the maintenance phase, AWS IoT Greengrass won't add new features or enhancements to the AWS IoT Greengrass Core software v1\.x\.

If you have an AWS Support plan, the maintenance phase for AWS IoT Greengrass Core software v1\.x doesn't affect your AWS Support plan\. You can continue to open AWS Support tickets even after the maintenance phase ends\. If you have questions or concerns, contact your AWS Support contact, or ask a question on [AWS re:Post](https://repost.aws/tags/TA4ckIed1sR4enZBey29rKTg/aws-io-t-greengrass) using the **AWS IoT Greengrass** tag\.

## Deprecation schedule<a name="deprecation-schedule"></a>

Currently, there is no plan to stop supporting the AWS IoT Greengrass Core software v1\.x\. The AWS IoT Greengrass V1 endpoints and API operations will remain available until further notice\. After the maintenance phase ends on June 30, 2023, the AWS IoT Greengrass Core software v1\.11\.6 will enter the extended life phase until further notice\. During this phase, devices that run the AWS IoT Greengrass Core software v1\.x can continue to connect to the AWS IoT Greengrass V1 service to operate until further notice\.

If AWS IoT Greengrass V1 stops being supported in the future, AWS IoT Greengrass will provide 12 months advance notice before this happens\. This will help you plan the update of your applications to use AWS IoT Greengrass V2 and the AWS IoT Greengrass Core software v2\.x\. For more information about how to update your applications to V2, see [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

## Support policy for AWS Lambda functions on Greengrass core devices<a name="lambda-function-support-policy"></a>

AWS IoT Greengrass enables you to run AWS Lambda functions on IoT devices\. AWS Lambda provides a support policy and timelines that determine support for Lambda runtimes in AWS IoT Greengrass\. After a Lambda runtime reaches the end of support phase, AWS IoT Greengrass also ends support for that runtime\. For more information, see [Runtime support policy](https://docs.aws.amazon.com/lambda/latest/dg/runtime-support-policy.html) in the *AWS Lambda Developer Guide*\.

When a Lambda runtime reaches end of support, you can't create or update Lambda functions that use that runtime\. However, you can continue to deploy these Lambda functions to Greengrass core devices and invoke deployed Lambda functions\. This policy also applies to AWS IoT Greengrass V2\.

## Support policy for AWS IoT Device Tester for AWS IoT Greengrass V1<a name="idt-support-policy-summary"></a>

AWS IoT Device Tester \(IDT\) for AWS IoT Greengrass V1 enables you to validate and [qualify](https://aws.amazon.com/partners/dqp/) your AWS IoT Greengrass devices for inclusion in the [AWS Partner Device Catalog](https://devices.amazonaws.com/)\. As of April 4, 2022, AWS IoT Device Tester \(IDT\) for AWS IoT Greengrass V1 no longer generates signed qualification reports\. You can no longer qualify new AWS IoT Greengrass V1 devices to list in the [AWS Partner Device Catalog](https://devices.amazonaws.com/) through the [AWS Device Qualification Program](http://aws.amazon.com/partners/programs/dqp/)\. While you can't qualify Greengrass V1 devices, you can continue to use IDT for AWS IoT Greengrass V1 to test your Greengrass V1 devices\. We recommend that you use [IDT for AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/device-tester-for-greengrass-ug.html) to qualify and list Greengrass devices in the [AWS Partner Device Catalog](https://devices.amazonaws.com/)\. For more information, see [Support policy for AWS IoT Device Tester for AWS IoT Greengrass V1](idt-support-policy.md)\.

## End of maintenance schedule<a name="end-of-maintenance-schedule"></a>

The following table lists end of maintenance dates for AWS IoT Greengrass Core v1\.x artifacts and features\. If you have questions about the maintenance schedule or policy, contact [AWS Support](http://aws.amazon.com/contact-us)\.


| Artifact or feature | End of maintenance date | 
| --- | --- | 
|  Greengrass APT repository installation  |  February 11, 2022  | 
|  ML Image Classification connector  |  March 31, 2022  | 
|  ML Object Detection connector  |  March 31, 2022  | 
|  ML Feedback connector  |  March 31, 2022  | 
|  AWS IoT Analytics connector  |  March 31, 2022  | 
|  Twilio Notifications connector  |  March 31, 2022  | 
|  Splunk Integration connector  |  March 31, 2022  | 
|  Serial Stream connector  |  March 31, 2022  | 
|  ServiceNow MetricBase Integration connector  |  March 31, 2022  | 
|  Raspberry Pi GPIO connector  |  March 31, 2022  | 
|  AWS IoT Greengrass Core software v1\.10\.x  |  March 31, 2022  | 
|  AWS IoT Greengrass Core software v1\.x Docker images  |  June 30, 2022  | 
|  AWS IoT Greengrass Core software v1\.11\.x  |  June 30, 2023  | 

### End of maintenance for AWS IoT Greengrass Core software v1\.x Docker images<a name="docker-images-end-of-maintenance"></a>

On June 30, 2022, AWS IoT Greengrass ends maintenance for AWS IoT Greengrass Core software v1\.x Docker images that are published to Amazon Elastic Container Registry \(Amazon ECR\) and Docker Hub\. You can continue to download these Docker images from Amazon ECR and Docker Hub until June 30, 2023, which is 1 year after maintenance ends\. However, the AWS IoT Greengrass Core software v1\.x Docker images won't receive security patches or bug fixes after maintenance ends on June 30, 2022\. If you run a production workload that depends on these Docker images, we recommend that you build your own Docker images using the Dockerfiles that AWS IoT Greengrass provides\. For more information, see [AWS IoT Greengrass Docker software](what-is-gg.md#gg-docker-download)\.

### End of maintenance for AWS IoT Greengrass Core software v1\.x APT repository<a name="apt-repository-end-of-maintenance"></a>

On February 11, 2022, AWS IoT Greengrass ended maintenance for the option to [install the AWS IoT Greengrass Core software v1\.x from an APT repository](install-ggc.md#ggc-package-manager)\. The APT repository was removed on this date, so you can no longer to use the APT repository to update the AWS IoT Greengrass Core software or install the AWS IoT Greengrass Core software on new devices\. On devices where you added the AWS IoT Greengrass repository, you must [remove the repository from the sources list](install-ggc.md#ggc-package-manager-remove-sources)\. We recommend that you update the AWS IoT Greengrass Core software v1\.x using [tar files](install-ggc.md#download-and-extract-tarball)\.