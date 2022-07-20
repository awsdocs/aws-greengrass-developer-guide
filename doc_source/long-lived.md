--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# Configure long\-lived Lambda functions for AWS IoT Greengrass<a name="long-lived"></a>

You are now ready to configure your Lambda function for AWS IoT Greengrass\.

1. <a name="console-gg-groups"></a>In the AWS IoT console navigation pane, under **Manage**, expand **Greengrass devices**, and then choose **Groups \(V1\)**\.

1. Under **Greengrass groups**, choose the group that you created in [Module 2](module2.md)\.

1. On the group configuration page, choose the **Lambda functions** tab, and then under **My Lambda functions**, choose **Add**\.

1. For **Lambda function**, choose **Greengrass\_HelloWorld\_Counter**\.

1. For **Lambda function version**, choose the alias to the version that you published\.

1. For **Timeout \(seconds\)**, enter **25**\. This Lambda function sleeps for 20 seconds before each invocation\.

1. For **Pinned**, choose **True**\.

1. Keep the default values for all other fields, and choose **Add Lambda function**\.