# How to configure local resource access using the AWS command line interface<a name="lra-cli"></a>

This feature is available for AWS IoT Greengrass Core v1\.3 and later\.

To use a local resource, you must add a resource definition to the group definition that is deployed to your Greengrass core device\. The group definition must also contain a Lambda function definition in which you grant access permissions for local resources to your Lambda functions\. For more information, including requirements and constraints, see [Access local resources with Lambda functions and connectors](access-local-resources.md)\.

This tutorial describes the process for creating a local resource and configuring access to it using the AWS Command Line Interface \(CLI\)\. To follow the steps in the tutorial, you must have already created a Greengrass group as described in [Getting started with AWS IoT Greengrass](gg-gs.md)\. 

For a tutorial that uses the AWS Management Console, see [How to configure local resource access using the AWS Management Console](lra-console.md)\.

## Create local resources<a name="lra-cli-create-resources"></a>

First, you use the `[CreateResourceDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/createresourcedefinition-post.html)` command to create a resource definition that specifies the resources to be accessed\. In this example, we create two resources, `TestDirectory` and `TestCamera`:

```
aws greengrass create-resource-definition  --cli-input-json '{
    "Name": "MyLocalVolumeResource",
    "InitialVersion": {
        "Resources": [
            {
                "Id": "data-volume",
                "Name": "TestDirectory",
                "ResourceDataContainer": {
                    "LocalVolumeResourceData": {
                        "SourcePath": "/src/LRAtest",
                        "DestinationPath": "/dest/LRAtest",
                        "GroupOwnerSetting": {
                            "AutoAddGroupOwner": true,
                            "GroupOwner": ""
                        }
                    }
                }
            },
            {
                "Id": "data-device",
                "Name": "TestCamera",
                "ResourceDataContainer": {
                    "LocalDeviceResourceData": {
                        "SourcePath": "/dev/video0",
                        "GroupOwnerSetting": {
                            "AutoAddGroupOwner": true,
                            "GroupOwner": ""
                        }
                    }
                }
            }
        ]
    }
}'
```

**Resources**: A list of `Resource` objects in the Greengrass group\. One Greengrass group can have up to 50 resources\.

**Resource\#Id**: The unique identifier of the resource\. The ID is used to refer to a resource in the Lambda function configuration\. Max length 128 characters\. Pattern: \[a\-zA\-Z0\-9:\_\-\]\+\. 

**Resource\#Name**: The name of the resource\. The resource name is displayed in the Greengrass console\. Max length 128 characters\. Pattern: \[a\-zA\-Z0\-9:\_\-\]\+\.

**LocalDeviceResourceData\#SourcePath**: The local absolute path of the device resource\. The source path for a device resource can refer only to a character device or block device under `/dev`\.

**LocalVolumeResourceData\#SourcePath**: The local absolute path of the volume resource on the Greengrass core device\. This location is outside of the [container](lambda-group-config.md#lambda-function-containerization) that the function runs in\. The source path for a volume resource type cannot start with `/sys`\.

**LocalVolumeResourceData\#DestinationPath**: The absolute path of the volume resource inside the Lambda environment\. This location is inside the container that the function runs in\.

