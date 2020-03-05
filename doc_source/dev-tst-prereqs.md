# Prerequisites<a name="dev-tst-prereqs"></a>

This section describes the prerequisites for using IDT for AWS IoT Greengrass\.

## Download the Latest Version of AWS IoT Device Tester for AWS IoT Greengrass<a name="install-dev-tst-gg"></a>

Download the latest version of IDT from [Supported Versions of AWS IoT Device Tester for AWS IoT Greengrass](dev-test-versions.md)\. Extract the software into a location on your file system where you have read and write permissions\. 

Windows has a path length limitation of 260 characters\. If you are using Windows, extract IDT for AWS IoT Greengrass into a root directory like `C:\ ` or `D:\` to keep your paths under the 260 character limit\.

## Create and Configure an AWS Account<a name="config-aws-account-for-idt"></a>

Before you can use IDT for AWS IoT Greengrass, you must create an AWS account and configure permissions\. This is required so IDT can access AWS services and create AWS resources on your behalf, such as AWS IoT things, Greengrass groups, and Lambda functions\.

<a name="idt-aws-credentials"></a>To create these resources, IDT for AWS IoT Greengrass uses the AWS credentials configured in the `config.json` file to make API calls on your behalf\. These resources are provisioned at various times during a test\.

**Note**  
Although most tests qualify for [AWS Free Tier](http://aws.amazon.com/free), you must provide a credit card when you sign up for an AWS account\. For more information, see [ Why do I need a payment method if my account is covered by the Free Tier?](http://aws.amazon.com/premiumsupport/knowledge-center/free-tier-payment-method/)\.

Follow these steps to create and configure an AWS account, IAM user, and IAM policy that grants IDT for AWS IoT Greengrass permission to access resources on your behalf while running tests\.

### Step 1: Create an AWS Account<a name="create-aws-account-for-idt"></a><a name="create-aws-account-steps"></a>

1. Open the [AWS home page](https://aws.amazon.com/), and choose **Create an AWS Account**\.
**Note**  
If you've signed in to AWS recently, you might see **Sign In to the Console** instead\.

1. Follow the online instructions\. Part of the sign\-up procedure includes registering a credit card, receiving a text message or phone call, and entering a PIN\.

   For more information, see [How do I create and activate a new Amazon Web Services account?](http://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)

### Step 2: Configure Permissions for IDT<a name="configure-idt-permissions"></a>

In this step, you configure the permissions that IDT for AWS IoT Greengrass uses to run tests and collect [IDT usage data](#usage-metrics)\. You create an IAM policy and an IAM user, and then attach policies to the user\.

1. Sign in to the [IAM console](https://console.aws.amazon.com/iam)\.

1. Create a customer managed policy that grants permissions to create roles with specific permissions\.

   1. In the navigation pane, choose **Policies**, and then choose **Create policy**\.

   1. On the **JSON** tab, replace the placeholder content with the following policy\.

      ```
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "ManageRolePoliciesForIDTGreengrass",
                  "Effect": "Allow",
                  "Action": [
                      "iam:DetachRolePolicy",
                      "iam:AttachRolePolicy"
                  ],
                  "Resource": [
                      "arn:aws:iam::*:role/idt-*",
                      "arn:aws:iam::*:role/GreengrassServiceRole"
                  ],
                  "Condition": {
                      "ArnEquals": {
                          "iam:PolicyARN": [
                              "arn:aws:iam::aws:policy/service-role/AWSGreengrassResourceAccessRolePolicy",
                              "arn:aws:iam::aws:policy/service-role/GreengrassOTAUpdateArtifactAccess",
                              "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
                          ]
                      }
                  }
              },
              {
                  "Sid": "ManageRolesForIDTGreengrass",
                  "Effect": "Allow",
                  "Action": [
                      "iam:CreateRole",
                      "iam:DeleteRole",
                      "iam:PassRole",
                      "iam:GetRole"
                  ],
                  "Resource": [
                      "arn:aws:iam::*:role/idt-*",
                      "arn:aws:iam::*:role/GreengrassServiceRole"
                  ]
              }
          ]
      }
      ```
**Important**  <a name="policy-grants-role-perms"></a>
The following policy grants permission to create and manage roles required by IDT for AWS IoT Greengrass\. This includes permissions to attach the following AWS managed policies:  
[AWSGreengrassResourceAccessRolePolicy](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/service-role/AWSGreengrassResourceAccessRolePolicy)
[GreengrassOTAUpdateArtifactAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/service-role/GreengrassOTAUpdateArtifactAccess)
[AWSLambdaBasicExecutionRole](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole)

   1. Choose **Review policy**\.

   1. For **Name**, enter **IDTGreengrassIAMPermissions**\. Under **Summary**, review the permissions granted by your policy\.

   1. Choose **Create policy**\.

1. Create an IAM user and attach the permissions required by IDT for AWS IoT Greengrass\.

   1. Create an IAM user\. Follow steps 1 through 5 in [Creating IAM Users \(Console\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console) in the *IAM User Guide*\.

   1. Attach the permissions to your IAM user:

      1. On the **Set permissions ** page, choose **Attach existing policies to user directly**\.

      1. Search for the **IDTGreengrassIAMPermissions** policy that you created in the previous step\. Select the check box\.

      1. Search for the **AWSIoTDeviceTesterForGreengrassFullAccess** policy\. Select the check box\.
**Note**  <a name="AWSIoTDeviceTesterForGreengrassFullAccess"></a>
The [AWSIoTDeviceTesterForGreengrassFullAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSIoTDeviceTesterForGreengrassFullAccess) is an AWS managed policy that defines the permissions IDT requires to create and access AWS resources used for testing\.

   1. Choose **Next: Tags**\.

   1. Choose **Next: Review** to view a summary of your choices\.

   1. Choose **Create user**\.

   1. To view the user's access keys \(access key IDs and secret access keys\), choose **Show** next to the password and access key\. To save the access keys, choose **Download\.csv** and save the file to a secure location\. You use this information later to configure your AWS credentials file\.

1. <a name="aws-account-config-next-steps"></a>Next step: Configure your [physical device](device-config-setup.md) or [Docker container](docker-config-setup.md)\.

## AWS IoT Device Tester Usage Metrics<a name="usage-metrics"></a>

When you run IDT for AWS IoT Greengrass, IDT collects usage data about your tests\. The `iot-device-tester:SendMetrics` permission allows IDT to send the usage data\. This permission is present in the `AWSIoTDeviceTesterForGreengrassFullAccess` managed policy\.