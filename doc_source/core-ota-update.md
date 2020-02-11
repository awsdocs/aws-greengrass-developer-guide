# OTA Updates of AWS IoT Greengrass Core Software<a name="core-ota-update"></a>

This feature is available for AWS IoT Greengrass Core v1\.3 and later\.

The AWS IoT Greengrass Core software is packaged with an agent that can update the core's software or the agent itself to the latest version\. These updates are sent over the air \(OTA\)\. OTA updates are the recommended way to update AWS IoT Greengrass software on your AWS IoT Greengrass core devices\. You can use the AWS IoT console or the `CreateSoftwareUpdateJob` API to start an update\. By using an OTA update, you can:
+ Fix security vulnerabilities\.
+ Address software stability issues\.
+ Deploy new or improved features\.

You do not have to perform manual steps or have the device that is running the Core software physically present\. In the event of a failed update, the OTA update agent performs a rollback\. 

To support OTA updates of AWS IoT Greengrass software, your Greengrass core device must:
+ Have available local storage three times the amount of the core's runtime usage requirement\. For more information, see [Service Quotas](https://docs.aws.amazon.com/general/latest/gr/greengrass.html#limits_greengrass) for the AWS IoT Greengrass core in the *Amazon Web Services General Reference*\.
+ Not have trusted boot enabled in the partition that contains the Greengrass Core platform software\. \(The AWS IoT Greengrass core can be installed and run on a partition with trusted boot enabled, but cannot perform an OTA update\.\) 
+ Have read/write permissions on the partition that contains the Greengrass Core platform software\.
+ Not be configured to use a network proxy\. In AWS IoT Greengrass Core v1\.9\.3 or later, the OTA update agent supports updates over port 443 when MQTT traffic is configured to use port 443 instead of the default port 8883\. However, the OTA update agent does not support updates through a network proxy\. For more information, see [Connect on Port 443 or Through a Network Proxy](gg-core.md#alpn-network-proxy)\.
+ Have a connection to the AWS Cloud\.
+ Have a correctly configured AWS IoT Greengrass core and appropriate certificates\.

Before you launch an OTA update of Greengrass Core software, be aware of the impact on the devices in your Greengrass group, both on the core device and on client devices connected locally to that core:
+ The core shuts down during the update\.
+ Any Lambda functions running on the core are shut down\. If those functions write to local resources, they might leave those resources in an incorrect state unless shut down properly\.
+ During the core's downtime, all its connections with the AWS Cloud are lost\. Messages routed through the core by client devices are lost\.
+ Credential caches are lost\.
+ Queues that hold pending work for Lambda functions are lost\.
+ Long\-lived Lambda functions lose their dynamic state information and all pending work is dropped\. 

The following state information is preserved during an OTA update:
+ Local shadows
+ Greengrass logs
+ OTA update agent logs

## Greengrass OTA Update Agent<a name="ota-agent"></a>

The Greengrass OTA update agent is the software component on the device that handles update jobs created and deployed in the cloud\. The Greengrass OTA update agent is distributed in the same software package as the AWS IoT Greengrass Core software\. The agent is located in `./greengrass/ota/ota_agent/ggc-ota`\. It creates its logs in `/var/log/greengrass/ota/ggc_ota.txt`\.

You can start the Greengrass OTA update agent by executing the binary manually or by integrating it as part of an init script such as a systemd service file\. The binary should be run as root\. When it starts, the Greengrass OTA update agent listens for Greengrass update jobs from the cloud and executes them sequentially\. The Greengrass OTA update agent ignores all other IoT job types\.

Do not start multiple OTA update agent instances because this might cause conflicts\.

If your Greengrass core or Greengrass OTA update agent is managed by an init system, see [Integration with Init Systems](#integration-with-init) for related configurations\.

**CreateSoftwareUpdateJob API**

The `CreateSoftwareUpdateJob` API creates a software update for a core or for several cores\. This API can be used to update the OTA update agent and the Greengrass Core software\. It makes use of AWS IoT jobs, which provide other commands to manage a software update job on a Greengrass core\. For more information, see [Jobs](https://docs.aws.amazon.com/iot/latest/developerguide/iot-jobs.html)\. 

The following example shows how to use the CLI to create a job that updates the AWS IoT Greengrass Core software on a core device:

```
aws greengrass create-software-update-job \
    --update-targets-architecture x86_64 \
    --update-targets arn:aws::iot:region:123456789012:thing/myDevice \
    --update-targets-operating-system ubuntu \
    --software-to-update core \
    --s3-url-signer-role arn:aws::iam::123456789012:role/IotS3UrlPresigningRole \
    --update-agent-log-level WARN \
    --amzn-client-token myClientToken1
```

The `create-software-update-job` command returns a JSON response that contains the job ID, job ARN, and software version that was installed by the update:

```
{
    "IotJobId": "Greengrass-OTA-c3bd7f36-ee80-4d42-8321-a1da0e5b1303",
    "IotJobArn": "arn:aws::iot:region:123456789012:job/Greengrass-OTA-c3bd7f36-ee80-4d42-8321-a1da0e5b1303",
    "PlatformSoftwareVersion": "1.9.2"
}
```

The `create-software-update-job` command has the following parameters:

`--update-targets-architecture`  
The architecture of the core device\. Must be one of `armv7l`, `armv6l`, `x86_64`, or `aarch64`\.

`--update-targets`  
A list of the targets to which the OTA update should be applied\. The list can contain the ARNs of things that are cores, and the ARNs of thing groups whose members are cores\. For more information, see [IoT Thing Groups](https://docs.aws.amazon.com/iot/latest/developerguide/thing-groups.html)\. 

`--update-targets-operating-system`  
The operating system of the core device\. Must be one of `ubuntu`, `amazon_linux`, `raspbian`, or `openwrt`\.

`--software-to-update`  
Specifies whether the core's software or the OTA update agent software should be updated\. Must be one of `core` or `ota_agent`\.

`--s3-url-signer-role`  <a name="s3-url-signer-role"></a>
The IAM role used to presign the S3 URL that links to the AWS IoT Greengrass software update\. You must provide a role that has the appropriate permissions policy attached\. The following example policy allows access to AWS IoT Greengrass software updates in the specified AWS Regions:  

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
You can also use a wildcard \* naming scheme for the `Resource` property to allow access to AWS IoT Greengrass software updates\. For example, the following format allows access to software updates for all [supported AWS Regions](https://docs.aws.amazon.com/general/latest/gr/greengrass.html) \(current and future\) that use the `aws` partition\. Make sure to use the correct partitions for the AWS Regions you want to support\.  

```
"Resource": "arn:aws:s3:::*-greengrass-updates/*"
```
 For more information, see [ Adding and Removing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html) in the *IAM User Guide*\.
Here is an example `AssumeRole` policy document with the minimum required trusted entities:  

```
{
     "Version": "2012-10-17",
     "Statement": [
         {
             "Action": "sts:AssumeRole",
             "Principal": {
                  "Service": "iot.amazonaws.com"
             },
             "Effect": "Allow",
             "Sid": "AllowIotToAssumeRole"
        }
    ]
}
```

`--amzn-client-token`  
\(Optional\) A client token used to make idempotent requests\. Provide a unique token to prevent duplicate updates from being created due to internal retries\. 

`--update-agent-log-level`  
\(Optional\) The logging level for log statements generated by the OTA update agent\. Must be one of `NONE`, `TRACE`, `DEBUG`, `VERBOSE`, `INFO`, `WARN`, `ERROR`, or `FATAL`\. The default is `ERROR`\.

Here is an example IAM policy with the minimum permissions required to call the API: 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCreateSoftwareUpdateJob",
            "Action": [
                "greengrass:CreateSoftwareUpdateJob"
            ],
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn:aws:s3:us-east-1:123456789012:role/IotS3UrlPresigningRole"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iot:CreateJob"
            ],
            "Resource": "*"
        }
    ]
}
```

**Note**  
 Because AWS IoT Greengrass is supported only on a subset of the architecture and operating system combinations possible with this command, `CreateSoftwareUpdateJob` rejects requests except for the following supported platforms:   
ubuntu/x86\_64 
ubuntu/aarch64 
amazon\_linux/x86\_64 
raspbian/armv7l 
raspbian/armv6l
openwrt/aarch64
openwrt/armv7l

## Integration with Init Systems<a name="integration-with-init"></a>

During an OTA update, binaries, including some that are running, are updated and restarted\. This might cause conflicts if an init system is monitoring the state of either the AWS IoT Greengrass Core software or the Greengrass OTA update agent during the update\. To help integrate the OTA update mechanism with your monitoring strategies, you can write shell scripts that run before and after an update\. To tell the OTA update agent to run these shell scripts, you must include the `"managedRespawn" : true` flag in the `./greengrass/config/config.json` file\. For example:

```
{

   "coreThing": {                                                     
       …
   },
   "runtime": {                                                       
        …
   },
   "managedRespawn": true                                            

}
```

When `managedRespawn` is set to `true`, the scripts must exist in the directory\. Otherwise, the update fails\. The directory tree should look like the following:

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

### OTA Self\-Update with Managed Respawn<a name="managed-respawn"></a>

As the OTA update agent prepares to do a self\-update, if `managedRespawn` is set to `true`, the OTA update agent looks in the `./greengrass/usr/scripts` directory for the `ota_pre_update.sh` script and runs it\.

After the OTA update agent completes the update, it attempts to run the `ota_post_update.sh` script from the `./greengrass/usr/scripts` directory\.

### AWS IoT Greengrass Core Update with Managed Respawn<a name="managed-respawn-ggc"></a>

As the OTA update agent prepares to do an AWS IoT Greengrass core update, if `managedRespawn` is set to `true`, the OTA update agent looks in the `./greengrass/usr/scripts` directory for the `ggc_pre_update.sh` script and runs it\.

After the OTA update agent completes the update, it attempts to run the `ggc_post_update.sh` script from the `./greengrass/usr/scripts` directory\.
+ The user\-defined scripts in `./greengrass/usr/scripts` should be owned by root and executable by root only\.
+ If `managedRespawn` is set to `true`, the scripts must exist and return a successful return code\.
+ If `managedRespawn` is set to `false`, the scripts do not run even if present on the device\.
+ A device that is the target of an update must not run two instances of the OTA update agent for the same AWS IoT thing\. Doing so causes the two agents to process the same jobs, which creates conflicts\.

## OTA Update Agent Self\-Update<a name="agent-self-update"></a>

Follow these steps to perform a self\-update of the OTA update agent:

1. Make sure that the AWS IoT Greengrass core device is correctly provisioned with valid `config.json` file entries and the required certificates\.

1. If the OTA update agent is managed by an init system, in the `config.json` file, make sure that `managedRespawn` property is set to `true`\. Also, make sure the `ota_pre_update.sh` and `ota_post_update.sh` scripts are in the `./greengrass/usr/scripts` directory\.

1. Run `./greengrass/ota/ota_agent/ggc-ota`\.

1. Use the `CreateSoftwareUpdateJob` API to create an OTA self\-update job\. Make sure the `--software-to-update` parameter is set to `ota_agent`\.

## Greengrass Core Software Update<a name="ggc-update"></a>

Follow these steps to perform an AWS IoT Greengrass Core software update:

1. Make sure that the AWS IoT Greengrass core device is correctly provisioned with valid `config.json` file entries and the required certificates\.

1. If the AWS IoT Greengrass Core software is managed by an init system, in the `config.json` file, make sure that `managedRespawn` property is set to `true`\. Also, make sure the `ggc_pre_update.sh` and `ggc_post_update.sh` scripts are in the `./greengrass/usr/scripts` directory\.

1. Run `./greengrass/ota/ota_agent/ggc-ota`\.

1. Use the `CreateSoftwareUpdateJob` API to create an update job for the Core software\. Make sure the `--software-to-update` parameter is set to `core`\.