# OTA updates of AWS IoT Greengrass Core software<a name="core-ota-update"></a>

The AWS IoT Greengrass Core software package includes an update agent that can perform over\-the\-air \(OTA\) updates of AWS IoT Greengrass software\. You can use OTA updates to install the latest version of the AWS IoT Greengrass Core software or OTA update agent software on one or more cores\. With OTA updates, your core devices don't have to be physically present\.

We recommend that you use OTA updates when possible\. They provide a mechanism you can use to track update status and update history\. If a failed update occurs, the OTA update agent rolls back to the previous software version\.

**Note**  
OTA updates are not supported when you use `apt` to install the AWS IoT Greengrass Core software\. For these installations, we recommend that you use `apt` to upgrade the software\. For more information, see [Install the AWS IoT Greengrass Core software from an APT repository](install-ggc.md#ggc-package-manager)\.

OTA updates make it more efficient to:
+ Fix security vulnerabilities\.
+ Address software stability issues\.
+ Deploy new or improved features\.

This feature integrates with [AWS IoT jobs](https://docs.aws.amazon.com/iot/latest/developerguide/iot-jobs.html)\.

## Requirements<a name="ota-update-requirements"></a>

The following requirements apply for OTA updates of AWS IoT Greengrass software\.
+ The Greengrass core must have at least <a name="req-core-ota-disk-space"></a>400 MB of disk space available in local storage\. The OTA update agent requires about three times the runtime usage requirement of the AWS IoT Greengrass Core software\. For more information, see [Service quotas](https://docs.aws.amazon.com/general/latest/gr/greengrass.html#limits_greengrass) for the Greengrass core in the *Amazon Web Services General Reference*\.
+ The Greengrass core must have a connection to the AWS Cloud\.
+ The Greengrass core must be correctly configured and provisioned with certificates and keys for authentication with AWS IoT Core and AWS IoT Greengrass\. For more information, see [X\.509 certificates](device-auth.md#x509-certificates)\.
+ The Greengrass core can't be configured to use a network proxy\.
**Note**  
Starting in AWS IoT Greengrass v1\.9\.3, OTA updates are supported on cores that configure MQTT traffic to use port 443 instead of the default port 8883\. However, the OTA update agent does not support updates through a network proxy\. For more information, see [Connect on port 443 or through a network proxy](gg-core.md#alpn-network-proxy)\.
+ Trusted boot can't be enabled in the partition that contains the AWS IoT Greengrass Core software\.
**Note**  
You can install and run the AWS IoT Greengrass Core software on a partition that has trusted boot enabled, but OTA updates aren't supported\.
+ AWS IoT Greengrass must have read/write permissions on the partition that contains the AWS IoT Greengrass Core software\.
+ If you use an init system to manage your Greengrass core, you must configure OTA updates to integrate with the init system\. For more information, see [Integration with init systems](#integration-with-init)\.
+ You must create a role that's used to presign the Amazon S3 URLs to AWS IoT Greengrass software update artifacts\. This signer role allows AWS IoT Core to access software update artifacts stored in Amazon S3 on your behalf\. For more information, see [IAM permissions for OTA updates](#ota-permissions)\.

### IAM permissions for OTA updates<a name="ota-permissions"></a>

When AWS IoT Greengrass releases a new version of the AWS IoT Greengrass Core software, AWS IoT Greengrass updates the software artifacts stored in Amazon S3 that are used for the OTA update\.

Your AWS account must include an Amazon S3 URL signer role that can be used to access these artifacts\. The role must have a permissions policy that allows the `s3:GetObject` action on the buckets in target AWS Regions\. The role must also have a trust policy that allows `iot.amazonaws.com` to assume the role as a trusted entity\.

**Permissions policy**  
For role permissions, you can use the AWS managed policy or create a custom policy\.  
+ **Use the AWS managed policy**

  The [ GreengrassOTAUpdateArtifactAccess](https://console.aws.amazon.com/iam/home?region=us-west-2#/policies/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2Fservice-role%2FGreengrassOTAUpdateArtifactAccess) managed policy is provided by AWS IoT Greengrass\. Use this policy if you want to allow access in all AWS Regions supported by AWS IoT Greengrass, both current and future\.
+ **Create a custom policy**

  You should create a custom policy if you want to explicitly specify the AWS Regions where your cores are deployed\. The following example policy allows access to AWS IoT Greengrass software updates in six Regions\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "AllowAccessToGreengrassOTAUpdateArtifacts",
              "Effect": "Allow",
              "Action": [
                  "s3:GetObject"
              ],
              "Resource": [
                  "arn:aws:s3:::us-east-1-greengrass-updates/*",
                  "arn:aws:s3:::us-west-2-greengrass-updates/*",
                  "arn:aws:s3:::ap-northeast-1-greengrass-updates/*",
                  "arn:aws:s3:::ap-southeast-2-greengrass-updates/*",
                  "arn:aws:s3:::eu-central-1-greengrass-updates/*",
                  "arn:aws:s3:::eu-west-1-greengrass-updates/*"
              ]
          }
      ]
  }
  ```

**Trust policy**  
The trust policy attached to the role must allow the `sts:AssumeRole` action and define `iot.amazonaws.com` as a principal\. This allows AWS IoT Core to assume the role as a trusted entity\. Here's an example policy document:  

```
{
     "Version": "2012-10-17",
     "Statement": [
         {
             "Sid": "AllowIotToAssumeRole",
             "Action": "sts:AssumeRole",
             "Principal": {
                  "Service": "iot.amazonaws.com"
             },
             "Effect": "Allow"
        }
    ]
}
```

In addition, the user who initiates an OTA update must have permissions to use `greengrass:CreateSoftwareUpdateJob` and `iot:CreateJob`, and to use `iam:PassRole` to pass the permissions of the signer role\. Here's an example IAM policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "greengrass:CreateSoftwareUpdateJob"
            ],
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iot:CreateJob"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn-of-s3-url-signer-role"
        }
    ]
}
```

## Considerations<a name="ota-updates-considerations"></a>

Before you launch an OTA update of Greengrass Core software, be aware of the impact on the devices in your Greengrass group, both on the core device and on client devices connected locally to that core:
+ The core shuts down during the update\.
+ Any Lambda functions running on the core are shut down\. If those functions write to local resources, they might leave those resources in an incorrect state unless shut down properly\.
+ During the core's downtime, all its connections with the AWS Cloud are lost\. Messages routed through the core by client devices are lost\.
+ Credential caches are lost\.
+ Queues that hold pending work for Lambda functions are lost\.
+ Long\-lived Lambda functions lose their dynamic state information and all pending work is dropped\. 

The following state information is preserved during an OTA update:
+ Core configuration
+ Greengrass group configuration
+ Local shadows
+ Greengrass logs
+ OTA update agent logs

## Greengrass OTA update agent<a name="ota-agent"></a>

The Greengrass OTA update agent is the software component on the device that handles update jobs created and deployed in the cloud\. The OTA update agent is distributed in the same software package as the AWS IoT Greengrass Core software\. The agent is located in `/greengrass-root/ota/ota_agent/ggc-ota`\. It writes logs to `/var/log/greengrass/ota/ggc_ota.txt`\.

**Note**  
*greengrass\-root* represents the path where the AWS IoT Greengrass Core software is installed on your device\. Typically, this is the `/greengrass` directory\.

You can start the OTA update agent by executing the binary manually or by integrating it as part of an init script, such as a systemd service file\. The binary should be run as root\. When it starts, the OTA update agent listens for AWS IoT Greengrass software update jobs from AWS IoT Core and executes them sequentially\. The OTA update agent ignores all other AWS IoT job types\.

A core that is the target of an update must not run two instances of the OTA update agent\. Doing so causes the two agents to process the same jobs, which creates conflicts\.

## Integration with init systems<a name="integration-with-init"></a>

During an OTA update, the OTA update agent restarts binaries on the core\. If the binaries are running, this might cause conflicts when an init system is monitoring the state of the AWS IoT Greengrass Core software or the agent during the update\.

To help integrate the OTA update mechanism with your init monitoring strategies, you can write shell scripts that run before and after an update\. For example, you can write a script that backs up data or stops processes before the device shuts down\. To tell the OTA update agent to run these scripts, you must include the `"managedRespawn" : true` flag in the [config\.json](gg-core.md#config-json) file\. This setting is shown in the following excerpt:

```
{
   "coreThing": {                                                     
       …
   },
   "runtime": {                                                       
       …
   },
   "managedRespawn": true
       …
}
```

When `managedRespawn` is set to `true`, the OTA update agent runs the scripts from the \./*greengrass\-root*/usr/scripts directory\. The directory tree should look like the following:

```
<greengrass_root>
|-- certs
|-- config
|   |-- config.json
|-- ggc
|-- usr/scripts
|   |-- ggc_pre_update.sh
|   |-- ggc_post_update.sh
|   |-- ota_pre_update.sh
|   |-- ota_post_update.sh
|-- ota
```

### Managed respawn with OTA updates<a name="ota-managed-respawn"></a>

If `managedRespawn` is set to `true`, the OTA update agent checks the /*greengrass\-root*/usr/scripts directory for the scripts before and after the software update\. If the scripts don't exist, the update fails\.
+ For OTA updates of the AWS IoT Greengrass Core software:

  Before starting the update, the agent runs the `ggc_pre_update.sh` script\. After completing the update, the agent runs the `ggc_post_update.sh` script\.
+ For OTA updates of the OTA update agent software:

  Before starting the update, the agent runs the `ota_pre_update.sh` script\. After completing the update, the agent runs the `ota_post_update.sh` script\.

If `managedRespawn` is set to `true`, the following requirements apply:
+ You must add the following scripts to the /*greengrass\-root*/usr/scripts directory:
  + ggc\_pre\_update\.sh
  + ggc\_post\_update\.sh
  + ota\_pre\_update\.sh
  + ota\_post\_update\.sh
+ The scripts must return a successful return code\.
+ The scripts must be owned by root and executable by root only\.

**Note**  
If `managedRespawn` is set to `false`, the OTA update agent does not run the scripts\.

## Create an OTA update<a name="create-ota-update"></a><a name="ggc-update"></a>

Follow these steps to perform an OTA update of AWS IoT Greengrass software on one or more cores:

1. Make sure that your cores meet the [requirements](#ota-update-requirements) for OTA updates\.
**Note**  
If you configured an init system to manage the AWS IoT Greengrass Core software or the OTA update agent, verify the following on your cores:  
The [config\.json](gg-core.md#config-json) file specifies `"managedRespawn" : true`\.
The /*greengrass\-root*/usr/scripts directory contains the following scripts:  
ggc\_pre\_update\.sh
ggc\_post\_update\.sh
ota\_pre\_update\.sh
ota\_post\_update\.sh
For more information, see [Integration with init systems](#integration-with-init)\.

1. In a core device terminal, start the OTA update agent\.

   ```
   cd /greengrass-root/ota/ota_agent
   sudo ./ggc-ota
   ```
**Note**  
*greengrass\-root* represents the path where the AWS IoT Greengrass Core software is installed on your device\. Typically, this is the `/greengrass` directory\.

   Don't start multiple instances of the OTA update agent on a core because it might cause conflicts\.

1. Use the AWS IoT console or AWS IoT Greengrass API to create a software update job\.

      
**Use the console**  

   1. In the AWS IoT console, choose **Manage**, and then choose **Jobs**\.

   1. Choose **Create**, and then choose **Create Core update job**\.

   1. On the **Create a Greengrass update** page, define the properties for the update job, and then choose **Create**\. For example:
      + For **Select devices to update**, choose the cores to update\. You can select individual core things and thing groups that contain cores\.
      + For **S3 URL Signer Role**, choose your [signer role](#ota-permissions)\.
      + For **Select what component of the Greengrass Core you want to update**, choose to update the AWS IoT Greengrass Core software or **** to update the OTA update agent software\.

   1. On the **Jobs** page, choose your new job to see the update status\.
   
**Use the API**  

   1. Call the [CreateSoftwareUpdateJob](#create-software-update-job) API\. In this example procedure, we use AWS CLI commands\.

      The following command creates a job that updates the AWS IoT Greengrass Core software on one core\. Replace the example values and then run the command\.

------
#### [ Linux or macOS terminal ]

      ```
      aws greengrass create-software-update-job \
      --update-targets-architecture x86_64 \
      --update-targets [\"arn:aws:iot:region:123456789012:thing/myCoreDevice\"] \
      --update-targets-operating-system ubuntu \
      --software-to-update core \
      --s3-url-signer-role arn:aws:iam::123456789012:role/myS3UrlSignerRole \
      --update-agent-log-level WARN \
      --amzn-client-token myClientToken1
      ```

------
#### [ Windows command prompt ]

      ```
      aws greengrass create-software-update-job ^
      --update-targets-architecture x86_64 ^
      --update-targets [\"arn:aws:iot:region:123456789012:thing/myCoreDevice\"] ^
      --update-targets-operating-system ubuntu ^
      --software-to-update core ^
      --s3-url-signer-role arn:aws:iam::123456789012:role/myS3UrlSignerRole ^
      --update-agent-log-level WARN ^
      --amzn-client-token myClientToken1
      ```

------

      The command returns the following response\.

      ```
      {
          "IotJobId": "GreengrassUpdateJob_c3bd7f36-ee80-4d42-8321-a1da0EXAMPLE",
          "IotJobArn": "arn:aws:iot:region:123456789012:job/GreengrassUpdateJob_c3bd7f36-ee80-4d42-8321-a1da0EXAMPLE",
          "PlatformSoftwareVersion": "1.10.1"
      }
      ```

   1. Copy the `IoTJobId` from the response\.

   1. Call [DescribeJob](https://docs.aws.amazon.com/iot/latest/developerguide/manage-job-cli.html#describe-job) in the AWS IoT Core API to see the job status\. Replace the example value with your job ID and then run the command\.

      ```
      aws iot describe-job --job-id GreengrassUpdateJob_c3bd7f36-ee80-4d42-8321-a1da0EXAMPLE
      ```

      The command returns a response object that contains information about the job, including `status` and `jobProcessDetails`\.

      ```
      {
          "job": {
              "jobArn": "arn:aws:iot:region:123456789012:job/GreengrassUpdateJob_c3bd7f36-ee80-4d42-8321-a1da0EXAMPLE",
              "jobId": "GreengrassUpdateJob_c3bd7f36-ee80-4d42-8321-a1da0EXAMPLE",
              "targetSelection": "SNAPSHOT",
              "status": "IN_PROGRESS",
              "targets": [
                  "arn:aws:iot:region:123456789012:thing/myCoreDevice"
              ],
              "description": "This job was created by Greengrass to update the Greengrass Cores in the targets with version 1.10.1 of the core software running on x86_64 architecture.",
              "presignedUrlConfig": {
                  "roleArn": "arn:aws::iam::123456789012:role/myS3UrlSignerRole",
                  "expiresInSec": 3600
              },
              "jobExecutionsRolloutConfig": {},
              "createdAt": 1588718249.079,
              "lastUpdatedAt": 1588718253.419,
              "jobProcessDetails": {
                  "numberOfCanceledThings": 0,
                  "numberOfSucceededThings": 0,
                  "numberOfFailedThings": 0,
                  "numberOfRejectedThings": 0,
                  "numberOfQueuedThings": 1,
                  "numberOfInProgressThings": 0,
                  "numberOfRemovedThings": 0,
                  "numberOfTimedOutThings": 0
              },
              "timeoutConfig": {}
          }
      }
      ```

For troubleshooting help, see [Troubleshooting AWS IoT Greengrass](gg-troubleshooting.md)\.

## CreateSoftwareUpdateJob API<a name="create-software-update-job"></a>

You can use the `CreateSoftwareUpdateJob` API to update the AWS IoT Greengrass Core software or OTA update agent software on your core devices\. This API creates an AWS IoT snapshot job that notifies devices when an update is available\. After you call `CreateSoftwareUpdateJob`, you can use other AWS IoT job commands to track the software update\. For more information, see [Jobs](https://docs.aws.amazon.com/iot/latest/developerguide/iot-jobs.html) in the *AWS IoT Developer Guide*\.

The following example shows how to use the AWS CLI to create a job that updates the AWS IoT Greengrass Core software on a core device:

```
aws greengrass create-software-update-job \
--update-targets-architecture x86_64 \
--update-targets [\"arn:aws:iot:region:123456789012:thing/myCoreDevice\"] \
--update-targets-operating-system ubuntu \
--software-to-update core \
--s3-url-signer-role arn:aws:iam::123456789012:role/myS3UrlSignerRole \
--update-agent-log-level WARN \
--amzn-client-token myClientToken1
```

The `create-software-update-job` command returns a JSON response that contains the job ID, job ARN, and software version that was installed by the update:

```
{
    "IotJobId": "GreengrassUpdateJob_c3bd7f36-ee80-4d42-8321-a1da0EXAMPLE",
    "IotJobArn": "arn:aws:iot:region:123456789012:job/GreengrassUpdateJob_c3bd7f36-ee80-4d42-8321-a1da0EXAMPLE",
    "PlatformSoftwareVersion": "1.9.2"
}
```

For steps that show you how to use `create-software-update-job` to update a core device, see [Create an OTA update](#create-ota-update)\.

The `create-software-update-job` command has the following parameters:

`--update-targets-architecture`  
The architecture of the core device\.  
Valid values: `armv7l`, `armv6l`, `x86_64`, or `aarch64`

`--update-targets`  
The cores to update\. The list can contain ARNs of individual cores and ARNs of thing groups whose members are cores\. For more information about thing groups, see [Static thing groups](https://docs.aws.amazon.com/iot/latest/developerguide/thing-groups.html) in the *AWS IoT Developer Guide*\.

`--update-targets-operating-system`  
The operating system of the core device\.  
Valid values: `ubuntu`, `amazon_linux`, `raspbian`, or `openwrt`

`--software-to-update`  
Specifies whether the core's software or the OTA update agent software should be updated\.  
Valid values: `core` or `ota_agent`

`--s3-url-signer-role`  <a name="s3-url-signer-role"></a>
The ARN of the IAM role used to presign the Amazon S3 URL that links to the AWS IoT Greengrass software update artifacts\. The role's attached permissions policy must allow the `s3:GetObject` action on the buckets in the target AWS Regions\. The role must also allow `iot.amazonaws.com` to assume the role as a trusted entity\. For more information, see [IAM permissions for OTA updates](#ota-permissions)\.

`--amzn-client-token`  
\(Optional\) A client token used to make idempotent requests\. Provide a unique token to prevent duplicate updates from being created because of internal retries\. 

`--update-agent-log-level`  
\(Optional\) The logging level for log statements generated by the OTA update agent\. The default is `ERROR`\.  
Valid values: `NONE`, `TRACE`, `DEBUG`, `VERBOSE`, `INFO`, `WARN`, `ERROR`, or `FATAL`

**Note**  
`CreateSoftwareUpdateJob` accepts requests only for the following supported architecture and operating system combinations:  
ubuntu/x86\_64 
ubuntu/aarch64 
amazon\_linux/x86\_64 
raspbian/armv7l 
raspbian/armv6l
openwrt/aarch64
openwrt/armv7l