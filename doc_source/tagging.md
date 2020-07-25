# Tagging your AWS IoT Greengrass resources<a name="tagging"></a>

Tags can help you organize and manage your AWS IoT Greengrass groups\. You can use tags to assign metadata to groups, bulk deployments, and the cores, devices, and other resources that are added to groups\. Tags can also be used in IAM policies to define conditional access to your Greengrass resources\.

**Note**  
Currently, Greengrass resource tags are not supported for AWS IoT billing groups or cost allocation reports\.

## Tag basics<a name="tagging-basics"></a>

Tags allow you to categorize your AWS IoT Greengrass resources, for example, by purpose, owner, and environment\. When you have many resources of the same type, you can quickly identify a resource based on the tags that are attached to it\. A tag consists of a key and optional value, both of which you define\. We recommend that you design a set of tag keys for each resource type\. Using a consistent set of tag keys makes it easier for you to manage your resources\. For example, you can define a set of tags for your groups that helps you track the factory location of your core devices\. For more information, see [AWS Tagging Strategies](https://aws.amazon.com/answers/account-management/aws-tagging-strategies)\.

### Tagging support in the AWS IoT console<a name="tagging-support-console"></a>

You can create, view, and manage tags for your Greengrass `Group` resources in the AWS IoT console\. Before you create tags, be aware of tagging restrictions\. For more information, see [Tag naming and usage conventions](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html) in the *Amazon Web Services General Reference*\.

**To assign tags when you create a group**  
You can assign tags to a group when you create the group\. To show the tagging input fields, on the **Name your Group** dialog box, choose **Apply tags to the Group \(optional\)**\.  

![\[The Apply tags to the Group section of the Name your Group page with one tag key and value defined.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-create-group-tags.png)

**To view and manage tags from the group configuration page**  
You can view and manage tags from the group configuration page\. On the **Tags** page for the group, choose **Add tags** or **Manage tags** to add, edit, or remove group tags\.  

![\[The Tags page showing several tag key-value pairs.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-tags.png)

### Tagging support in the AWS IoT Greengrass API<a name="tagging-support-api"></a>

You can use the AWS IoT Greengrass API to create, list, and manage tags for AWS IoT Greengrass resources that support tagging\. Before you create tags, be aware of tagging restrictions\. For more information, see [Tag naming and usage conventions](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html) in the *Amazon Web Services General Reference*\.
+ To add tags during resource creation, define them in the `tags` property of the resource\.
+ To add tags after a resource is created, or to update tag values, use the `TagResource` action\.
+ To remove tags from a resource, use the `UntagResource` action\.
+ To retrieve the tags that are associated with a resource, use the `ListTagsForResource` action or get the resource and inspect its `tags` property\.

The following table lists resources you can tag in the AWS IoT Greengrass API and their corresponding `Create` and `Get` actions\.


| Resource | Create | Get | 
| --- | --- | --- | 
| Group | [https://docs.aws.amazon.com/greengrass/latest/apireference/creategroup-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/creategroup-post.html) | [https://docs.aws.amazon.com/greengrass/latest/apireference/getgroup-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getgroup-get.html) | 
| ConnectorDefinition | [https://docs.aws.amazon.com/greengrass/latest/apireference/createconnectordefinition-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/createconnectordefinition-post.html) | [https://docs.aws.amazon.com/greengrass/latest/apireference/getconnectordefinition-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getconnectordefinition-get.html) | 
| CoreDefinition | [https://docs.aws.amazon.com/greengrass/latest/apireference/createcoredefinition-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/createcoredefinition-post.html) | [https://docs.aws.amazon.com/greengrass/latest/apireference/getcoredefinition-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getcoredefinition-get.html) | 
| DeviceDefinition | [https://docs.aws.amazon.com/greengrass/latest/apireference/createdevicedefinition-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/createdevicedefinition-post.html) | [https://docs.aws.amazon.com/greengrass/latest/apireference/getdevicedefinition-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getdevicedefinition-get.html) | 
| FunctionDefinition | [https://docs.aws.amazon.com/greengrass/latest/apireference/createfunctiondefinition-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/createfunctiondefinition-post.html) | [https://docs.aws.amazon.com/greengrass/latest/apireference/getfunctiondefinition-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getfunctiondefinition-get.html) | 
| LoggerDefinition | [https://docs.aws.amazon.com/greengrass/latest/apireference/createloggerdefinition-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/createloggerdefinition-post.html) | [https://docs.aws.amazon.com/greengrass/latest/apireference/getloggerdefinition-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getloggerdefinition-get.html) | 
| ResourceDefinition | [https://docs.aws.amazon.com/greengrass/latest/apireference/createresourcedefinition-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/createresourcedefinition-post.html) | [https://docs.aws.amazon.com/greengrass/latest/apireference/getresourcedefinition-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getresourcedefinition-get.html) | 
| SubscriptionDefinition | [https://docs.aws.amazon.com/greengrass/latest/apireference/createsubscriptiondefinition-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/createsubscriptiondefinition-post.html) | [https://docs.aws.amazon.com/greengrass/latest/apireference/getsubscriptiondefinition-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getsubscriptiondefinition-get.html) | 
| BulkDeployment  | [https://docs.aws.amazon.com/greengrass/latest/apireference/startbulkdeployment-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/startbulkdeployment-post.html) | [https://docs.aws.amazon.com/greengrass/latest/apireference/getbulkdeploymentstatus-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/getbulkdeploymentstatus-get.html) | 

Use the following actions to list and manage tags for resources that support tagging:
+ [https://docs.aws.amazon.com/greengrass/latest/apireference/tagresource-post.html](https://docs.aws.amazon.com/greengrass/latest/apireference/tagresource-post.html)\. Adds tags to a resource\. Also used to change the value of the tag's key\-value pair\.
+ [https://docs.aws.amazon.com/greengrass/latest/apireference/listtagsforresource-get.html](https://docs.aws.amazon.com/greengrass/latest/apireference/listtagsforresource-get.html)\. Lists the tags for a resource\.
+ [https://docs.aws.amazon.com/greengrass/latest/apireference/untagresource-delete.html](https://docs.aws.amazon.com/greengrass/latest/apireference/untagresource-delete.html)\. Removes tags from a resource\.

You can add or remove tags on a resource at any time\. To change the value of a tag key, add a tag to the resource that defines the same key and the new value\. The new value overwrites the old value\. You can set a value to an empty string, but you can't set a value to null\.

When you delete a resource, tags that are associated with the resource are also deleted\.

**Note**  
Don't confuse resource tags with the attributes that you can assign to AWS IoT things\. Although Greengrass cores are AWS IoT things, the resource tags that are described in this topic are attached to a `CoreDefinition`, not the core thing\.

## Using tags with IAM policies<a name="tagging-iam-policies"></a>

In your IAM policies, you can use resource tags to control user access and permissions\. For example, policies can allow users to create only those resources that have a specific tag\. Policies can also restrict users from creating or modifying resources that have certain tags\. You can tag resources during creation \(called *tag on create*\) so you don't have to run custom tagging scripts later\. When new environments are launched with tags, the corresponding IAM permissions are applied automatically\.

The following condition context keys and values can be used in the `Condition` element \(also called the `Condition` block\) of the policy\.

`greengrass:ResourceTag/tag-key: tag-value`  
Allow or deny user actions on resources with specific tags\.

`aws:RequestTag/tag-key: tag-value`  
Require that a specific tag be used \(or not used\) when making API requests to create or modify tags on a taggable resource\.

`aws:TagKeys: [tag-key, ...]`  
Require that a specific set of tag keys be used \(or not used\) when making an API request to create or modify a taggable resource\.

Condition context keys and values can be used only on AWS IoT Greengrass actions that act on a taggable resource\. These actions take the resource as a required parameter\. For example, you can set conditional access on the `GetGroupVersion`\. You can't set conditional access on `AssociateServiceRoleToAccount` because no taggable resource \(for example, group, core definition, or device defintion\) is referenced in the request\.

For more information, see [Controlling access using tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html) and [IAM JSON policy reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\. The JSON policy reference includes detailed syntax, descriptions and examples of the elements, variables, and evaluation logic of JSON policies in IAM\.

### Example IAM policies<a name="tagging-iam-policies-examples"></a>

The following example policy applies tag\-based permissions that constrain a beta user to actions on beta resources only\.
+ The first statement allows an IAM user to act on resources that have the *env=beta* tag only\.
+ The second statement prevents an IAM user from removing the *env=beta* tag from resources\. This protects the user from removing their own access\.
**Note**  
If you use tags to control access to resources, you should also manage the permissions that allow users to add tags or remove tags from those same resources\. Otherwise, in some cases, it might be possible for users to circumvent your restrictions and gain access to a resource by modifying its tags\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "greengrass:*",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "greengrass:ResourceTag/env": "beta"
                }
            }
        },
        {
            "Effect": "Deny",
            "Action": "greengrass:UntagResource",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/env": "beta"
                }
            }
        }
    ]
}
```

To allow users to tag on create, you must give them appropriate permissions\. The following example policy includes the `"aws:RequestTag/env": "beta"` condition on the `greengrass:TagResource` and `greengrass:CreateGroup` actions, which allows users to create a group only if they tag the group with *env=beta*\. This effectively forces users to tag new groups\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "greengrass:TagResource",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/env": "beta"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": "greengrass:CreateGroup",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/env": "beta"
                }
            }
        }
    ]
}
```

The following snippet shows how you can specify multiple tag values for a tag key by enclosing them in a list:

```
"StringEquals" : {
    "greengrass:ResourceTag/env" : ["dev", "test"]
}
```

## See also<a name="tagging-see-also"></a>
+ [Tagging AWS resources](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html) in the *Amazon Web Services General Reference*