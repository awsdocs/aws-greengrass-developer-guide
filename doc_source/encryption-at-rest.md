--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

--------

# Encryption at rest<a name="encryption-at-rest"></a>

AWS IoT Greengrass stores your data:
+ [Data at rest in the AWS Cloud](#data-at-rest-cloud)\. This data is encrypted\.
+ [Data at rest on the Greengrass core](#data-at-rest-device)\. This data is not encrypted \(except local copies of your secrets\)\.

## Data at rest in the AWS Cloud<a name="data-at-rest-cloud"></a>

AWS IoT Greengrass encrypts customer data stored in the AWS Cloud\. This data is protected using AWS KMS keys that are managed by AWS IoT Greengrass\.

## Data at rest on the Greengrass core<a name="data-at-rest-device"></a>

AWS IoT Greengrass relies on Unix file permissions and full\-disk encryption \(if enabled\) to protect data at rest on the core\. It is your responsibility to secure the file system and device\.

However, AWS IoT Greengrass does encrypt local copies of your secrets retrieved from AWS Secrets Manager\. For more information, see [Secrets encryption](secrets.md#secrets-encryption)\.