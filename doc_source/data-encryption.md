--------

You are viewing the documentation for AWS IoT Greengrass Version 1, which has moved into [maintenance mode](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. If you're new to AWS IoT Greengrass, we strongly recommend that you use AWS IoT Greengrass Version 2, which receives new features, includes all key V1 features, and supports additional platforms and continuous deployments to large fleets of devices\. For more information, see [What's new in AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [Move from AWS IoT Greengrass V1 to V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html)\.

--------

# Data encryption<a name="data-encryption"></a>

AWS IoT Greengrass uses encryption to protect data while in\-transit \(over the internet or local network\) and at rest \(stored in the AWS Cloud\)\.

Devices in a AWS IoT Greengrass environment often collect data that's sent to AWS services for further processing\. For more information about data encryption on other AWS services, see the security documentation for that service\.

**Topics**
+ [Encryption in transit](encryption-in-transit.md)
+ [Encryption at rest](encryption-at-rest.md)
+ [Key management for the Greengrass core device](key-management.md)