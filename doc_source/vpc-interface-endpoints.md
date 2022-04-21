--------

AWS IoT Greengrass Version 1 no longer receives feature updates, and will receive only security patches and bug fixes until June 30, 2023\. For more information, see the [AWS IoT Greengrass V1 maintenance policy](https://docs.aws.amazon.com/greengrass/v1/developerguide/maintenance-policy.html)\. We strongly recommend that you [migrate to AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/move-from-v1.html), which adds [significant new features](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html) and [support for additional platforms](https://docs.aws.amazon.com/greengrass/v2/developerguide/operating-system-feature-support-matrix.html)\.

--------

# AWS IoT Greengrass and interface VPC endpoints \(AWS PrivateLink\)<a name="vpc-interface-endpoints"></a>

You can establish a private connection between your VPC and the AWS IoT Greengrass control plane by creating an *interface VPC endpoint*\. You can use this endpoint to manage groups, Lambda functions, deployments, and other resources in the AWS IoT Greengrass service\. Interface endpoints are powered by [AWS PrivateLink](http://aws.amazon.com/privatelink), a technology that enables you to access AWS IoT Greengrass APIs privately without an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection\. Instances in your VPC don't need public IP addresses to communicate with AWS IoT Greengrass APIs\. Traffic between your VPC and AWS IoT Greengrass does not leave the Amazon network\.

**Note**  
Currently, you can't configure Greengrass core devices to operate completely within your VPC\.

Each interface endpoint is represented by one or more [Elastic Network Interfaces](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) in your subnets\. 

For more information, see [Interface VPC endpoints \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html) in the *Amazon VPC User Guide*\.

**Topics**
+ [Considerations for AWS IoT Greengrass VPC endpoints](#vpc-endpoint-considerations)
+ [Create an interface VPC endpoint for AWS IoT Greengrass control plane operations](#create-vpc-endpoint-control-plane)
+ [Creating a VPC endpoint policy for AWS IoT Greengrass](#vpc-endpoint-policy)

## Considerations for AWS IoT Greengrass VPC endpoints<a name="vpc-endpoint-considerations"></a>

Before you set up an interface VPC endpoint for AWS IoT Greengrass, review [Interface endpoint properties and limitations](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#vpce-interface-limitations) in the *Amazon VPC User Guide*\. Additionally, be aware of the following considerations:
+ AWS IoT Greengrass supports making calls to all of its control plane API actions from your VPC\. The control plane includes operations such as [CreateDeployment](https://docs.aws.amazon.com/greengrass/v1/apireference/createdeployment-post.html) and [StartBulkDeployment](https://docs.aws.amazon.com/greengrass/v1/apireference/startbulkdeployment-post.html)\. The control plane does *not* include operations such as [GetDeployment](device-auth.md#iot-policies) and [Discover](gg-discover-api.md), which are data plane operations\.
+ VPC endpoints for AWS IoT Greengrass are currently not supported in AWS China Regions\.

## Create an interface VPC endpoint for AWS IoT Greengrass control plane operations<a name="create-vpc-endpoint-control-plane"></a>

You can create a VPC endpoint for the AWS IoT Greengrass control plane using either the Amazon VPC console or the AWS Command Line Interface \(AWS CLI\)\. For more information, see [Creating an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#create-interface-endpoint) in the *Amazon VPC User Guide*\.

Create a VPC endpoint for AWS IoT Greengrass using the following service name: 
+ com\.amazonaws\.*region*\.greengrass

If you enable private DNS for the endpoint, you can make API requests to AWS IoT Greengrass using its default DNS name for the Region, for example, `greengrass.us-east-1.amazonaws.com`\. Private DNS is enabled by default\.

For more information, see [Accessing a service through an interface endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/vpce-interface.html#access-service-though-endpoint) in the *Amazon VPC User Guide*\.

## Creating a VPC endpoint policy for AWS IoT Greengrass<a name="vpc-endpoint-policy"></a>

You can attach an endpoint policy to your VPC endpoint that controls access to AWS IoT Greengrass control plane operations\. The policy specifies the following information:
+ The principal that can perform actions\.
+ The actions that the principal can perform\.
+ The resources that the principal can perform actions on\.

For more information, see [Controlling access to services with VPC endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-access.html) in the *Amazon VPC User Guide*\.

**Example: VPC endpoint policy for AWS IoT Greengrass actions**  
The following is an example of an endpoint policy for AWS IoT Greengrass\. When attached to an endpoint, this policy grants access to the listed AWS IoT Greengrass actions for all principals on all resources\.  

```
{
    "Statement": [
        {
            "Principal": "*",
            "Effect": "Allow",
            "Action": [
                "greengrass:CreateDeployment",
                "greengrass:StartBulkDeployment"
            ],
            "Resource": "*"
        }
    ]
}
```