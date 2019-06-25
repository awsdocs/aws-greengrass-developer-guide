# OTA Updates of AWS IoT Greengrass Core Software<a name="core-ota-update"></a>

This feature is available for AWS IoT Greengrass Core v1\.3 and later\.

The AWS IoT Greengrass Core software comes packaged with an OTA Update Agent that is capable of updating the core's software or the OTA Update Agent itself to the latest respective versions\. You can start an update by invoking the CreateSoftwareUpdateJob API or from the Greengrass console \. Updating the Greengrass core software provides the following benefits:
+ Fix security vulnerabilities\.
+ Address software stability issues\.
+ Deploy new or improved features\.

You do not have perform manual steps or have the device that is running the core software physically present\. In the event of a failed update, the OTA Update agent performs a rollback\. 

To support OTA updates of Greengrass core software, your Greengrass core device must:
+ Have available local storage three times the amount of the core's runtime usage requirement\. 
+ Not have trusted boot enabled in the partition that contains the Greengrass Core platform software\. \(The AWS IoT Greengrass core can be installed and run on a partition with trusted boot enabled, but cannot perform an OTA update\.\) 
+ Have read/write permissions on the partition containing the Greengrass Core platform software\.
+ Have a connection to the AWS Cloud\.
+ Have a correctly configured AWS IoT Greengrass core and appropriate certificates\.

Before you launch an OTA update of Greengrass Core software, be aware of the impact on the devices in your Greengrass group, both on the core device and on client devices connected locally to that core:
+ The core shuts down during the update\.
+ Any Lambda functions running on the core are shut down\. If those functions write to local resources, they might leave those resources in an incorrect state unless shut down properly\.
+ During the core's downtime, all its connections with the cloud are lost\. Messages routed through the core by client devices are lost\.
+ Credential caches are lost\.
+ Queues that hold pending work for Lambda functions are lost\.
+ Long\-lived Lambda functions lose their dynamic state information and all pending work is dropped\. 

The following state information is preserved during an OTA update:
+ Local shadows
+ Greengrass logs
+ OTA agent logs

## Greengrass OTA Agent<a name="ota-agent"></a>

The Greengrass OTA agent is the software component on the device that handles update jobs created and deployed in the cloud\. The Greengrass OTA agent is distributed in the same software package as the Greengrass Core software\. The agent is located in `./greengrass/ota/ota_agent/ggc-ota`\. It creates its logs in `/var/log/greengrass/ota/ggc_ota.txt`\.

You can start the Greengrass OTA agent by executing the binary manually or by integrating it as part of an init script such as a systemd service file\. The binary should be run as root\. When it starts, the Greengrass OTA agent listens for Greengrass update jobs from the cloud and executes them sequentially\. The Greengrass OTA agent ignores all other IoT job types\.

Do not start multiple OTA agent instances because this might cause conflicts\.

