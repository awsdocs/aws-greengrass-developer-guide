# Prerequisites for running IDT tests<a name="dev-tst-prereqs"></a>

This section describes the prerequisites for using AWS IoT Device Tester \(IDT\) for AWS IoT Greengrass\.

## Download the latest version of AWS IoT Device Tester for AWS IoT Greengrass<a name="install-dev-tst-gg"></a>

Download the latest version of IDT from [Supported versions of AWS IoT Device Tester for AWS IoT Greengrass](dev-test-versions.md)\. Extract the software into a location on your file system where you have read and write permissions\. 

**Note**  
<a name="unzip-package-to-local-drive"></a>IDT does not support being run by multiple users from a shared location, such as an NFS directory or a Windows network shared folder\. Doing so may result in crashes or data corruption\. We recommend that you extract the IDT package to a local drive and run the IDT binary on your local workstation\.  
Windows has a path length limitation of 260 characters\. If you are using Windows, extract IDT to a root directory like `C:\ ` or `D:\` to keep your paths under the 260 character limit\.

## Create and configure an AWS account<a name="config-aws-account-for-idt"></a>

Before you can use IDT for AWS IoT Greengrass, you must create an AWS account and configure permissions that IDT needs while running tests\. The permissions allow IDT to access AWS services and create AWS resources, such as AWS IoT things, Greengrass groups, and Lambda functions, on your behalf\.

<a name="idt-aws-credentials"></a>To create these resources, IDT for AWS IoT Greengrass uses the AWS credentials configured in the `config.json` file to make API calls on your behalf\. These resources are provisioned at various times during a test\.

**Note**  
Although most tests qualify for [AWS Free Tier](http://aws.amazon.com/free), you must provide a credit card when you sign up for an AWS account\. For more information, see [ Why do I need a payment method if my account is covered by the Free Tier?](http://aws.amazon.com/premiumsupport/knowledge-center/free-tier-payment-method/)\.

### Step 1: Create an AWS account<a name="create-aws-account-for-idt"></a>

In this step, create and configure an AWS account\. If you already have an AWS account, skip to [Step 2: Configure permissions for IDT](#configure-idt-permissions)\.<a name="create-aws-account-steps"></a>

1. Open the [AWS home page](https://aws.amazon.com/), and choose **Create an AWS Account**\.
**Note**  
If you've signed in to AWS recently, you might see **Sign In to the Console** instead\.

1. Follow the online instructions\. Part of the sign\-up procedure includes registering a credit card, receiving a text message or phone call, and entering a PIN\.

   For more information, see [How do I create and activate a new Amazon Web Services account?](http://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)

### Step 2: Configure permissions for IDT<a name="configure-idt-permissions"></a>

In this step, configure the permissions that IDT for AWS IoT Greengrass uses to run tests and collect IDT usage data\. You can use the AWS Management Console or AWS Command Line Interface \(AWS CLI\) to create an IAM policy and a test user for IDT, and then attach policies to the user\. If you already created a test user for IDT, skip to [Configuring your device](device-config-setup.md) or [Optional: Configuring your Docker container for IDT for AWS IoT Greengrass](docker-config-setup.md)\.
+ [To Configure Permissions for IDT \(Console\)](#configure-idt-permissions-console)
+ [To Configure Permissions for IDT \(AWS CLI\)](#configure-idt-permissions-cli)<a name="configure-idt-permissions-console"></a>

**To configure permissions for IDT \(console\)**

Follow these steps to use the console to configure permissions for IDT for AWS IoT Greengrass\.

1. Sign in to the [IAM console](https://console.aws.amazon.com/iam)\.

1. Create a customer managed policy that grants permissions to create roles with specific permissions\. 

   1. In the navigation pane, choose **Policies**, and then choose **Create policy**\.

   1. On the **JSON** tab, replace the placeholder content with the following policy\.

      ```
      <a name="customer-managed-policy-cli"></a>{
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

   1. Create an IAM user\. Follow steps 1 through 5 in [Creating IAM users \(console\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console) in the *IAM User Guide*\.

   1. Attach the permissions to your IAM user:

      1. On the **Set permissions ** page, choose **Attach existing policies to user directly**\.

      1. Search for the **IDTGreengrassIAMPermissions** policy that you created in the previous step\. Select the check box\.

      1. Search for the **AWSIoTDeviceTesterForGreengrassFullAccess** policy\. Select the check box\.
**Note**  <a name="AWSIoTDeviceTesterForGreengrassFullAccess"></a>
The [AWSIoTDeviceTesterForGreengrassFullAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSIoTDeviceTesterForGreengrassFullAccess) is an AWS managed policy that defines the permissions IDT requires to create and access AWS resources used for testing\. For more information, see [AWS managed policy for AWS IoT Device Tester](#idt-managed-policy)\.

   1. Choose **Next: Tags**\.

   1. Choose **Next: Review** to view a summary of your choices\.

   1. Choose **Create user**\.

   1. To view the user's access keys \(access key IDs and secret access keys\), choose **Show** next to the password and access key\. To save the access keys, choose **Download\.csv** and save the file to a secure location\. You use this information later to configure your AWS credentials file\.

1. <a name="aws-account-config-next-steps"></a>Next step: Configure your [physical device](device-config-setup.md)\.

 <a name="configure-idt-permissions-cli"></a>

**To configure permissions for IDT \(AWS CLI\)**

Follow these steps to use the AWS CLI to configure permissions for IDT for AWS IoT Greengrass\. If you already configured permissions in the console, skip to [Configuring your device](device-config-setup.md) or [Optional: Configuring your Docker container for IDT for AWS IoT Greengrass](docker-config-setup.md)\.

1. On your computer, install and configure the AWS CLI if it's not already installed\. Follow the steps in [ Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) in the *AWS Command Line Interface User Guide*\.
**Note**  
The AWS CLI is an open source tool that you can use to interact with AWS services from your command\-line shell\.

1. Create a customer managed policy that grants permissions to manage IDT and AWS IoT Greengrass roles\.

------
#### [ Linux, macOS, or Unix ]

   ```
   aws iam create-policy --policy-name IDTGreengrassIAMPermissions --policy-document '<a name="customer-managed-policy-cli"></a>{
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
   }'
   ```

------
#### [ Windows command prompt ]

   ```
   aws iam create-policy --policy-name IDTGreengrassIAMPermissions --policy-document '{\"Version\": \"2012-10-17\", \"Statement\": [{\"Sid\": \"ManageRolePoliciesForIDTGreengrass\",\"Effect\": \"Allow\",\"Action\": [\"iam:DetachRolePolicy\", \"iam:AttachRolePolicy\"], \"Resource\": [\"arn:aws:iam::*:role/idt-*\",\"arn:aws:iam::*:role/GreengrassServiceRole\"],\"Condition\": {\"ArnEquals\": {\"iam:PolicyARN\": [\"arn:aws:iam::aws:policy/service-role/AWSGreengrassResourceAccessRolePolicy\",\"arn:aws:iam::aws:policy/service-role/GreengrassOTAUpdateArtifactAccess\",\"arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole\"]}}},{\"Sid\": \"ManageRolesForIDTGreengrass\",\"Effect\": \"Allow\",\"Action\": [\"iam:CreateRole\",\"iam:DeleteRole\", \"iam:PassRole\", \"iam:GetRole\"],\"Resource\": [\"arn:aws:iam::*:role/idt-*\",\"arn:aws:iam::*:role/GreengrassServiceRole\"]}]}'
   ```

**Note**  
This step includes a Windows command prompt example because it uses a different JSON syntax than Linux, macOS, or Unix terminal commands\.

------

1. Create an IAM user and attach the permissions required by IDT for AWS IoT Greengrass\.

   1. Create an IAM user\. In this example setup, the user is named `IDTGreengrassUser`\.

      ```
      aws iam create-user --user-name IDTGreengrassUser
      ```

   1. Attach the `IDTGreengrassIAMPermissions` policy you created in step 2 to your IAM user\. Replace *<account\-id>* in the command with the ID of your AWS account\.

      ```
      aws iam attach-user-policy --user-name IDTGreengrassUser --policy-arn arn:aws:iam::<account-id>:policy/IDTGreengrassIAMPermissions
      ```

   1. Attach the `AWSIoTDeviceTesterForGreengrassFullAccess` policy to your IAM user\.

      ```
      aws iam attach-user-policy --user-name IDTGreengrassUser --policy-arn arn:aws:iam::aws:policy/AWSIoTDeviceTesterForGreengrassFullAccess
      ```
**Note**  <a name="AWSIoTDeviceTesterForGreengrassFullAccess"></a>
The [AWSIoTDeviceTesterForGreengrassFullAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSIoTDeviceTesterForGreengrassFullAccess) is an AWS managed policy that defines the permissions IDT requires to create and access AWS resources used for testing\. For more information, see [AWS managed policy for AWS IoT Device Tester](#idt-managed-policy)\.

1. Create a secret access key for the user\.

   ```
   aws iam create-access-key --user-name IDTGreengrassUser
   ```

   Store the output in a secure location\. You use this information later to configure your AWS credentials file\.

1. <a name="aws-account-config-next-steps"></a>Next step: Configure your [physical device](device-config-setup.md)\.

## AWS managed policy for AWS IoT Device Tester<a name="idt-managed-policy"></a>

The [AWSIoTDeviceTesterForGreengrassFullAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSIoTDeviceTesterForGreengrassFullAccess) managed policy allows IDT to run operations and collect usage metrics\. This policy grants the following IDT permissions:
+ `iot-device-tester:CheckVersion`\. Check whether a set of AWS IoT Greengrass, test suite, and IDT versions are compatible\.
+ `iot-device-tester:DownloadTestSuite`\. Download test suites\.
+ `iot-device-tester:LatestIdt`\. Get information about the latest IDT version that is available for download\.
+ `iot-device-tester:SendMetrics`\. Publish usage data that IDT collects about your tests\.
+ `iot-device-tester:SupportedVersion`\. Get the list of AWS IoT Greengrass and test suite versions that are supported by IDT\. This information is displayed in the command\-line window\.