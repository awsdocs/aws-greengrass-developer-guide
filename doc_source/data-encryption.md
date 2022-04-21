--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Data encryption<a name="data-encryption"></a>

AWS IoT Greengrass uses encryption to protect data while in\-transit \(over the internet or local network\) and at rest \(stored in the AWS Cloud\)\.

Devices in a AWS IoT Greengrass environment often collect data that's sent to AWS services for further processing\. For more information about data encryption on other AWS services, see the security documentation for that service\.

**Topics**
+ [Encryption in transit](encryption-in-transit.md)
+ [Encryption at rest](encryption-at-rest.md)
+ [Key management for the Greengrass core device](key-management.md)