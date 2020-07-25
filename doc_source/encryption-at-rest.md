# Encryption at rest<a name="encryption-at-rest"></a>

AWS IoT Greengrass stores your data:
+ [Data at rest in the AWS Cloud](#data-at-rest-cloud)\. This data is encrypted\.
+ [Data at rest on the Greengrass core](#data-at-rest-device)\. This data is not encrypted \(except local copies of your secrets\)\.

## Data at rest in the AWS Cloud<a name="data-at-rest-cloud"></a>

AWS IoT Greengrass encrypts customer data stored in the AWS Cloud\. This data is protected using AWS KMS keys that are managed by AWS IoT Greengrass\.

## Data at rest on the Greengrass core<a name="data-at-rest-device"></a>

AWS IoT Greengrass relies on Unix file permissions and full\-disk encryption \(if enabled\) to protect data at rest on the core\. It is your responsibility to secure the file system and device\.

However, AWS IoT Greengrass does encrypt local copies of your secrets retrieved from AWS Secrets Manager\. For more information, see [Secrets encryption](secrets.md#secrets-encryption)\.