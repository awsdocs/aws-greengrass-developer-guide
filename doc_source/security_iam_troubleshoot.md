# Troubleshooting identity and access issues for AWS IoT Greengrass<a name="security_iam_troubleshoot"></a>

Use the following information to help you diagnose and fix common issues that you might encounter when working with AWS IoT Greengrass and IAM\.

**Topics**
+ [I'm not authorized to perform an action in AWS IoT Greengrass](#security_iam_troubleshoot-no-permissions)
+ [Error: Greengrass is not authorized to assume the Service Role associated with this account, or the error: Failed: TES service role is not associated with this account\.](#troubleshoot-assume-service-role)
+ [Error: Permission denied when attempting to use role arn:aws:iam::<account\-id>:role/<role\-name> to access s3 url https://<region>\-greengrass\-updates\.s3\.<region>\.amazonaws\.com/core/<architecture>/greengrass\-core\-<distribution\-version>\.tar\.gz\.](#troubleshoot-ota-region-access)
+ [Device shadow does not sync with the cloud\.](#troubleshoot-shadow-sync)
+ [I'm not authorized to perform iam:PassRole](#security_iam_troubleshoot-passrole)
+ [I'm an administrator and want to allow others to access AWS IoT Greengrass](#security_iam_troubleshoot-admin-delegate)
+ [I want to allow people outside of my AWS account to access my AWS IoT Greengrass resources](#security_iam_troubleshoot-cross-account-access)

For general troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## I'm not authorized to perform an action in AWS IoT Greengrass<a name="security_iam_troubleshoot-no-permissions"></a>

If you receive an error that states you're not authorized to perform an action, you must contact your administrator for assistance\. Your administrator is the person who provided you with your user name and password\.

The following example error occurs when the `mateojackson` IAM user tries to view details about a core definition version, but does not have `greengrass:GetCoreDefinitionVersion` permissions\.

```
User: arn:aws:iam::123456789012:user/mateojackson is not authorized to perform: greengrass:GetCoreDefinitionVersion on resource: resource: arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/cores/78cd17f3-bc68-ee18-47bd-5bda5EXAMPLE/versions/368e9ffa-4939-6c75-859c-0bd4cEXAMPLE
```

In this case, Mateo asks his administrator to update his policies to allow him to access the `arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/cores/78cd17f3-bc68-ee18-47bd-5bda5EXAMPLE/versions/368e9ffa-4939-6c75-859c-0bd4cEXAMPLE` resource using the `greengrass:GetCoreDefinitionVersion` action\.

## Error: Greengrass is not authorized to assume the Service Role associated with this account, or the error: Failed: TES service role is not associated with this account\.<a name="troubleshoot-assume-service-role"></a>

**Solution:** You might see this error when the deployment fails\. Check that a Greengrass service role is associated with your AWS account in the current AWS Region\. For more information, see [Managing the Greengrass service role \(CLI\)](service-role.md#manage-service-role-cli) or [Managing the Greengrass service role \(console\)](service-role.md#manage-service-role-console)\.

## Error: Permission denied when attempting to use role arn:aws:iam::<account\-id>:role/<role\-name> to access s3 url https://<region>\-greengrass\-updates\.s3\.<region>\.amazonaws\.com/core/<architecture>/greengrass\-core\-<distribution\-version>\.tar\.gz\.<a name="troubleshoot-ota-region-access"></a>

**Solution:** You might see this error when an over\-the\-air \(OTA\) update fails\. In the signer role policy, add the target AWS Region as a `Resource`\. This signer role is used to presign the S3 URL for the AWS IoT Greengrass software update\. For more information, see [S3 URL signer role](core-ota-update.md#s3-url-signer-role)\.

## Device shadow does not sync with the cloud\.<a name="troubleshoot-shadow-sync"></a>

**Solution:** Make sure that AWS IoT Greengrass has permissions for `iot:UpdateThingShadow` and `iot:GetThingShadow` actions in the [Greengrass service role](service-role.md)\. If the service role uses the `AWSGreengrassResourceAccessRolePolicy` managed policy, these permissions are included by default\.

See [Troubleshooting shadow synchronization timeout issues](gg-troubleshooting.md#troubleshooting-shadow-sync)\.

The following are general IAM issues that you might encounter when working with AWS IoT Greengrass\.

## I'm not authorized to perform iam:PassRole<a name="security_iam_troubleshoot-passrole"></a>

If you receive an error that you're not authorized to perform the `iam:PassRole` action, then you must contact your administrator for assistance\. Your administrator is the person that provided you with your user name and password\. Ask that person to update your policies to allow you to pass a role to AWS IoT Greengrass\.

Some AWS services allow you to pass an existing role to that service, instead of creating a new service role or service\-linked role\. To do this, you must have permissions to pass the role to the service\.

The following example error occurs when an IAM user named `marymajor` tries to use the console to perform an action in AWS IoT Greengrass\. However, the action requires the service to have permissions granted by a service role\. Mary does not have permissions to pass the role to the service\.

```
User: arn:aws:iam::123456789012:user/marymajor is not authorized to perform: iam:PassRole
```

In this case, Mary asks her administrator to update her policies to allow her to perform the `iam:PassRole` action\.

## I'm an administrator and want to allow others to access AWS IoT Greengrass<a name="security_iam_troubleshoot-admin-delegate"></a>

To allow others to access AWS IoT Greengrass, you must create an IAM entity \(user or role\) for the person or application that needs access\. They will use the credentials for that entity to access AWS\. You must then attach a policy to the entity that grants them the correct permissions in AWS IoT Greengrass\.

To get started right away, see [Creating Your First IAM Delegated User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-delegated-user.html) in the *IAM User Guide*\.

## I want to allow people outside of my AWS account to access my AWS IoT Greengrass resources<a name="security_iam_troubleshoot-cross-account-access"></a>

You can create an IAM role that users in other accounts or people outside of your organization can use to access your AWS resources\. You can specify the who is trusted to assume the role\. For more information, see [Providing access to an IAM user in another AWS account that you own](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_aws-accounts.html) and [Providing access to AWS accounts owned by third parties](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_third-party.html) in the *IAM User Guide*\.

AWS IoT Greengrass doesn't support cross\-account access based on resource\-based policies or access control lists \(ACLs\)\.