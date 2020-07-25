# Access machine learning resources from Lambda functions<a name="access-ml-resources"></a>

User\-defined Lambda functions can access machine learning resources to run local inference on the AWS IoT Greengrass core\. A machine learning resource consists of the trained model and other artifacts that are downloaded to the core device\.

To allow a Lambda function to access a machine learning resource on the core, you must attach the resource to the Lambda function and define access permissions\. The [containerization mode](lambda-group-config.md#lambda-function-containerization) of the affiliated \(or *attached*\) Lambda function determines how you do this\.

## Access permissions for machine learning resources<a name="ml-resource-permissions"></a>

Starting in AWS IoT Greengrass Core v1\.10\.0, you can define a resource owner for a machine learning resource\. The resource owner represents the OS group and permissions that AWS IoT Greengrass uses to download the resource artifacts\. If a resource owner is not defined, the downloaded resource artifacts are accessible only to root\.
+ If non\-containerized Lambda functions access a machine learning resource, you must define a resource owner because there's no permission control from the container\. Non\-containerized Lambda functions can inherit resource owner permissions and use them to access the resource\.

   
+ If only containerized Lambda functions access the resource, we recommend that you use function\-level permissions instead of defining a resource owner\.

   

### Resource owner properties<a name="ml-resource-owner"></a>

A resource owner specifies a group owner and group owner permissions\.

  
**Group owner**\. The ID of the group \(GID\) of an existing Linux OS group on the core device\. The group's permissions are added to the Lambda process\. Specifically, the GID is added to the supplemental group IDs of the Lambda function\.  
If a Lambda function in the Greengrass group is configured to [run as](lambda-group-config.md#lambda-access-identity) the same OS group as the resource owner for a machine learning resource, the resource must be attached to the Lambda function\. Otherwise, deployment fails because this configuration gives implicit permissions the Lambda function can use to access the resource without AWS IoT Greengrass authorization\. The deployment validation check is skipped if the Lambda function runs as root \(UID=0\)\.  
We recommend that you use an OS group that's not used by other resources, Lambda functions, or files on the Greengrass core\. Using a shared OS group gives attached Lambda functions more access permissions than they need\. If you use a shared OS group, an attached Lambda function must also be attached to all machine learning resources that use the shared OS group\. Otherwise, deployment fails\.

  
**Group owner permissions**\. The read\-only or read and write permission to add to the Lambda process\.  
Non\-containerized Lambda functions must inherit these access permissions to the resource\. Containerized Lambda functions can inherit these resource\-level permissions or define function\-level permissions\. If they define function\-level permissions, the permissions must be the same or more restrictive than the resource\-level permissions\.

The following table shows supported access permission configurations\.

------
#### [ GGC v1\.10 or later ]


| Property | If only containerized Lambda functions access the resource | If any non\-containerized Lambda functions access the resource | 
| --- |--- |--- |
| **Function\-level properties** | 
| --- |
| Permissions \(read/write\) |  Required unless the resource defines a resource owner\. If a resource owner is defined, function\-level permissions must be the same or more restrictive than the resource owner permissions\. If only containerized Lambda functions access the resource, we recommend that you don't define a resource owner\.  |  **Non\-containerized Lambda functions:** Not supported\. Non\-containerized Lambda functions must inherit resource\-level permissions\. **Containerized Lambda functions:** Optional, but must be the same or more restrictive than resource\-level permissions\. | 
| **Resource\-level properties** | 
| --- |
| Resource owner | Optional \(not recommended\)\. | Required\. | 
| Permissions \(read/write\) | Optional \(not recommended\)\. | Required\. | 

------
#### [ GGC v1\.9 or earlier ]


| Property | If only containerized Lambda functions access the resource | If any non\-containerized Lambda functions access the resource | 
| --- |--- |--- |
| **Function\-level properties** | 
| --- |
| Permissions \(read/write\) |  Required\.  | Not supported\. | 
| **Resource\-level properties** | 
| --- |
| Resource owner | Not supported\. | Not supported\. | 
| Permissions \(read/write\) | Not supported\. | Not supported\. | 

------

**Note**  
When you use the AWS IoT Greengrass API to configure Lambda functions and resources, the function\-level `ResourceId` property is also required\. The `ResourceId` property attaches the machine learning resource to the Lambda function\.

## Defining access permissions for Lambda functions \(console\)<a name="ml-resource-permissions-console"></a>

In the AWS IoT console, you define access permissions when you configure a machine learning resource or attach one to a Lambda function\.

**Containerized Lambda functions**  
If only containerized Lambda functions are attached to the machine learning resource:  
+ Choose **No OS group** as the resource owner for the machine learning resource\. This is the recommended setting when only containerized Lambda functions access the machine learning resource\. Otherwise, you might give attached Lambda functions more access permissions than they need\.

   
+ Choose **Read\-only access** or **Read and write access** for the Lambda function access permissions\. You can do this when you attach the Lambda function to the machine learning resource:  
![\[Resource owner set to No OS group and Lambda function permissions set to Read-only access in the resource configuration.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/ml-resource-access-perms.png)

  Or, when you attach the machine learning resource to the Lambda function:  
![\[Resource owner set to No OS group and Lambda function permissions set to Read-only access in the function configuration.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/lambda-ml-resource-access.png)
 

**Non\-containerized Lambda functions** \(requires GGC v1\.10 or later\)  
If any non\-containerized Lambda functions are attached to the machine learning resource:  
+ Specify the ID of the OS group \(GID\) to use as the resource owner for the machine learning resource\. Choose **Specify OS group and permission** and enter the GID\. You can use the `getent group` command on your core device to look up the ID of an OS group\.

   
+ Choose **Read\-only access** or **Read and write access** for the OS group permissions\.  
![\[Specifying an OS group and access permissions for a machine learning resource.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/ml-resource-resource-owner.png)
+ Choose **Inherit resource owner permissions** for non\-containerized Lambda function access permissions\. You can do this when you affiliate the Lambda function and the resource:  
![\[Inheriting the access permissions of a machine learning resource.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/ml-inference/inherit-ml-resource-access-perms.png)

  For containerized Lambda functions that also access the machine learning resource, choose to inherit the OS group permissions or choose function\-level permissions\. If you choose function\-level permissions, they must be the same or more restrictive than the OS group permissions\.

## Defining access permissions for Lambda functions \(API\)<a name="ml-resource-permissions-api"></a>

In the AWS IoT Greengrass API, you define permissions to machine learning resources in the `ResourceAccessPolicy` property for the Lambda function or the `OwnerSetting` property for the resource\.

**Containerized Lambda functions**  
If only containerized Lambda functions are attached to the machine learning resource:  
+ For containerized Lambda functions, define access permissions in the `Permission` property of the `ResourceAccessPolicies` property\. For example:

  ```
  "Functions": [
      {
          "Id": "my-containerized-function",
          "FunctionArn": "arn:aws:lambda:us-west-2:123456789012:function:function-name:alias-or-version", 
          "FunctionConfiguration": {
              "Environment": {
                  "ResourceAccessPolicies": [
                      { 
                          "ResourceId": "my-resource-id",
                          "Permission": "ro-or-rw"
                      }
                  ]
              }, 
              "MemorySize": 512, 
              "Pinned": true, 
              "Timeout": 5
          }
      }
  ]
  ```
+ For machine learning resources, omit the `OwnerSetting` property\. For example:

  ```
  "Resources": [
      {
          "Id": "my-resource-id",
          "Name": "my-resource-name",
          "ResourceDataContainer": {
              "S3MachineLearningModelResourceData": {
                  "DestinationPath": "/local-destination-path",
                  "S3Uri": "s3://uri-to-resource-package"
              }
          }
      }
  ]
  ```

  This is the recommended configuration when only containerized Lambda functions access the machine learning resource\. Otherwise, you might give attached Lambda functions more access permissions than they need\.
 

**Non\-containerized Lambda functions** \(requires GGC v1\.10 or later\)  
If any non\-containerized Lambda functions are attached to the machine learning resource:  
+ For non\-containerized Lambda functions, omit the `Permission` property in `ResourceAccessPolicies`\. This configuration is required and allows the function to inherit the resource\-level permission\. For example:

  ```
  "Functions": [
      {
          "Id": "my-non-containerized-function",
          "FunctionArn": "arn:aws:lambda:us-west-2:123456789012:function:function-name:alias-or-version", 
          "FunctionConfiguration": {
              "Environment": {
                  "Execution": {
                      "IsolationMode": "NoContainer",
                  },            
                  "ResourceAccessPolicies": [
                      { 
                          "ResourceId": "my-resource-id"
                      }
                  ]
              }, 
              "Pinned": true, 
              "Timeout": 5
          }
      }
  ]
  ```
+ For containerized Lambda functions that also access the machine learning resource, omit the `Permission` property in `ResourceAccessPolicies` or define a permission that is the same or more restrictive as the resource\-level permission\. For example:

  ```
  "Functions": [
      {
          "Id": "my-containerized-function",
          "FunctionArn": "arn:aws:lambda:us-west-2:123456789012:function:function-name:alias-or-version", 
          "FunctionConfiguration": {
              "Environment": {
                  "ResourceAccessPolicies": [
                      { 
                          "ResourceId": "my-resource-id",
                          "Permission": "ro-or-rw" // Optional, but cannot exceed the GroupPermission defined for the resource.
                      }
                  ]
              }, 
              "MemorySize": 512, 
              "Pinned": true, 
              "Timeout": 5
          }
      }
  ]
  ```
+ For machine learning resources, define the `OwnerSetting` property, including the child `GroupOwner` and `GroupPermission` properties\. For example:

  ```
  "Resources": [
      {
          "Id": "my-resource-id",
          "Name": "my-resource-name",
          "ResourceDataContainer": {
              "S3MachineLearningModelResourceData": {
                  "DestinationPath": "/local-destination-path",
                  "S3Uri": "s3://uri-to-resource-package",
                  "OwnerSetting": { 
                      "GroupOwner": "os-group-id",
                      "GroupPermission": "ro-or-rw"
                  }
              }
          }
      }
  ]
  ```

## Accessing machine learning resources from Lambda function code<a name="access-resource-function-code"></a>

User\-defined Lambda functions use platform\-specific OS interfaces to access machine learning resources on a core device\.

------
#### [ GGC v1\.10 or later ]

For containerized Lambda functions, the resource is mounted inside the Greengrass container and available at the local destination path defined for the resource\. For non\-containerized Lambda functions, the resource is symlinked to a Lambda\-specific working directory and passed to the `AWS_GG_RESOURCE_PREFIX` environment variable in the Lambda process\.

To get the path to the downloaded artifacts of a machine learning resource, Lambda functions append the `AWS_GG_RESOURCE_PREFIX` environment variable to the local destination path defined for the resource\. For containerized Lambda functions, the returned value is a single forward slash \(`/`\)\.

```
resourcePath = os.getenv("AWS_GG_RESOURCE_PREFIX") + "/destination-path"
with open(resourcePath, 'r') as f:
    # load_model(f)
```

------
#### [ GGC v1\.9 or earlier ]

The downloaded artifacts of a machine learning resource are located in the local destination path defined for the resource\. Only containerized Lambda functions can access machine learning resources in AWS IoT Greengrass Core v1\.9 and earlier\.

```
resourcePath = "/local-destination-path"
with open(resourcePath, 'r') as f:
    # load_model(f)
```

------

Your model loading implementation depends on your ML library\.

## Troubleshooting<a name="access-ml-resources-troubleshooting"></a>

Use the following information to help troubleshoot issues with accessing machine learning resources\.

**Topics**
+ [InvalidMLModelOwner \- GroupOwnerSetting is provided in ML model resource, but GroupOwner or GroupPermission is not present](#nocontainer-lambda-invalid-ml-model-owner)
+ [NoContainer function cannot configure permission when attaching Machine Learning resources\. <function\-arn> refers to Machine Learnin resource <resource\-id> with permission <ro/rw> in resource access policy\.](#nocontainer-lambda-invalid-resource-access-policy)
+ [Function <function\-arn> refers to Machine Learning resource <resource\-id> with missing permission in both ResourceAccessPolicy and resource OwnerSetting\.](#nocontainer-lambda-missing-access-permission)
+ [Function <function\-arn> refers to Machine Learning resource <resource\-id> with permission \\"rw\\", while resource owner setting GroupPermission only allows \\"ro\\"\.](#container-lambda-invalid-rw-permissions)
+ [NoContainer Function <function\-arn> refers to resources of nested destination path\.](#nocontainer-lambda-nested-destination-path)
+ [Lambda <function\-arn> gains access to resource <resource\-id> by sharing the same group owner id](#lambda-runas-and-resource-owner)

### InvalidMLModelOwner \- GroupOwnerSetting is provided in ML model resource, but GroupOwner or GroupPermission is not present<a name="nocontainer-lambda-invalid-ml-model-owner"></a>

**Solution:** You receive this error if a machine learning resource contains the [ResourceDownloadOwnerSetting](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-resourcedownloadownersetting.html) object but the required `GroupOwner` or `GroupPermission` property isn't defined\. To resolve this issue, define the missing property\.

 

### NoContainer function cannot configure permission when attaching Machine Learning resources\. <function\-arn> refers to Machine Learnin resource <resource\-id> with permission <ro/rw> in resource access policy\.<a name="nocontainer-lambda-invalid-resource-access-policy"></a>

**Solution:** You receive this error if a non\-containerized Lambda function specifies function\-level permissions to a machine learning resource\. Non\-containerized functions must inherit permissions from the resource owner permissions defined on the machine learning resource\. To resolve this issue, choose to [inherit resource owner permissions](#non-container-config-console) \(console\) or [remove the permissions from the Lambda function's resource access policy](#non-container-config-api) \(API\)\.

 

### Function <function\-arn> refers to Machine Learning resource <resource\-id> with missing permission in both ResourceAccessPolicy and resource OwnerSetting\.<a name="nocontainer-lambda-missing-access-permission"></a>

**Solution:** You receive this error if permissions to the machine learning resource aren't configured for the attached Lambda function or the resource\. To resolve this issue, configure permissions in the [ResourceAccessPolicy](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-resourceaccesspolicy.html) property for the Lambda function or the [OwnerSetting](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-ownersetting.html) property for the resource\.

 

### Function <function\-arn> refers to Machine Learning resource <resource\-id> with permission \\"rw\\", while resource owner setting GroupPermission only allows \\"ro\\"\.<a name="container-lambda-invalid-rw-permissions"></a>

**Solution:** You receive this error if the access permissions defined for the attached Lambda function exceed the resource owner permissions defined for the machine learning resource\. To resolve this issue, set more restrictive permissions for the Lambda function or less restrictive permissions for the resource owner\.

 

### NoContainer Function <function\-arn> refers to resources of nested destination path\.<a name="nocontainer-lambda-nested-destination-path"></a>

**Solution:** You receive this error if multiple machine learning resources attached to a non\-containerized Lambda function use the same destination path or a nested destination path\. To resolve this issue, specify separate destination paths for the resources\.

 

### Lambda <function\-arn> gains access to resource <resource\-id> by sharing the same group owner id<a name="lambda-runas-and-resource-owner"></a>

**Solution:** You receive this error in `runtime.log` if the same OS group is specified as the Lambda function's [Run as](lambda-group-config.md#lambda-access-identity) identity and the [resource owner](#ml-resource-owner) for a machine learning resource, but the resource is not attached to the Lambda function\. This configuration gives the Lambda function implicit permissions that it can use to access the resource without AWS IoT Greengrass authorization\.

To resolve this issue, use a different OS group for one of the properties or attach the machine learning resource to the Lambda function\.

## See also<a name="access-ml-resources-see-also"></a>
+ [Perform machine learning inference](ml-inference.md)
+ [How to configure machine learning inference using the AWS Management Console](ml-console.md)
+ [How to configure optimized machine learning inference using the AWS Management Console](ml-dlc-console.md)
+ [AWS IoT Greengrass API Reference](https://docs.aws.amazon.com/greengrass/latest/apireference/api-doc.html)