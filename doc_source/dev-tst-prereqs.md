# Prerequisites<a name="dev-tst-prereqs"></a>

This section describes the prerequisites for using IDT for AWS IoT Greengrass\.

## Download the Latest Version of AWS IoT Device Tester for AWS IoT Greengrass<a name="install-dev-tst-gg"></a>

Download the latest version of IDT from [AWS IoT Device Tester for AWS IoT Greengrass Versions](dev-test-versions.md)\. Extract the software into a location on your file system where you have read and write permissions\. Windows has a path length limitation of 260 characters\. If you are using Windows, extract IDT for AWS IoT Greengrass into a root directory like `C:\ ` or `D:\` to keep your paths under the 260 character limit\.

## Create and Configure an AWS Account<a name="config-aws-account"></a>

Follow these steps to create and configure an AWS account, create an IAM user, and create an IAM policy that grants IDT for AWS IoT Greengrass permission to access resources on your behalf while running tests\.

1. [Create an AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)\.

1. [Create an IAM policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_managed-policies.html) that grants IDT for AWS IoT Greengrass the permissions it needs to run the tests\. Use the policy templates described in [Permissions Policy Template](policy-template.md)\.

1. [Create an IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) in your AWS account and attach the policy you just created\.

If you are using AWS IoT Greengrass 1\.6\.1 or earlier, see [Using IDT for Greengrass 1\.6\.1 and Earlier](old-idt.md)\.