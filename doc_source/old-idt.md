# Using IDT for Greengrass 1\.6\.1 and Earlier<a name="old-idt"></a>

If you are using a version of IDT for AWS IoT Greengrass earlier than 1\.7\.0, you must install the AWS Command Line Interface \(CLI\) and create a AWS IoT Greengrass service role\. In versions 1\.7\.1 or later, IDT creates the service role for you\.

## Install the AWS Command Line Interface \(CLI\)<a name="install-cli"></a>

You use the CLI to perform some operations\. If you don't have the CLI installed, follow the instructions in [Install the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)\.

Configure the CLI for the AWS Region you want to use by running aws configure from a command line\. For information about the AWS Regions that support IDT for AWS IoT Greengrass, see [AWS Regions and Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#amazon-freertos-ota-control)\.

## Configure the AWS IoT Greengrass Service Role<a name="config-gg-role"></a>

To successfully deploy an AWS IoT Greengrass group, AWS IoT Greengrass requires permissions to perform actions on your behalf\. If you are using IDT v1\.1 for AWS IoT Greengrass or later, IDT creates and associates the service role with your AWS account for you\. If you are using IDT v1\.0 for AWS IoT Greengrass, follow these instructions to create and associate the service role with your AWS account\.

**To create the AWS IoT Greengrass service role**

1. Use the following AWS CLI command on your host computer to create a service role\.

   ```
   aws iam create-role --role-name GreengrassServiceRole --assume-role-policy-document '{ 
   	"Version": "2012-10-17",
   	"Statement": [{
   		"Effect": "Allow",
   		"Principal": {
   			"Service": "greengrass.amazonaws.com"
   		},
   		"Action": "sts:AssumeRole"
   	}]
   }'
   ```

   This command generates a service role ARN that you use when you associate your AWS IoT Greengrass service role with your AWS account\.

1. Use the following AWS CLI command to attach the `AWSGreengrassResourceAccessRolePolicy` to your AWS IoT Greengrass service role\.

   ```
   aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/service-role/AWSGreengrassResourceAccessRolePolicy --role-name GreengrassServiceRole
   ```

1. Use the following AWS CLI command to associate your AWS IoT Greengrass service role with your AWS account\.

   ```
   aws greengrass associate-service-role-to-account --role-arn <your-greengrass-service-role-arn>
   ```