If your Greengrass core or Greengrass OTA agent is managed by an init system, see [Integration with Init Systems](#integration-with-init) for related configurations\.

**CreateSoftwareUpdateJob API**

The `CreateSoftwareUpdateJob` API creates a software update for a core or for several cores\. This API can be used to update the OTA agent and the Greengrass Core software\. It makes use of the AWS IoT jobs, which provide additional commands to manage a Greengrass core software update job\. For more information, see [Jobs](https://docs.aws.amazon.com/iot/latest/developerguide/iot-jobs.html)\. 

The following example shows how to use the CLI to create a Greengrass core software update job: 

```
aws greengrass create-software-update-job \
    --update-targets-architecture x86_64 \
    --update-targets arn:aws:iot:us-east-1:123456789012:thing/myDevice \
    --update-targets-operating-system ubuntu \
    --software-to-update core \
    --s3-url-signer-role arn:aws:iam::123456789012:role/IotS3UrlPresigningRole \
    --update-agent-log-level WARN \
    --amzn-client-token myClientToken1
```

The create\-software\-update\-job command returns a JSON object that contains the job ID and job ARN:

```
{
    "IotJobId": "Greengrass-OTA-c3bd7f36-ee80-4d42-8321-a1da0e5b1303",
    "IotJobArn": "arn:aws:iot:us-east-1:123456789012:job/Greengrass-OTA-c3bd7f36-ee80-4d42-8321-a1da0e5b1303"
}
```

The create\-software\-update\-job command has the following parameters:

`--update-targets-architecture`  
The architecture of the core device\. Must be one of `armv7l`, `x86_64`, or `aarch64`\.

`--update-targets`  
A list of the targets to which the OTA update should be applied\. The list can contain the ARNs of things that are cores, and the ARNs of thing groups whose members are cores\. For more information, see [IoT Thing Groups](https://docs.aws.amazon.com/iot/latest/developerguide/thing-groups.html)\. 

`--update-targets-operating-system`  
The operating system of the core device\. Must be one of `ubuntu`, `amazon_linux`, `raspbian`, or `openwrt`\.

`--software-to-update`  
Specifies whether the core's software or the OTA agent software should be updated\. Must be one of `core` or `ota_agent`\.

`--s3-url-signer-role`  <a name="s3-url-signer-role"></a>
The IAM role that is used to presign the S3 URL that links to the Greengrass software update\. You must provide a role that has the appropriate policy attached\. Here is an example policy document with the minimum required permissions:  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowsIotToAccessGreengrassOTAUpdateArtifacts",
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
\(Optional\) The logging level for log statements generated by the OTA agent\. Must be one of `NONE`, `TRACE`, `DEBUG`, `VERBOSE`, `INFO`, `WARN`, `ERROR`, or `FATAL`\. The default is `ERROR`\.

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
openwrt/aarch64
openwrt/armv7l

## Integration with Init Systems<a name="integration-with-init"></a>

During an OTA update, binaries, some of which may be running, will be updated and restarted\. This may cause conflicts if an init system is monitoring the state of either the AWS IoT Greengrass Core software or the Greengrass OTA Agent during the update\. To help integrate the OTA update mechanism with your monitoring strategies, Greengrass provides the opportunity for user\-defined shell scripts to run before and after an update\. To tell the OTA agent to run these shell scripts, you must include the `managedRespawn = true` flag in the `./greengrass/config/config.json` file\. For example:

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

When the `managedRespawn` flag is set, the scripts must exist in the directory\. Otherwise, the update fails\. The directory tree should look like the following:

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

As the OTA agent prepares to do a self\-update, if `managedRespawn` is set to `true`, the OTA agent looks in the `./greengrass/usr/scripts` directory for the `ota_pre_update.sh` script and runs it\.

After the OTA agent completes the update, it attempts to run the `ota_post_update.sh` script from the `./greengrass/usr/scripts` directory\.

### AWS IoT Greengrass Core Update with Managed Respawn<a name="managed-respawn-ggc"></a>

As the OTA agent prepares to do an AWS IoT Greengrass core update, if `managedRespawn` is set to `true`, the OTA agent looks in the `./greengrass/usr/scripts` directory for the `ggc_pre_update.sh` script and runs it\.

After the OTA agent completes the update, it attempts to run the `ggc_post_update.sh` script from the `./greengrass/usr/scripts` directory\.
+ The user\-defined scripts in `./greengrass/usr/scripts` should be owned by root and executable by root only\.
+ If `managedRespawn` is set to `true`, the scripts must exist and return a successful return code\.
+ If `managedRespawn` is set to `false`, the scripts do not run even if present on the device\.
+ A device that is the target of an update must not run two OTA agents for the same AWS IoT thing\. Doing so causes the two agents to process the same jobs, which creates conflicts\.

## OTA Agent Self\-Update<a name="agent-self-update"></a>

Follow these steps to perform an OTA Agent self\-update:

1. Ensure that the AWS IoT Greengrass core is correctly provisioned with valid `config.json` file entries and the required certificates\.

1. If the OTA agent is being managed by an init system, in the `config.json` file, make sure that `managedRespawn = true`, Make sure the `ota_pre_update.sh` and `ota_post_update.sh` scripts are in the `./greengrass/usr/scripts` directory\.

1. Run `./greengrass/ota/ota_agent/ggc-ota`\.

1. Use the `CreateSoftwareUpdateJob` API to create an OTA self\-update job \. Make sure the `--software-to-update` parameter is set to `ota_agent`\.

## Greengrass Core Software Update<a name="ggc-update"></a>

To perform an AWS IoT Greengrass Core software update follow these steps:

1. Make sure that the AWS IoT Greengrass core is correctly provisioned with valid `config.json` file entries and the required certificates\.

1. If the AWS IoT Greengrass Core software is being managed by an init system, ensure that `managedRespawn = true` in the `config.json` file and the scripts `ggc_pre_update.sh` and `ggc_post_update.sh` are present in the `./greengrass/usr/scripts` directory\.

1. Run `./greengrass/ota/ota_agent/ggc-ota`\.

1. Create an OTA self update job in the cloud with the CreateSoftwareUpdateJob API \(`aws greengrass create-software-update-job`\), making sure the `--software-to-update` parameter is set to `core`\.

1. The OTA Agent will perform an update of AWS IoT Greengrass Core software\.