# Deploy secrets to the AWS IoT Greengrass core<a name="secrets"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

AWS IoT Greengrass lets you authenticate with services and applications from Greengrass devices without hard\-coding passwords, tokens, or other secrets\.

AWS Secrets Manager is a service that you can use to securely store and manage your secrets in the cloud\. AWS IoT Greengrass extends Secrets Manager to Greengrass core devices, so your [connectors](connectors.md) and Lambda functions can use local secrets to interact with services and applications\. For example, the Twilio Notifications connector uses a locally stored authentication token\.

To integrate a secret into a Greengrass group, you create a group resource that references the Secrets Manager secret\. This *secret resource* references the cloud secret by ARN\. To learn how to create, manage, and use secret resources, see [Working with secret resources](secrets-using.md)\.

AWS IoT Greengrass encrypts your secrets while in transit and at rest\. During group deployment, AWS IoT Greengrass fetches the secret from Secrets Manager and creates a local, encrypted copy on the Greengrass core\. After you rotate your cloud secrets in Secrets Manager, redeploy the group to propagate the updated values to the core\.

The following diagram shows the high\-level process of deploying a secret to the core\. Secrets are encrypted in transit and at rest\.

![\[AWS IoT Greengrass fetches a secret from AWS Secrets Manager and deploys it as a secret resource to the core device, where it is available to connectors and Lambda functions.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/deploy-local-secret.png)

Using AWS IoT Greengrass to store your secrets locally offers these advantages:
+ **Decoupled from code \(not hard\-coded\)\.** This supports centrally managed credentials and helps protect sensitive data from the risk of compromise\.
+ **Available for offline scenarios\.** Connectors and functions can securely access local services and software when disconnected from the internet\.
+ **Controlled access to secrets\.** Only authorized connectors and functions in the group can access your secrets\. AWS IoT Greengrass uses private key encryption to secure your secrets\. Secrets are encrypted in transit and at rest\. For more information, see [Secrets encryption](#secrets-encryption)\.
+ **Controlled rotation\.** After you rotate your secrets in Secrets Manager, redeploy the Greengrass group to update the local copies of your secrets\. For more information, see [Creating and managing secrets](secrets-using.md#secrets-create-manage)\.
**Important**  
AWS IoT Greengrass doesn't automatically update the values of local secrets after cloud versions are rotated\. To update local values, you must redeploy the group\.

## Secrets encryption<a name="secrets-encryption"></a>

AWS IoT Greengrass encrypts secrets in transit and at rest\.

**Important**  
Make sure that your user\-defined Lambda functions handle secrets securely and don't log any any sensitive data that's stored in the secret\. For more information, see [ Mitigate the Risks of Logging and Debugging Your Lambda Function](https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html#best-practice_lamda-debug-statements) in the *AWS Secrets Manager User Guide*\. Although this documentation specifically refers to rotation functions, the recommendation also applies to Greengrass Lambda functions\.

**Encryption in transit**  
AWS IoT Greengrass uses Transport Layer Security \(TLS\) to encrypt all communication over the internet and local network\. This protects secrets while in transit, which occurs when secrets are retrieved from Secrets Manager and deployed to the core\. For supported TLS cipher suites, see [TLS cipher suites support](gg-sec.md#gg-cipher-suites)\.

**Encryption at rest**  
AWS IoT Greengrass uses the private key specified in [`config.json`](gg-core.md#config-json) for encryption of the secrets that are stored on the core\. For this reason, secure storage of the private key is critical for protecting local secrets\. In the AWS [ shared responsibility model](https://aws.amazon.com/compliance/shared-responsibility-model/), it's the responsibility of the customer to guarantee secure storage of the private key on the core device\.  
AWS IoT Greengrass supports two modes of private key storage:  
+ Using hardware security modules\. For more information, see [Hardware security integration](hardware-security.md)\.
**Note**  
Currently, AWS IoT Greengrass supports only the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism for encryption and decryption of local secrets when using hardware\-based private keys\. If you're following vendor\-provided instructions to manually generate hardware\-based private keys, make sure to choose PKCS\#1 v1\.5\. AWS IoT Greengrass doesn't support Optimal Asymmetric Encryption Padding \(OAEP\)\.
+ Using file system permissions \(default\)\.
The private key is used to secure the data key, which is used to encrypt local secrets\. The data key is rotated with each group deployment\.  
The AWS IoT Greengrass core is the only entity that has access to the private key\. Greengrass connectors or Lambda functions that are affiliated with a secret resource get the value of the secret from the core\.

## Requirements<a name="secrets-reqs"></a>

These are the requirements for local secret support:
+ You must be using AWS IoT Greengrass Core v1\.7 or later\.
+ To get the values of local secrets, your user\-defined Lambda functions must use AWS IoT Greengrass Core SDK v1\.3\.0 or later\.
+ The private key used for local secrets encryption must be specified in the Greengrass configuration file\. By default, AWS IoT Greengrass uses the core private key stored in the file system\. To provide your own private key, see [Specify the private key for secret encryption](#secrets-config-private-key)\. Only the RSA key type is supported\.
**Note**  
Currently, AWS IoT Greengrass supports only the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism for encryption and decryption of local secrets when using hardware\-based private keys\. If you're following vendor\-provided instructions to manually generate hardware\-based private keys, make sure to choose PKCS\#1 v1\.5\. AWS IoT Greengrass doesn't support Optimal Asymmetric Encryption Padding \(OAEP\)\.
+ AWS IoT Greengrass must be granted permission to get your secret values\. This allows AWS IoT Greengrass to fetch the values during group deployment\. If you're using the default Greengrass service role, then AWS IoT Greengrass already has access to secrets with names that start with *greengrass\-*\. To customize access, see [Allow AWS IoT Greengrass to get secret values](#secrets-config-service-role)\.
**Note**  
We recommend that you use this naming convention to identify the secrets that AWS IoT Greengrass is allowed to access, even if you customize permissions\. The console uses different permissions to read your secrets, so it's possible that you can select secrets in the console that AWS IoT Greengrass doesn't have permission to fetch\. Using a naming convention can help avoid a permission conflict, which results in a deployment error\.

## Specify the private key for secret encryption<a name="secrets-config-private-key"></a>

In this procedure, you provide the path to a private key that's used for local secret encryption\. This must be an RSA key with a minimum length of 2048 bits\. For more information about private keys used on the AWS IoT Greengrass core, see [AWS IoT Greengrass core security principals](gg-sec.md#gg-principals)\. 

AWS IoT Greengrass supports two modes of private key storage: hardware\-based or file system\-based \(default\)\. For more information, see [Secrets encryption](#secrets-encryption)\.

**Follow this procedure only** if you want to change the default configuration, which uses the core private key in the file system\. These steps are written with the assumption that you created your group and core as described in [Module 2](module2.md) of the Getting Started tutorial\.

1. Open the [`config.json`](gg-core.md#config-json) file that's located in the `/greengrass-root/config` directory\.
**Note**  
*greengrass\-root* represents the path where the AWS IoT Greengrass Core software is installed on your device\. Typically, this is the `/greengrass` directory\.

1. In the `crypto.principals.SecretsManager` object, for the `privateKeyPath` property, enter the path of the private key:
   + If your private key is stored in the file system, specify the absolute path to the key\. For example:

     ```
     "SecretsManager" : {
       "privateKeyPath" : "file:///somepath/hash.private.key"
     }
     ```
   + If your private key is stored in a hardware security module \(HSM\), specify the path using the [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) URI scheme\. For example:

     ```
     "SecretsManager" : {
       "privateKeyPath" : "pkcs11:object=private-key-label;type=private"
     }
     ```

     For more information, see [Hardware security configuration for an AWS IoT Greengrass core](hardware-security.md#configure-hardware-security)\.
**Note**  
Currently, AWS IoT Greengrass supports only the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism for encryption and decryption of local secrets when using hardware\-based private keys\. If you're following vendor\-provided instructions to manually generate hardware\-based private keys, make sure to choose PKCS\#1 v1\.5\. AWS IoT Greengrass doesn't support Optimal Asymmetric Encryption Padding \(OAEP\)\.

## Allow AWS IoT Greengrass to get secret values<a name="secrets-config-service-role"></a>

In this procedure, you add an inline policy to the Greengrass service role that allows AWS IoT Greengrass to get the values of your secrets\.

**Follow this procedure only** if you want to grant AWS IoT Greengrass custom permissions to your secrets or if your Greengrass service role doesn't include the `AWSGreengrassResourceAccessRolePolicy` managed policy\. `AWSGreengrassResourceAccessRolePolicy` grants access to secrets with names that start with *greengrass\-*\.

1. Run the following CLI command to get the ARN of the Greengrass service role:

   ```
   aws greengrass get-service-role-for-account --region region
   ```

   The returned ARN contains the role name\.

   ```
   {
     "AssociatedAt": "time-stamp",
     "RoleArn": "arn:aws:iam::account-id:role/service-role/role-name"
   }
   ```

   You use the ARN or name in the following step\.

1. Add an inline policy that allows the `secretsmanager:GetSecretValue` action\. For instructions, see [Adding and removing IAM policies ](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.

   You can grant granular access by explicitly listing secrets or using a wildcard `*` naming scheme, or you can grant conditional access to versioned or tagged secrets\. For example, the following policy allows AWS IoT Greengrass to read only the specified secrets\.

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "secretsmanager:GetSecretValue"
               ],
               "Resource": [
                   "arn:aws:secretsmanager:region:account-id:secret:greengrass-SecretA-abc",
                   "arn:aws:secretsmanager:region:account-id:secret:greengrass-SecretB-xyz"
               ]
           }
       ]
   }
   ```
**Note**  
If you use a customer\-managed AWS KMS key to encrypt secrets, your Greengrass service role must also allow the `kms:Decrypt` action\.

For more information about IAM policies for Secrets Manager, see [Authentication and access control for AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/auth-and-access.html) and [Actions, resources, and context keys you can use in an IAM policy or secret policy for AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/reference_iam-permissions.html) in the *AWS Secrets Manager User Guide*\.

## See also<a name="secrets-seealso"></a>
+ [What is AWS Secrets Manager?](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) in the *AWS Secrets Manager User Guide*
+ [PKCS \#1: RSA Encryption Version 1\.5](https://tools.ietf.org/html/rfc2313)