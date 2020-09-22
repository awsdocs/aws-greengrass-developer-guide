# Setting configuration to run the AWS IoT Greengrass qualification suite<a name="set-config"></a>

Before you run tests, you must configure settings for AWS credentials and devices on your host computer\.

## Configure your AWS credentials<a name="cfg-aws-gg"></a>

You must configure your IAM user credentials in the `<device_tester_extract_location> /configs/config.json` file\. Use the credentials for the IDT for AWS IoT Greengrass user created in [Create and configure an AWS account](dev-tst-prereqs.md#config-aws-account-for-idt)\. You can specify your credentials in one of two ways:
+ Credentials file
+ Environment variables

### Configure AWS credentials with a credentials file<a name="config-cred-file"></a>

IDT uses the same credentials file as the AWS CLI\. For more information, see [Configuration and credential files](https://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html)\.

The location of the credentials file varies, depending on the operating system you are using:
+ macOS, Linux: `~/.aws/credentials`
+ Windows: `C:\Users\UserName\.aws\credentials`

Add your AWS credentials to the `credentials` file in the following format:

```
[default]
aws_access_key_id = <your_access_key_id>
aws_secret_access_key = <your_secret_access_key>
```

To configure IDT for AWS IoT Greengrass to use AWS credentials from your `credentials` file, edit your `config.json` file as follows:

```
{
	"awsRegion": "us-west-2",
	"auth": {
		"method": "file",
		"credentials": {
			"profile": "default"
		}
	}
}
```

**Note**  
If you do not use the `default` AWS profile, be sure to change the profile name in your `config.json` file\. For more information, see [Named profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)\.

### Configure AWS credentials with environment variables<a name="config-env-vars"></a>

Environment variables are variables maintained by the operating system and used by system commands\. They are not saved if you close the SSH session\. IDT for AWS IoT Greengrass can use the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables to store your AWS credentials\.

To set these variables on Linux, macOS, or Unix, use export:

```
export AWS_ACCESS_KEY_ID=<your_access_key_id>
export AWS_SECRET_ACCESS_KEY=<your_secret_access_key>
```

To set these variables on Windows, use set:

```
set AWS_ACCESS_KEY_ID=<your_access_key_id>
set AWS_SECRET_ACCESS_KEY=<your_secret_access_key>
```

To configure IDT to use the environment variables, edit the `auth` section in your `config.json` file\. Here is an example:

```
{
	"awsRegion": "us-west-2",
	"auth": {
		"method": "environment"
	}
}
```

## Configure device\.json<a name="device-config"></a>

In addition to AWS credentials, IDT for AWS IoT Greengrass needs information about the devices that tests are run on \(for example, IP address, login information, operating system, and CPU architecture\)\.

You must provide this information using the `device.json` template located in ` <device_tester_extract_location>/configs/device.json`:

------
#### [ Physical device ]

```
[
  {
    "id": "<pool-id>",
    "sku": "<sku>",
    "features": [
      {
        "name": "os",
        "value": "linux | ubuntu | openwrt"
      },
      {
        "name": "arch",
        "value": "x86_64 | armv6l | armv7l | aarch64"
      },
      {
        "name": "container",
        "value": "yes | no"
      },
      {
        "name": "docker",
        "value": "yes | no"
      },
      {
        "name": "streamManagement",
        "value": "yes | no"
      },
      {
        "name": "hsi",
        "value": "yes | no"
      },
      {
        "name": "ml",
        "value": "mxnet | tensorflow | dlr | mxnet,dlr,tensorflow | no"
      },
      *********** Remove the section below if the device is not qualifying for ML **************,
      {
        "name": "mlLambdaContainerizationMode",
        "value": "container | process | both"
      },
      {
        "name": "processor",
        "value": "cpu | gpu"
      },
      ******************************************************************************************
    ],
    *********** Remove the section below if the device is not qualifying for HSI ***************
    "hsm": {
      "p11Provider": "/path/to/pkcs11ProviderLibrary",
      "slotLabel": "<slot_label>",
      "slotUserPin": "<slot_pin>",
      "privateKeyLabel": "<key_label>",
      "openSSLEngine": "/path/to/openssl/engine"
    },
    ********************************************************************************************
    *********** Remove the section below if the device is not qualifying for ML ****************
    "machineLearning": {
      "dlrModelPath": "/path/to/compiled/dlr/model",
      "environmentVariables": [
        {
          "key": "<environment-variable-name>",
          "value": "<Path:$PATH>"
        }
      ],
      "deviceResources": [
        {
          "name": "<resource-name>",
          "path": "<resource-path>",
          "type": "device | volume"
        }
      ]
    },
    ******************************************************************************************
    "kernelConfigLocation": "",
    "greengrassLocation": "",
    "devices": [
      {
        "id": "<device-id>",
        "connectivity": {
          "protocol": "ssh",
          "ip": "<ip-address>",
          "port": 22,
          "auth": {
            "method": "pki | password",
            "credentials": {
              "user": "<user-name>",
              "privKeyPath": "/path/to/private/key",
              "password": "<password>"
            }
          }
        }
      }
    ]
  }
]
```

**Note**  
Specify `privKeyPath` only if `method` is set to `pki`\.  
Specify `password` only if `method` is set to `password`\.

------
#### [ Docker container ]

```
[
  {
    "id": "<pool-id>",
    "sku": "<sku>",
    "features": [
      {
        "name": "os",
        "value": "linux | ubuntu | openwrt"
      },
      {
        "name": "arch",
        "value": "x86_64"
      },
      {
        "name": "container",
        "value": "no"
      },
      {
        "name": "docker",
        "value": "no"
      },
      {
        "name": "streamManagement",
        "value": "yes | no"
      },
      {
        "name": "hsi",
        "value": "no"
      },
      {
        "name": "ml",
        "value": "mxnet | tensorflow | dlr | mxnet,dlr,tensorflow | no"
      },
      *********** Remove the section below if the device is not qualifying for ML **************,
      {
        "name": "mlLambdaContainerizationMode",
        "value": "process"
      },
      {
        "name": "processor",
        "value": "cpu | gpu"
      },
      ******************************************************************************************
    ],
    *********** Remove the section below if the device is not qualifying for ML ****************
    "machineLearning": {
      "dlrModelPath": "/path/to/compiled/dlr/model",
      "environmentVariables": [
        {
          "key": "<environment-variable-name>",
          "value": "<Path:$PATH>"
        }
      ],
      "deviceResources": [
        {
          "name": "<resource-name>",
          "path": "<resource-path>",
          "type": "device | volume"
        }
      ]
    },
    ******************************************************************************************
    "kernelConfigLocation": "",
    "greengrassLocation": "",
    "devices": [
      {
        "id": "<device-id>",
        "connectivity": {
          "protocol": "docker",
          "containerId": "<container-name | container-id>",
          "containerUser": "<user>"
        }
      }
    ]
  }
]
```

------

All fields that contain values are required as described here:

`id`  
A user\-defined alphanumeric ID that uniquely identifies a collection of devices called a *device pool*\. Devices that belong to a pool must have identical hardware\. When you run a suite of tests, devices in the pool are used to parallelize the workload\. Multiple devices are used to run different tests\.

`sku`  
An alphanumeric value that uniquely identifies the device under test\. The SKU is used to track qualified boards\.  
If you want to list your board in the AWS Partner Device Catalog, the SKU you specify here must match the SKU that you use in the listing process\.

`features`  
An array that contains the device's supported features\. All features are required\.    
`os` and `arch`  
Supported operating system \(OS\) and architecture combinations:  
+ `linux`, `x86_64`
+ `linux`, `armv6l`
+ `linux`, `armv7l`
+ `linux`, `aarch64`
+ `ubuntu`, `x86_64`
+ `openwrt`, `armv7l`
+ `openwrt`, `aarch64`
If you use IDT to test AWS IoT Greengrass running in a Docker container, only the x86\_64 Docker architecture is supported\.  
`container`  
<a name="description-container"></a>Validates that the device meets all of the software and hardware requirements to run Lambda functions in container mode on a Greengrass core\.  
The valid value is `yes` or `no`\.  
`docker`  
<a name="description-docker"></a>Validates that the device meets all the required technical dependencies to use the Greengrass Docker application deployment connector to run containers  
The valid value is `yes` or `no`\.  
`streamManagement`  
<a name="description-sm"></a>Validates that the device meets all of the required technical dependencies to run AWS IoT Greengrass stream manager\.  
The valid value is `yes` or `no`\.  
`hsi`  
<a name="description-hsi"></a>Verifies that the provided HSI shared library can interface with the hardware security module \(HSM\) and implements the required PKCS\#11 APIs correctly\. The HSM and shared library must be able to sign a CSR, perform TLS operations, and provide the correct key lengths and public key algorithm\.  
The valid value is `yes` or `no`\.  
`ml`  
<a name="description-ml"></a>Validates that the device meets all of the required technical dependencies to perform ML inference locally\.  
The valid value can be any combination of `mxnet`, `tensorflow`, `dlr`, and `no` \(for example, `mxnet`, `mxnet,tensorflow`, `mxnet,tensorflow,dlr`, or `no`\)\.  
`mlLambdaContainerizationMode`  
Validates that the device meets all of the required technical dependencies to perform ML inference in container mode on a Greengrass device\.  
The valid value is `container`, `process`, or `both`\.  
`processor`  
Validates that the device meets all of the hardware requirements for the specified processor type\.  
The valid value is `cpu` or `gpu`\.
If you don't want to use the `container`, `docker`, `streamManager`, `hsi`, or `ml` feature, you can set the corresponding `value` to `no`\.  
Docker only supports feature qualification for `streamManagement` and `ml`\.

`machineLearning`  
Optional\. Configuration information for ML qualification tests\. For more information, see [Configure device\.json for ML qualification](#device-json-ml-qualification)\.

`hsm`  
Optional\. Configuration information for testing with an AWS IoT Greengrass Hardware Security Module \(HSM\)\. Otherwise, the `hsm` property should be omitted\. For more information, see [Hardware security integration](hardware-security.md)\.  
<a name="connectivity-protocol-ssh-only"></a>This property applies only if `connectivity.protocol` is set to `ssh`\.    
`hsm.p11Provider`  
The absolute path to the PKCS\#11 implementation's libdl\-loadable library\.  
`hsm.slotLabel`  
The slot label used to identify the hardware module\.  
`hsm.slotUserPin`  
The user PIN used to authenticate the AWS IoT Greengrass core to the module\.  
`hsm.privateKeyLabel`  
The label used to identify the key in the hardware module\.  
`hsm.openSSLEngine`  
The absolute path to the OpenSSL engine's `.so` file that enables PKCS\#11 support on OpenSSL\. Used by the AWS IoT Greengrass OTA update agent\.

`devices.id`  
A user\-defined unique identifier for the device being tested\.

`connectivity.protocol`  
The communication protocol used to communicate with this device\. Currently, the only supported values are `ssh` for physical devices and `docker` for Docker containers\.

`connectivity.ip`  
The IP address of the device being tested\.  
<a name="connectivity-protocol-ssh-only"></a>This property applies only if `connectivity.protocol` is set to `ssh`\.

`connectivity.containerId`  
The container ID or name of the Docker container being tested\.  
<a name="connectivity-protocol-docker-only"></a>This property applies only if `connectivity.protocol` is set to `docker`\.

`connectivity.auth`  
Authentication information for the connection\.  
<a name="connectivity-protocol-ssh-only"></a>This property applies only if `connectivity.protocol` is set to `ssh`\.    
`connectivity.auth.method`  
The authentication method used to access a device over the given connectivity protocol\.  
Supported values are:  
+ `pki`
+ `password`  
`connectivity.auth.credentials`  
The credentials used for authentication\.    
`connectivity.auth.credentials.password`  
The password used for signing in to the device being tested\.  
This value applies only if `connectivity.auth.method` is set to `password`\.  
`connectivity.auth.credentials.privKeyPath`  
The full path to the private key used to sign in to the device under test\.  
This value applies only if `connectivity.auth.method` is set to `pki`\.  
`connectivity.auth.credentials.user`  
The user name for signing in to the device being tested\.  
`connectivity.auth.credentials.privKeyPath`  
The full path to the private key used to sign in to the device being tested\.

`connectivity.port`  
Optional\. The port number to use for SSH connections\.  
The default value is 22\.  
This property only applies if `connectivity.protocol` is set to `ssh`\.

`greengrassLocation`  
The location of AWS IoT Greengrass Core software on your devices\.  
For physical devices, this value is only used when you use an existing installation of AWS IoT Greengrass\. Use this attribute to tell IDT to use the version of the AWS IoT Greengrass Core software installed on your devices\.  
When running tests in a Docker container from Docker image or Dockerfile provided by AWS IoT Greengrass, set this value to `/greengrass`\.

`kernelConfigLocation`  
Optional\. The path to the kernel configuration file\. AWS IoT Device Tester uses this file to check if the devices have the required kernel features enabled\. If not specified, IDT uses the following paths to search for the kernel configuration file: `/proc/config.gz` and `/boot/config-<kernel-version>`\. AWS IoT Device Tester uses the first path it finds\.

## Configure device\.json for ML qualification<a name="device-json-ml-qualification"></a>

This section describes the optional properties in the device configuration file that apply to ML qualification\. If you plan to run tests for ML qualification, you must define the properties that apply to your use case\.

You can use the `device-ml.json` template to define the configuration settings for your device\. This template contains the optional ML properties\. You can also use `device.json` and add the ML qualification properties\. These files are located in `<device_tester_extract_location>/configs` and includes ML qualification properties\. If you use `device-ml.json`, you must rename the file to `device.json` before you run IDT tests\.

For information about device configuration properties that don't apply to ML qualification, see [Configure device\.json](#device-config)\.

 

`ml` in the `features` array  
The ML frameworks that your board supports\. <a name="idt-version-ml-qualification"></a>This property requires IDT v3\.1\.0 or later\.  
+ If your board supports only one framework, specify the framework\. For example:

  ```
  {
      "name": "ml",
      "value": "mxnet"
  }
  ```
+ If your board supports multiple frameworks, specify the frameworks as a comma\-separated list\. For example:

  ```
  {
      "name": "ml",
      "value": "mxnet,tensorflow"
  }
  ```

`mlLambdaContainerizationMode` in the `features` array  
The [containerization mode](lambda-group-config.md#lambda-containerization-considerations) that you want to test with\. <a name="idt-version-ml-qualification"></a>This property requires IDT v3\.1\.0 or later\.  
+ Choose `process` to run ML inference code with a non\-containerized Lambda function\. This option requires AWS IoT Greengrass v1\.10\.x or later\.
+ Choose `container` to run ML inference code with a containerized Lambda function\.
+ Choose `both` to run ML inference code with both modes\. This option requires AWS IoT Greengrass v1\.10\.x or later\.

`processor` in the `features` array  
Indicates the hardware accelerator that your board supports\. <a name="idt-version-ml-qualification"></a>This property requires IDT v3\.1\.0 or later\.  
+ Choose `cpu` if your board uses a CPU as the processor\.
+ Choose `gpu` if your board uses a GPU as the processor\.

`machineLearning`  
Optional\. Configuration information for ML qualification tests\. <a name="idt-version-ml-qualification"></a>This property requires IDT v3\.1\.0 or later\.    
`dlrModelPath`  
Required to use the `dlr` framework\. The absolute path to your DLR compiled model directory, which must be named `resnet18`\. For more information, see [Compile the DLR model](idt-ml-qualification.md#ml-qualification-dlr-compile-model)\.  
The following is an example path on macOS: `/Users/<user>/Downloads/resnet18`\.  
`environmentVariables`  
An array of key\-value pairs that can dynamically pass settings to ML inference tests\. Optional for CPU devices\. You can use this section to add framework\-specific environment variables required by your device type\. For information about these requirements, see the official website of the framework or the device\. For example, to run MXNet inference tests on some devices, the following environment variables might be required\.  

```
"environmentVariables": [
    ...
    {
        "key": "PYTHONPATH",      
        "value": "$MXNET_HOME/python:$PYTHONPATH"    
    },
    {
        "key": "MXNET_HOME",
        "value": "$HOME/mxnet/"
    },
    ...
]
```
The `value` field might vary based on your MXNet installation\.
If you're testing Lambda functions that run with [containerization](lambda-group-config.md#lambda-containerization-considerations) on GPU devices, add environment variables for the GPU library\. This makes it possible for the GPU to perform computations\. To use different GPU libraries, see the official documentation for the library or device\.  
Configure the following keys if the `mlLambdaContainerizationMode` feature is set to `container` or `both`\.

```
"environmentVariables": [
    {
        "key": "PATH",      
        "value": "<path/to/software/bin>:$PATH"    
    },
    {
        "key": "LD_LIBRARY_PATH",      
        "value": "<path/to/ld/lib>"    
    },
    ...
]
```  
`deviceResources`  
Required by GPU devices\. Contains [local resources](access-local-resources.md#lra-resource-types) that can be accessed by Lambda functions\. Use this section to add local device and volume resources\.  
+ For device resources, specify `"type": "device"`\. For GPU devices, device resources should be GPU\-related device files under `/dev`\.
**Note**  
The `/dev/shm` directory is an exception\. It can be configured as a volume resource only\.
+ For volume resources, specify `"type": "volume"`\.