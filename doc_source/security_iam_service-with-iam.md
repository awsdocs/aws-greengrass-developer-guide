# How AWS IoT Greengrass works with IAM<a name="security_iam_service-with-iam"></a>

Before you use IAM to manage access to AWS IoT Greengrass, you should understand the IAM features that you can use with AWS IoT Greengrass\.


| IAM feature | Supported by Greengrass? | 
| --- | --- | 
| [Identity\-based policies with resource\-level permissions](#security_iam_service-with-iam-id-based-policies) | Yes | 
| [Resource\-based policies](#security_iam_service-with-iam-resource-based-policies) | No | 
| [Access control lists \(ACLs\)](#security_iam_service-with-iam-acls) | No | 
| [Tags\-based authorization](#security_iam_service-with-iam-tags) | Yes | 
| [Temporary credentials](#security_iam_service-with-iam-roles-tempcreds) | Yes | 
| [Service\-linked roles](#security_iam_service-with-iam-roles-service-linked) | No | 
| [Service roles](#security_iam_service-with-iam-roles-service-linked) | Yes | 

For a high\-level view of how other AWS services work with IAM, see [AWS services that work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html) in the *IAM User Guide*\.

## Identity\-based policies for AWS IoT Greengrass<a name="security_iam_service-with-iam-id-based-policies"></a>

With IAM identity\-based policies, you can specify allowed or denied actions and resources and the conditions under which actions are allowed or denied\. AWS IoT Greengrass supports specific actions, resources, and condition keys\. To learn about all of the elements that you use in a policy, see [IAM JSON policy elements reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) in the *IAM User Guide*\.

### Actions<a name="security_iam_service-with-iam-id-based-policies-actions"></a>

The `Action` element of an IAM identity\-based policy describes the specific action or actions that will be allowed or denied by the policy\. Policy actions usually have the same name as the associated AWS API operation\. The action is used in a policy to grant permissions to perform the associated operation\. 

Policy actions for AWS IoT Greengrass use the `greengrass:` prefix before the action\. For example, to allow someone to use the `ListGroups` API operation to list the groups in their AWS account, you include the `greengrass:ListGroups` action in their policy\. Policy statements must include either an `Action` or `NotAction` element\. AWS IoT Greengrass defines its own set of actions that describe tasks that you can perform with this service\.

To specify multiple actions in a single statement, list them between brackets \(`[` `]`\) and separate them with commas, as follows:

```
"Action": [
    "greengrass:action1",
    "greengrass:action2",
    "greengrass:action3"
]
```

You can use wildcards \(`*`\) to specify multiple actions\. For example, to specify all actions that begin with the word `List`, include the following action:

```
"Action": "greengrass:List*"
```

**Note**  
We recommend that you avoid the use of wildcards to specify all available actions for a service\. As a best practice, you should grant least privilege and narrowly scope permissions in a policy\. For more information, see [Grant minimum possible permissions](security-best-practices.md#least-privilege)\.

For the complete list of AWS IoT Greengrass actions, see [Actions Defined by AWS IoT Greengrass](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsiotgreengrass.html#awsiotgreengrass-actions-as-permissions) in the *IAM User Guide*\.

### Resources<a name="security_iam_service-with-iam-id-based-policies-resources"></a>

The `Resource` element specifies the object or objects to which the action applies\. Statements must include either a `Resource` or a `NotResource` element\. You specify a resource using an ARN or using the wildcard \(\*\) to indicate that the statement applies to all resources\.

The following table contains the AWS IoT Greengrass resource ARNs that can be used in the `Resource` element of a policy statement\. For a mapping of supported resource\-level permissions for AWS IoT Greengrass actions, see [Actions Defined by AWS IoT Greengrass](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_awsiotgreengrass.html#awsiotgreengrass-actions-as-permissions) in the *IAM User Guide*\.


| Resource | ARN | 
| --- | --- | 
|   [ Group ](https://docs.aws.amazon.com/greengrass/latest/apireference/listgroups-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/groups/$\{GroupId\}  | 
|   [ GroupVersion ](https://docs.aws.amazon.com/greengrass/latest/apireference/listgroupversions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/groups/$\{GroupId\}/versions/$\{VersionId\}  | 
|   [ CertificateAuthority ](https://docs.aws.amazon.com/greengrass/latest/apireference/listgroupcertificateauthorities-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/groups/$\{GroupId\}/certificateauthorities/$\{CertificateAuthorityId\}  | 
|   [ Deployment ](https://docs.aws.amazon.com/greengrass/latest/apireference/listdeployments-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/groups/$\{GroupId\}/deployments/$\{DeploymentId\}  | 
|   [ BulkDeployment ](https://docs.aws.amazon.com/greengrass/latest/apireference/listbulkdeployments-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/bulk/deployments/$\{BulkDeploymentId\}  | 
|   [ ConnectorDefinition ](https://docs.aws.amazon.com/greengrass/latest/apireference/listconnectordefinitions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/connectors/$\{ConnectorDefinitionId\}  | 
|   [ ConnectorDefinitionVersion ](https://docs.aws.amazon.com/greengrass/latest/apireference/listconnectordefinitionversions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/connectors/$\{ConnectorDefinitionId\}/versions/$\{VersionId\}  | 
|   [ CoreDefinition ](https://docs.aws.amazon.com/greengrass/latest/apireference/listcoredefinitions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/cores/$\{CoreDefinitionId\}  | 
|   [ CoreDefinitionVersion ](https://docs.aws.amazon.com/greengrass/latest/apireference/listcoredefinitionversions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/cores/$\{CoreDefinitionId\}/versions/$\{VersionId\}  | 
|   [ DeviceDefinition ](https://docs.aws.amazon.com/greengrass/latest/apireference/listdevicedefinitions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/devices/$\{DeviceDefinitionId\}  | 
|   [ DeviceDefinitionVersion ](https://docs.aws.amazon.com/greengrass/latest/apireference/listdevicedefinitionversions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/devices/$\{DeviceDefinitionId\}/versions/$\{VersionId\}  | 
|   [ FunctionDefinition ](https://docs.aws.amazon.com/greengrass/latest/apireference/listfunctiondefinitions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/functions/$\{FunctionDefinitionId\}  | 
|   [ FunctionDefinitionVersion ](https://docs.aws.amazon.com/greengrass/latest/apireference/listfunctiondefinitionversions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/functions/$\{FunctionDefinitionId\}/versions/$\{VersionId\}  | 
|   [ LoggerDefinition ](https://docs.aws.amazon.com/greengrass/latest/apireference/listloggerdefinitions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/loggers/$\{LoggerDefinitionId\}  | 
|   [ LoggerDefinitionVersion ](https://docs.aws.amazon.com/greengrass/latest/apireference/listloggerdefinitionversions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/loggers/$\{LoggerDefinitionId\}/versions/$\{VersionId\}  | 
|   [ ResourceDefinition ](https://docs.aws.amazon.com/greengrass/latest/apireference/listresourcedefinitions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/resources/$\{ResourceDefinitionId\}  | 
|   [ ResourceDefinitionVersion ](https://docs.aws.amazon.com/greengrass/latest/apireference/listresourcedefinitionversions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/resources/$\{ResourceDefinitionId\}/versions/$\{VersionId\}  | 
|   [ SubscriptionDefinition ](https://docs.aws.amazon.com/greengrass/latest/apireference/listsubscriptiondefinitions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/subscriptions/$\{SubscriptionDefinitionId\}  | 
|   [ SubscriptionDefinitionVersion ](https://docs.aws.amazon.com/greengrass/latest/apireference/listsubscriptiondefinitionversions-get.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/definition/subscriptions/$\{SubscriptionDefinitionId\}/versions/$\{VersionId\}  | 
|   [ ConnectivityInfo ](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-connectivityinfo.html)  |  arn:$\{Partition\}:greengrass:$\{Region\}:$\{Account\}:/greengrass/things/$\{ThingName\}/connectivityInfo  | 

The following example `Resource` element specifies the ARN of a group in the US West \(Oregon\) Region in the AWS account `123456789012`:

```
"Resource": "arn:aws:greengrass:us-west-2:123456789012:/greengrass/groups/a1b2c3d4-5678-90ab-cdef-EXAMPLE11111
```

Or, to specify all groups that belong to an AWS account in a specific AWS Region, use the wildcard in place of the group ID:

```
"Resource": "arn:aws:greengrass:us-west-2:123456789012:/greengrass/groups/*"
```

Some AWS IoT Greengrass actions \(for example, some list operations\), cannot be performed on a specific resource\. In those cases, you must use the wildcard alone\.

```
"Resource": "*"
```

To specify multiple resource ARNs in a statement, list them between brackets \(`[` `]`\) and separate them with commas, as follows:

```
"Resource": [
    "resource-arn1",
    "resource-arn2",
    "resource-arn3"
]
```

For more information about ARN formats, see [Amazon Resource Names \(ARNs\) and AWS service namespaces](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) in the *Amazon Web Services General Reference*\.

### Condition keys<a name="security_iam_service-with-iam-id-based-policies-conditionkeys"></a>

The `Condition` element \(or `Condition` *block*\) lets you specify conditions in which a statement is in effect\. The `Condition` element is optional\. You can create conditional expressions that use [condition operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html), such as equals or less than, to match the condition in the policy with values in the request\. 

If you specify multiple `Condition` elements in a statement, or multiple keys in a single `Condition` element, AWS evaluates them using a logical `AND` operation\. If you specify multiple values for a single condition key, AWS evaluates the condition using a logical `OR` operation\. All of the conditions must be met before the statement's permissions are granted\.

 You can also use placeholder variables when you specify conditions\. For example, you can grant an IAM user permission to access a resource only if it is tagged with their IAM user name\. For more information, see [IAM Policy Elements: Variables and Tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html) in the *IAM User Guide*\. 

AWS IoT Greengrass supports the following global condition keys\.


| Key | Description | 
| --- | --- | 
| aws:CurrentTime  | Filters access by checking date/time conditions for the current date and time\. | 
| aws:EpochTime | Filters access by checking date/time conditions for the current date and time in epoch or Unix time\. | 
| aws:MultiFactorAuthAge | Filters access by checking how long ago \(in seconds\) the security credentials validated by multi\-factor authentication \(MFA\) in the request were issued using MFA\. | 
| aws:MultiFactorAuthPresent |  Filters access by checking whether multi\-factor authentication \(MFA\) was used to validate the temporary security credentials that made the current request\. | 
| aws:RequestTag/$\{TagKey\} | Filters create requests based on the allowed set of values for each of the mandatory tags\. | 
| aws:ResourceTag/$\{TagKey\} | Filters actions based on the tag value associated with the resource\. | 
| aws:SecureTransport | Filters access by checking whether the request was sent using SSL\. | 
| aws:TagKeys | Filters create requests based on the presence of mandatory tags in the request\. | 
| aws:UserAgent | Filters access by the requester's client application\. | 

For more information, see [AWS global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\.

### Examples<a name="security_iam_service-with-iam-id-based-policies-examples"></a>



To view examples of AWS IoT Greengrass identity\-based policies, see [Identity\-based policy examples for AWS IoT Greengrass](security_iam_id-based-policy-examples.md)\.

## Resource\-based policies for AWS IoT Greengrass<a name="security_iam_service-with-iam-resource-based-policies"></a>

AWS IoT Greengrass does not support [resource\-based policies](security-iam.md#security_iam_access-manage-resource-based-policies)\.

## Access control lists \(ACLs\)<a name="security_iam_service-with-iam-acls"></a>

AWS IoT Greengrass does not support [ACLs](security-iam.md#security_iam_access-manage-acl)\.

## Authorization based on AWS IoT Greengrass tags<a name="security_iam_service-with-iam-tags"></a>

You can attach tags to supported AWS IoT Greengrass resources or pass tags in a request to AWS IoT Greengrass\. To control access based on tags, you provide tag information in the [Condition element](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) of a policy using the `aws:ResourceTag/${TagKey}`, `aws:RequestTag/${TagKey}`, or `aws:TagKeys` condition keys\. For more information, see [Tagging your AWS IoT Greengrass resources](tagging.md)\.

## IAM roles for AWS IoT Greengrass<a name="security_iam_service-with-iam-roles"></a>

An [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) is an entity within your AWS account that has specific permissions\.

### Using temporary credentials with AWS IoT Greengrass<a name="security_iam_service-with-iam-roles-tempcreds"></a>

Temporary credentials are used to sign in with federation, assume an IAM role, or to assume a cross\-account role\. You obtain temporary security credentials by calling AWS STS API operations such as [AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) or [GetFederationToken](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetFederationToken.html)\.

On the Greengrass core, temporary credentials for the [group role](group-role.md) are made available to user\-defined Lambda functions and connectors\. If your Lambda functions use the AWS SDK, you don't need to add logic to obtain the credentials because the AWS SDK does this for you\.

### Service\-linked roles<a name="security_iam_service-with-iam-roles-service-linked"></a>

AWS IoT Greengrass does not support [service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-linked-role)\.

### Service roles<a name="security_iam_service-with-iam-roles-service"></a>

This feature allows a service to assume a [service role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-role) on your behalf\. This role allows the service to access resources in other services to complete an action on your behalf\. Service roles appear in your IAM account and are owned by the account\. This means that an IAM administrator can change the permissions for this role\. However, doing so might break the functionality of the service\.

AWS IoT Greengrass uses a service role to access some of your AWS resources on your behalf\. For more information, see [Greengrass service role](service-role.md)\.

### Choosing an IAM role in the AWS IoT Greengrass console<a name="security_iam_service-with-iam-roles-choose"></a>

In the AWS IoT Greengrass console, you might need to choose a Greengrass service role or a Greengrass group role from a list of IAM roles in your account\.
+ The Greengrass service role allows AWS IoT Greengrass to access your AWS resources in other services on your behalf\. Typically, you don't need to choose the service role because the console can create and configure it for you\. For more information, see [Greengrass service role](service-role.md)\.
+ The Greengrass group role is used to allow Greengrass Lambda functions and connectors in the group to access your AWS resources\. It can also give AWS IoT Greengrass permissions to export streams to AWS services and write CloudWatch logs\. For more information, see [Greengrass group role](group-role.md)\.