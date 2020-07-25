# Working with secret resources<a name="secrets-using"></a>

AWS IoT Greengrass uses *secret resources* to integrate secrets from AWS Secrets Manager into a Greengrass group\. A secret resource is a reference to a Secrets Manager secret\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\.

On the AWS IoT Greengrass core device, connectors and Lambda functions can use the secret resource to authenticate with services and applications, without hard\-coding passwords, tokens, or other credentials\.

## Creating and managing secrets<a name="secrets-create-manage"></a>

In a Greengrass group, a secret resource references the ARN of a Secrets Manager secret\. When the secret resource is deployed to the core, the value of the secret is encrypted and made available to affiliated connectors and Lambda functions\. For more information, see [Secrets encryption](secrets.md#secrets-encryption)\.

You use Secrets Manager to create and manage the cloud versions of your secrets\. You use AWS IoT Greengrass to create, manage, and deploy your secret resources\.

**Important**  
We recommend that you follow the best practice of rotating your secrets in Secrets Manager\. Then, deploy the Greengrass group to update the local copies of your secrets\. For more information, see [ Rotating your AWS Secrets Manager secrets](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html) in the *AWS Secrets Manager User Guide*\.

**To make a secret available on the Greengrass core**

1. Create a secret in Secrets Manager\. This is the cloud version of your secret, which is centrally stored and managed in Secrets Manager\. Management tasks include rotating secret values and applying resource policies\.

1. Create a secret resource in AWS IoT Greengrass\. This is a type of group resource that references the cloud secret by ARN\. You can reference a secret only once per group\.

1. Configure your connector or Lambda function\. You must affiliate the resource with a connector or function by specifying corresponding parameters or properties\. This allows them to get the value of the locally deployed secret resource\. For more information, see [Using local secrets in connectors and Lambda functions](#secrets-access)\.

1. Deploy the Greengrass group\. During deployment, AWS IoT Greengrass fetches the value of the cloud secret and creates \(or updates\) the local secret on the core\.

Secrets Manager logs an event in AWS CloudTrail each time that AWS IoT Greengrass retrieves a secret value\. AWS IoT Greengrass doesn't log any events related to the deployment or usage of local secrets\. For more information about Secrets Manager logging, see [ Monitor the use of your AWS Secrets Manager secrets](https://docs.aws.amazon.com/secretsmanager/latest/userguide/monitoring.html) in the *AWS Secrets Manager User Guide*\.

### Including staging labels in secret resources<a name="secret-resources-labels"></a>

Secrets Manager uses staging labels to identify specific versions of a secret value\. Staging labels can be system\-defined or user\-defined\. Secrets Manager assigns the `AWSCURRENT` label to the most recent version of the secret value\. Staging labels are commonly used to manage secrets rotation\. For more information about Secrets Manager versioning, see [ Key terms and concepts for AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/terms-concepts.html) in the *AWS Secrets Manager User Guide*\.

Secret resources always include the `AWSCURRENT` staging label, and they can optionally include other staging labels if they're required by a Lambda function or connector\. During group deployment, AWS IoT Greengrass retrieves the values of the staging labels that are referenced in the group, and then creates or updates the corresponding values on the core\.

### Create and manage secret resources \(console\)<a name="create-manage-secret-resource-console"></a>

#### Creating secret resources \(console\)<a name="create-manage-secret-resource-console-create"></a>

In the AWS IoT Greengrass console, you create and manage secret resources from the **Secrets** tab on the group's **Resources** page\. For tutorials that create a secret resource and add it to a group, see [How to create a secret resource \(console\)](secrets-console.md) and [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.

![\[A secret resource on the Secret tab on the Resources page.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/connectors/secret-resource-twilio-auth-token.png)

**Note**  
Alternatively, the console allows you to create a secret and secret resource when you configure a connector or Lambda function\. You can do this from the connector's **Configure parameters** page or the Lambda function's **Resources** page\.

#### Managing secret resources \(console\)<a name="create-manage-secret-resource-console-manage"></a>

Management tasks for the secret resources in your Greengrass group include adding secret resources to the group, removing secret resources from the group, and changing the set of [staging labels](#secret-resources-labels) that are included in a secret resource\.

If you point to a different secret from Secrets Manager, you must also edit any connectors that use the secret:

1. On the group configuration page, choose **Connectors**\.

1. From the connector's contextual menu, choose **Edit**\.

1. The **Edit parameters** page displays a message to inform you that the secret ARN changed\. To confirm the change, choose **Save**\.

If you delete a secret in Secrets Manager, remove the corresponding secret resource from the group and from connectors and Lambda functions that reference it\. Otherwise, during group deployment, AWS IoT Greengrass returns an error that the secret can't be found\. Also update your Lambda function code as needed\.

### Create and manage secret resources \(CLI\)<a name="create-manage-secret-resource-cli"></a>

#### Creating secret resources \(CLI\)<a name="create-manage-secret-resource-cli-create"></a>

In the AWS IoT Greengrass API, a secret is a type of group resource\. The following example creates a resource definition with an initial version that includes a secret resource named `MySecretResource`\. For a tutorial that creates a secret resource and adds it to a group version, see [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)\.

The secret resource references the ARN of the corresponding Secrets Manager secret and includes two staging labels in addition to `AWSCURRENT`, which is always included\.

```
aws greengrass create-resource-definition --name MyGreengrassResources --initial-version '{
    "Resources": [
        {
            "Id": "my-resource-id",
            "Name": "MySecretResource",
            "ResourceDataContainer": {
                "SecretsManagerSecretResourceData": {
                    "ARN": "arn:aws:secretsmanager:us-west-2:123456789012:secret:greengrass-SomeSecret-KUj89s",
                    "AdditionalStagingLabelsToDownload": [
                        "Label1",
                        "Label2"
                    ]
                }
            }
        }
    ]
}'
```

#### Managing secret resources \(CLI\)<a name="create-manage-secret-resource-cli-manage"></a>

Management tasks for the secret resources in your Greengrass group include adding secret resources to the group, removing secret resources from the group, and changing the set of [staging labels](#secret-resources-labels) that are included in a secret resource\.

In the AWS IoT Greengrass API, these changes are implemented by using versions\.

The AWS IoT Greengrass API uses versions to manage groups\. Versions are immutable, so to add or change group components—for example, the group's devices, functions, and resources—you must create versions of new or updated components\. Then, you create and deploy a group version that contains the target version of each component\. To learn more about groups, see [AWS IoT Greengrass groups](what-is-gg.md#gg-group)\.

For example, to change the set of staging labels for a secret resource:

1. Create a resource definition version that contains the updated secret resource\. The following example adds a third staging label to the secret resource from the previous section\.
**Note**  
To add more resources to the version, include them in the `Resources` array\.

   ```
   aws greengrass create-resource-definition --name MyGreengrassResources --initial-version '{
       "Resources": [
           {
               "Id": "my-resource-id",
               "Name": "MySecretResource",
               "ResourceDataContainer": {
                   "SecretsManagerSecretResourceData": {
                       "ARN": "arn:aws:secretsmanager:us-west-2:123456789012:secret:greengrass-SomeSecret-KUj89s",
                       "AdditionalStagingLabelsToDownload": [
                           "Label1",
                           "Label2",
                           "Label3"
                       ]
                   }
               }
           }
       ]
   }'
   ```

1. If the ID of the secret resource is changed, update connectors and functions that use the secret resource\. In the new versions, update the parameter or property that corresponds to the resource ID\. If the ARN of the secret is changed, you must also update the corresponding parameter for any connectors that use the secret\.
**Note**  
The resource ID is an arbitrary identifier that's provided by the customer\.

1. Create a group version that contains the target version of each component that you want to send to the core\.

1. Deploy the group version\.

For a tutorial that shows how to create and deploy secret resources, connectors, and functions, see [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)\.

If you delete a secret in Secrets Manager, remove the corresponding secret resource from the group and from connectors and Lambda functions that reference it\. Otherwise, during group deployment, AWS IoT Greengrass returns an error that the secret can't be found\. Also update your Lambda function code as needed\. You can remove a local secret by deploying a resource definition version that doesn't contain the corresponding secret resource\.

## Using local secrets in connectors and Lambda functions<a name="secrets-access"></a>

Greengrass connectors and Lambda functions use local secrets to interact with services and applications\. The `AWSCURRENT` value is used by default, but values for other [ staging labels](#secret-resources-labels) included in the secret resource are also available\.

Connectors and functions must be configured before they can access local secrets\. This affiliates the secret resource with connector or function\.

**Connectors**  
If a connector requires access to a local secret, it provides parameters that you configure with the information it needs to access the secret\.  
+ To learn how to do this in the AWS IoT Greengrass console, see [Getting started with Greengrass connectors \(console\)](connectors-console.md)\.
+ To learn how to do this with the AWS IoT Greengrass CLI, see [Getting started with Greengrass connectors \(CLI\)](connectors-cli.md)\.
For information about requirements for individual connectors, see [AWS\-provided Greengrass connectors](connectors-list.md)\.  
The logic for accessing and using the secret is built into the connector\.

**Lambda functions**  
To allow a Greengrass Lambda function to access a local secret, you configure the function's properties\.  
+ To learn how to do this in the AWS IoT Greengrass console, see [How to create a secret resource \(console\)](secrets-console.md)\.
+ To do this in the AWS IoT Greengrass API, you provide the following information in the `ResourceAccessPolicies` property\.
  + `ResourceId`: The ID of the secret resource in the Greengrass group\. This is the resource that references the ARN of the corresponding Secrets Manager secret\.
  + `Permission`: The type of access that the function has to the resource\. Only `ro` \(read\-only\) permission is supported for secret resources\.

  The following example creates a Lambda function that can access the `MyApiKey` secret resource\.

  ```
  aws greengrass create-function-definition --name MyGreengrassFunctions --initial-version '{
      "Functions": [
          {
              "Id": "MyLambdaFunction",
              "FunctionArn": "arn:aws:lambda:us-west-2:123456789012:function:myFunction:1",
              "FunctionConfiguration": {
                  "Pinned": false,
                  "MemorySize": 16384,
                  "Timeout": 10,
                  "Environment": {
                      "ResourceAccessPolicies": [
                          {
                              "ResourceId": "MyApiKey",
                              "Permission": "ro"
                          }                          
                      ],
                      "AccessSysfs": true
                  }
              }
          }
      ]
  }'
  ```

   

  To access local secrets at runtime, Greengrass Lambda functions call the `get_secret_value` function from the `secretsmanager` client in the AWS IoT Greengrass Core SDK \(v1\.3\.0 or later\)\.

  The following example shows how to use the AWS IoT Greengrass Core SDK for Python to get a secret\. It passes the name of the secret to the `get_secret_value` function\. `SecretId` can be the name or ARN of the Secrets Manager secret \(not the secret resource\)\.

  ```
  import greengrasssdk
   
  # Creating a Greengrass Core SDK client
  client = greengrasssdk.client('secretsmanager')
   
  # This handler is called when the function is invoked
  # It uses the secretsmanager client to get the value of a secret
  def function_handler(event, context):
      response = client.get_secret_value(SecretId='greengrass-MySecret-abc')
      raw_secret = response.get('SecretString')
  ```

  For text type secrets, the `get_secret_value` function returns a string\. For binary type secrets, it returns a base64\-encoded string\.
**Important**  
Make sure that your user\-defined Lambda functions handle secrets securely and don't log any any sensitive data that's stored in the secret\. For more information, see [ Mitigate the Risks of Logging and Debugging Your Lambda Function](https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html#best-practice_lamda-debug-statements) in the *AWS Secrets Manager User Guide*\. Although this documentation specifically refers to rotation functions, the recommendation also applies to Greengrass Lambda functions\.

  The current value of the secret is returned by default\. This is the version that the `AWSCURRENT` staging label is attached to\. To access a different version, pass the name of the corresponding staging label for the optional `VersionStage` argument\. For example:

  ```
  import greengrasssdk
   
  # Creating a greengrass core sdk client
  client = greengrasssdk.client('secretsmanager')
   
  # This handler is called when the function is invoked
  # It uses the secretsmanager client to get the value of a specific secret version
  def function_handler(event, context):
      response = client.get_secret_value(SecretId='greengrass-MySecret-abc', VersionStage='MyTargetLabel')
      raw_secret = response.get('SecretString')
  ```

  For another example function that calls `get_secret_value`, see [Create a Lambda function deployment package](secrets-console.md#secrets-console-create-deployment-package)\.