**GroupOwnerSetting**: Allows you to configure additional group privileges for the Lambda process\. This field is optional\. For more information, see [Group owner file access permission](access-local-resources.md#lra-group-owner)\.

**GroupOwnerSetting\#AutoAddGroupOwner**: If true, Greengrass automatically adds the specified Linux OS group owner of the resource to the Lambda process privileges\. Thus the Lambda process has the file access permissions of the added Linux group\.

**GroupOwnerSetting\#GroupOwner**: Specifies the name of the Linux OS group whose privileges are added to the Lambda process\. This field is optional\. 

A resource definition version ARN is returned by `[CreateResourceDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/createresourcedefinition-post.html)`\. The ARN should be used when updating a group definition\.

```
{
    "LatestVersionArn": "arn:aws:greengrass:us-west-2:012345678901:/greengrass/definition/resources/ab14d0b5-116e-4951-a322-9cde24a30373/versions/a4d9b882-d025-4760-9cfe-9d4fada5390d",
    "Name": "MyLocalVolumeResource",
    "LastUpdatedTimestamp": "2017-11-15T01:18:42.153Z",
    "LatestVersion": "a4d9b882-d025-4760-9cfe-9d4fada5390d",
    "CreationTimestamp": "2017-11-15T01:18:42.153Z",
    "Id": "ab14d0b5-116e-4951-a322-9cde24a30373",
    "Arn": "arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/resources/ab14d0b5-116e-4951-a322-9cde24a30373"
}
```

## Create the Greengrass function<a name="lra-cli-create-function"></a>

After the resources are created, use the `[CreateFunctionDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/createfunctiondefinition-post.html)` command to create the Greengrass function and grant the function access to the resource: 

```
aws greengrass create-function-definition --cli-input-json '{
    "Name": "MyFunctionDefinition",
    "InitialVersion": {
        "Functions": [
            {
                "Id": "greengrassLraTest",
                "FunctionArn": "arn:aws:lambda:us-west-2:012345678901:function:lraTest:1",
                "FunctionConfiguration": {
                    "Pinned": false,
                    "MemorySize": 16384,
                    "Timeout": 30,
                    "Environment": {
                        "ResourceAccessPolicies": [
                            {
                                "ResourceId": "data-volume",
                                "Permission": "rw"
                            },
                            {
                                "ResourceId": "data-device",
                                "Permission": "ro"
                            }                            
                        ],
                        "AccessSysfs": true
                    }
                }
            }
        ]
    }
}'
```

**ResourceAccessPolicies**: Contains the `resourceId` and `permission` which grant the Lambda function access to the resource\. A Lambda function can access a maximum of 20 resources\.

**ResourceAccessPolicy\#Permission**: Specifies which permissions the Lambda function has on the resource\. The available options are `rw` \(read/write\) or `ro` \(read\-only\)\. 

**AccessSysfs**: If true, the Lambda process can have read access to the `/sys` folder on the Greengrass core device\. This is used in cases where the Greengrass Lambda function needs to read device information from `/sys`\.

Again, `[CreateFunctionDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/createfunctiondefinition-post.html)` returns a function definition version ARN\. The ARN should be used in your group definition version\. 

```
{
    "LatestVersionArn": "arn:aws:greengrass:us-west-2:012345678901:/greengrass/definition/functions/3c9b1685-634f-4592-8dfd-7ae1183c28ad/versions/37f0d50e-ef50-4faf-b125-ade8ed12336e", 
    "Name": "MyFunctionDefinition", 
    "LastUpdatedTimestamp": "2017-11-22T02:28:02.325Z", 
    "LatestVersion": "37f0d50e-ef50-4faf-b125-ade8ed12336e", 
    "CreationTimestamp": "2017-11-22T02:28:02.325Z", 
    "Id": "3c9b1685-634f-4592-8dfd-7ae1183c28ad", 
    "Arn": "arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/functions/3c9b1685-634f-4592-8dfd-7ae1183c28ad"
}
```

## Add the Lambda function to the group<a name="lra-cli-add-function"></a>

Finally, use `[CreateGroupVersion](https://docs.aws.amazon.com/greengrass/latest/apireference/creategroupversion-post.html)` to add the function to the group\. For example:

```
aws greengrass create-group-version --group-id "b36a3aeb-3243-47ff-9fa4-7e8d98cd3cf5" \
--resource-definition-version-arn "arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/resources/db6bf40b-29d3-4c4e-9574-21ab7d74316c/versions/31d0010f-e19a-4c4c-8098-68b79906fb87" \
--core-definition-version-arn "arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/cores/adbf3475-f6f3-48e1-84d6-502f02729067/versions/297c419a-9deb-46dd-8ccc-341fc670138b" \
--function-definition-version-arn "arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/functions/d1123830-da38-4c4c-a4b7-e92eec7b6d3e/versions/a2e90400-caae-4ffd-b23a-db1892a33c78" \
--subscription-definition-version-arn "arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/subscriptions/7a8ef3d8-1de3-426c-9554-5b55a32fbcb6/versions/470c858c-7eb3-4abd-9d48-230236bfbf6a"
```

**Note**  
To learn how to get the group ID to use with this command, see [Getting the group ID](deployments.md#api-get-group-id)\.

A new group version is returned:

```
{
    "Arn": "arn:aws:greengrass:us-west-2:012345678901:/greengrass/groups/b36a3aeb-3243-47ff-9fa4-7e8d98cd3cf5/versions/291917fb-ec54-4895-823e-27b52da25481",
    "Version": "291917fb-ec54-4895-823e-27b52da25481",
    "CreationTimestamp": "2017-11-22T01:47:22.487Z",
    "Id": "b36a3aeb-3243-47ff-9fa4-7e8d98cd3cf5"
}
```

Your Greengrass group now contains the *lraTest* Lambda function that has access to two resources: TestDirectory and TestCamera\.

This example Lambda function, `lraTest.py`, written in Python, writes to the local volume resource: 

```
# Demonstrates a simple use case of local resource access.
# This Lambda function writes a file test to a volume mounted inside
# the Lambda environment under destLRAtest. Then it reads the file and 
# publishes the content to the AWS IoT LRAtest topic. 

import sys
import greengrasssdk
import platform
import os
import logging

# Setup logging to stdout
logger = logging.getLogger(__name__)
logging.basicConfig(stream=sys.stdout, level=logging.DEBUG)

# Create a Greengrass Core SDK client.
client = greengrasssdk.client('iot-data')
volumePath = '/dest/LRAtest'

def function_handler(event, context):
    try:
        client.publish(topic='LRA/test', payload='Sent from AWS IoT Greengrass Core.')
        volumeInfo = os.stat(volumePath)
        client.publish(topic='LRA/test', payload=str(volumeInfo))
        with open(volumePath + '/test', 'a') as output:
            output.write('Successfully write to a file.')
        with open(volumePath + '/test', 'r') as myfile:
            data = myfile.read()
        client.publish(topic='LRA/test', payload=data)
    except Exception as e:
        logger.error('Failed to publish message: ' + repr(e))
    return
```

These commands are provided by the Greengrass API to create and manage resource definitions and resource definition versions:
+ [CreateResourceDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/createresourcedefinition-post.html)
+ [CreateResourceDefinitionVersion](https://docs.aws.amazon.com/greengrass/latest/apireference/createresourcedefinitionversion-post.html)
+  [DeleteResourceDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/deleteresourcedefinition-delete.html)
+  [GetResourceDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/getresourcedefinition-get.html)
+  [GetResourceDefinitionVersion](https://docs.aws.amazon.com/greengrass/latest/apireference/getresourcedefinitionversion-get.html)
+  [ListResourceDefinitions](https://docs.aws.amazon.com/greengrass/latest/apireference/listresourcedefinitions-get.html)
+  [ListResourceDefinitionVersions](https://docs.aws.amazon.com/greengrass/latest/apireference/listresourcedefinitionversions-get.html)
+  [UpdateResourceDefinition](https://docs.aws.amazon.com/greengrass/latest/apireference/updateresourcedefinition-put.html)

## Troubleshooting<a name="lra-faqs"></a>
+ **Q:** Why does my Greengrass group deployment fail with an error similar to:

  ```
  group config is invalid: 
      ggc_user or [ggc_group root tty] don't have ro permission on the file: /dev/tty0
  ```

  **A:** This error indicates that the Lambda process doesn't have permission to access the specified resource\. The solution is to change the file permission of the resource so that Lambda can access it\. \(See [Group owner file access permission](access-local-resources.md#lra-group-owner) for details\)\.
+ **Q:** When I configure `/var/run` as a volume resource, why does the Lambda function fail to start with an error message in the runtime\.log: 

  ```
  [ERROR]-container_process.go:39,Runtime execution error: unable to start lambda container. 
  container_linux.go:259: starting container process caused "process_linux.go:345: 
  container init caused \"rootfs_linux.go:62: mounting \\\"/var/run\\\" to rootfs \\\"/greengrass/ggc/packages/1.3.0/rootfs_sys\\\" at \\\"/greengrass/ggc/packages/1.3.0/rootfs_sys/run\\\" 
  caused \\\"invalid argument\\\"\""
  ```

  **A:** AWS IoT Greengrass core currently doesn't support the configuration of `/var`, `/var/run`, and `/var/lib` as volume resources\. One workaround is to first mount `/var`, `/var/run` or `/var/lib` in a different folder and then configure the folder as a volume resource\.
+ **Q:** When I configure `/dev/shm` as a volume resource with read\-only permission, why does the Lambda function fail to start with an error in the runtime\.log: 

  ```
  [ERROR]-container_process.go:39,Runtime execution error: unable to start lambda container. 
  container_linux.go:259: starting container process caused "process_linux.go:345: 
  container init caused \"rootfs_linux.go:62: mounting \\\"/dev/shm\\\" to rootfs \\\"/greengrass/ggc/packages/1.3.0/rootfs_sys\\\" at \\\"/greengrass/ggc/packages/1.3.0/rootfs_sys/dev/shm\\\" 
  caused \\\"operation not permitted\\\"\""”
  ```

  **A:** `/dev/shm` can only be configured as read/write\. Change the resource permission to `rw` to resolve the issue\.