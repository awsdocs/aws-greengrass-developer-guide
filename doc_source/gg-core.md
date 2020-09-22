# Configure the AWS IoT Greengrass core<a name="gg-core"></a>

An AWS IoT Greengrass core is an AWS IoT thing \(device\) that acts as a hub or gateway in edge environments\. Like other AWS IoT devices, a core exists in the registry, has a device shadow, and uses a device certificate to authenticate with AWS IoT Core and AWS IoT Greengrass\. The core device runs the AWS IoT Greengrass Core software, which enables it to manage local processes for Greengrass groups, such as communication, shadow sync, and token exchange\.

The AWS IoT Greengrass Core software provides the following functionality:<a name="ggc-software-features"></a>
+ Deployment and local execution of connectors and Lambda functions\.
+ Process data streams locally with automatic exports to the AWS Cloud\.
+ MQTT messaging over the local network between devices, connectors, and Lambda functions using managed subscriptions\.
+ MQTT messaging between AWS IoT and devices, connectors, and Lambda functions using managed subscriptions\.
+ Secure connections between devices and the AWS Cloud using device authentication and authorization\.
+ Local shadow synchronization of devices\. Shadows can be configured to sync with the AWS Cloud\.
+ Controlled access to local device and volume resources\.
+ Deployment of cloud\-trained machine learning models for running local inference\.
+ Automatic IP address detection that enables devices to discover the Greengrass core device\.
+ Central deployment of new or updated group configuration\. After the configuration data is downloaded, the core device is restarted automatically\.
+ Secure, over\-the\-air \(OTA\) software updates of user\-defined Lambda functions\.
+ Secure, encrypted storage of local secrets and controlled access by connectors and Lambda functions\.

## AWS IoT Greengrass core configuration file<a name="config-json"></a>

The configuration file for the AWS IoT Greengrass Core software is `config.json`\. It is located in the `/greengrass-root/config` directory\.

**Note**  
*greengrass\-root* represents the path where the AWS IoT Greengrass Core software is installed on your device\. Typically, this is the `/greengrass` directory\.  
If you use the **Default Group creation** option from the AWS IoT Greengrass console, then the `config.json` file is deployed to the core device in a working state\.

 You can review the contents of this file by running the following command:

```
cat /greengrass-root/config/config.json
```

The following is an example `config.json` file\. This is the version that's generated when you create the core from the AWS IoT Greengrass console\.

------
#### [ GGC v1\.11 ]

```
{
  "coreThing" : {
    "caPath" : "root.ca.pem",
    "certPath" : "hash.cert.pem",
    "keyPath" : "hash.private.key",
    "thingArn" : "arn:partition:iot:region:account-id:thing/core-thing-name",
    "iotHost" : "host-prefix-ats.iot.region.amazonaws.com",
    "ggHost" : "greengrass-ats.iot.region.amazonaws.com",
    "keepAlive" : 600,   
    "ggDaemonPort": 8000,
    "systemComponentAuthTimeout": 5000
  },
  "runtime" : {
    "maxWorkItemCount" : 1024,
    "cgroup" : {
      "useSystemd" : "yes"
    }
  },
  "managedRespawn" : false,
  "crypto" : {
    "principals" : {
      "SecretsManager" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key"
      },
      "IoTCertificate" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key",
        "certificatePath" : "file:///greengrass/certs/hash.cert.pem"
      } 
    },
    "caPath" : "file:///greengrass/certs/root.ca.pem"
  }
}
```

The `config.json` file supports the following properties:

**coreThing**


| Field | Description | Notes | 
| --- | --- | --- | 
| <a name="shared-config-capath"></a>caPath |  The path to the AWS IoT root CA relative to the `/greengrass-root/certs` directory\.  |  For backward compatibility with versions earlier than 1\.7\.0\. This property is ignored when the `crypto` object is present\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\.   | 
| <a name="shared-config-certpath"></a>certPath |  The path to the core device certificate relative to the `/greengrass-root/certs` directory\.  | For backward compatibility with versions earlier than 1\.7\.0\. This property is ignored when the crypto object is present\. | 
| <a name="shared-config-keypath"></a>keyPath | The path to the core private key relative to /greengrass\-root/certs directory\. | For backward compatibility with versions earlier than 1\.7\.0\. This property is ignored when the crypto object is present\. | 
| <a name="shared-config-thingarn"></a>thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS IoT Greengrass core device\. | Find this for your core in the AWS IoT Greengrass console under Cores, or by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-core-definition-version.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-core-definition-version.html) CLI command\. | 
| <a name="shared-config-iothost-v1.9"></a>iotHost | Your AWS IoT endpoint\. |  Find this in the AWS IoT console under **Settings**, or by running the [https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) CLI command\. This command returns the Amazon Trust Services \(ATS\) endpoint\. For more information, see the [Server authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) documentation\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\. Make sure that your [ endpoints correspond to your AWS Region](https://docs.aws.amazon.com/general/latest/gr/greengrass.html)\.    | 
| <a name="shared-config-gghost-v1.9"></a>ggHost | Your AWS IoT Greengrass endpoint\. |  This is your `iotHost` endpoint with the host prefix replaced by *greengrass* \(for example, `greengrass-ats.iot.region.amazonaws.com`\)\. Use the same AWS Region as `iotHost`\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\. Make sure that your [ endpoints correspond to your AWS Region](https://docs.aws.amazon.com/general/latest/gr/greengrass.html)\.    | 
| <a name="shared-config-iotmqttport"></a>iotMqttPort | Optional\. The port number to use for MQTT communication with AWS IoT\. | Valid values are 8883 or 443\. The default value is 8883\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="shared-config-iothttpport"></a>iotHttpPort | Optional\. The port number used to create HTTPS connections to AWS IoT\. | Valid values are 8443 or 443\. The default value is 8443\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="shared-config-ggmqttport"></a>ggMqttPort | Optional\. The port number to use for MQTT communication over the local network\. | Valid values are 1024 through 65535\. The default value is 8883\. For more information, see [Configure the MQTT port for local messaging](#config-local-mqtt-port)\. | 
| <a name="shared-config-gghttpport"></a>ggHttpPort | Optional\. The port number used to create HTTPS connections to the AWS IoT Greengrass service\. | Valid values are 8443 or 443\. The default value is 8443\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="shared-config-keepalive"></a>keepAlive | Optional\. The MQTT KeepAlive period, in seconds\. | Valid range is between 30 and 1200 seconds\. The default value is 600\. | 
| <a name="shared-config-networkproxy"></a>networkProxy | Optional\. An object that defines a proxy server to connect to\. | This can be an HTTP or HTTPS proxy\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="config-mqttOperationTimeout-v1.11.0"></a>mqttOperationTimeout | Optional\. The amount of time \(in seconds\) to allow the Greengrass core to complete a publish, subscribe, or unsubscribe operation in MQTT connections to AWS IoT Core\. | The default value is 5\. The minimum value is 5\. | 
| <a name="shared-conifg-ggDaemonPort"></a>ggDaemonPort | Optional\. The Greengrass core IPC port number\. |  This property is available in AWS IoT Greengrass v1\.11\.0 or later\. Valid values are between 1024 and 65535\. The default value is 8000\.  | 
| <a name="shared-config-systemComponentAuthTimeout"></a>systemComponentAuthTimeout | Optional\. The time \(in milliseconds\) to allow the Greengrass core IPC to complete authentication\. |  This property is available in AWS IoT Greengrass v1\.11\.0 or later\. Valid values are between 500 and 5000\. The default value is 5000\.  | 

**runtime**


| Field | Description | Notes | 
| --- |--- |--- |
| maxWorkItemCount | Optional\. The maximum number of work items that the Greengrass daemon can process at a time\. Work items that exceed this limit are ignored\. The work item queue is shared by system components, user\-defined Lambda functions, and connectors\. | The default value is 1024\. The maximum value is limited by your device hardware\. Increasing this value increases the memory that AWS IoT Greengrass uses\. You can increase this value if you expect your core to receive heavy MQTT message traffic\. | 
| postStartHealthCheckTimeout | Optional\. The time \(in milliseconds\) after starting that the Greengrass daemon waits for the health check to finish\. | The default timeout is 30 seconds \(30000 ms\)\. | 
| `cgroup` | 
| --- |
| useSystemd | Indicates whether your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Valid values are yes or no\. Run the check\_ggc\_dependencies script in [Module 1](module1.md) to see if your device uses systemd\. | 

**crypto**

The `crypto` contains properties that support private key storage on a hardware security module \(HSM\) through PKCS\#11 and local secret storage\. For more information, see [AWS IoT Greengrass core security principals](gg-sec.md#gg-principals), [Hardware security integration](hardware-security.md), and [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\. Configurations for private key storage on HSMs or in the file system are supported\.


| Field | Description | Notes | 
| --- |--- |--- |
| caPath |  The absolute path to the AWS IoT root CA\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\.   | 
| `PKCS11` | 
| --- |
| OpenSSLEngine |  Optional\. The absolute path to the OpenSSL engine `.so` file to enable PKCS\#11 support on OpenSSL\.  |  Must be a path to a file on the file system\. This property is required if you're using the Greengrass OTA update agent with hardware security\. For more information, see [Configure support for over\-the\-air updates](hardware-security.md#hardware-security-ota-updates)\.  | 
| P11Provider |  The absolute path to the PKCS\#11 implementation's libdl\-loadable library\.  |  Must be a path to a file on the file system\.  | 
| slotLabel |  The slot label that's used to identify the hardware module\.  |  Must conform to PKCS\#11 label specifications\.  | 
| slotUserPin |  The user pin that's used to authenticate the Greengrass core to the module\.  |  Must have sufficient permissions to perform C\_Sign with the configured private keys\.  | 
| `principals` | 
| --- |
| IoTCertificate | The certificate and private key that the core uses to make requests to AWS IoT\. | 
| IoTCertificate  \.privateKeyPath  |  The path to the core private key\.  |  For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\.  | 
| IoTCertificate  \.certificatePath |  The absolute path to the core device certificate\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\.  | 
| MQTTServerCertificate |  Optional\. The private key that the core uses in combination with the certificate to act as an MQTT server or gateway\.  | 
| MQTTServerCertificate  \.privateKeyPath |  The path to the local MQTT server private key\.  |  Use this value to specify your own private key for the local MQTT server\. For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. If this property is omitted, AWS IoT Greengrass rotates the key based your rotation settings\. If specified, the customer is responsible for rotating the key\.  | 
| SecretsManager | The private key that secures the data key used for encryption\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\. | 
| SecretsManager  \.privateKeyPath |  The path to the local secrets manager private key\.  |  Only an RSA key is supported\. For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. The private key must be generated using the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism\.  | 

The following configuration properties are also supported:


****  

| Field | Description | Notes | 
| --- | --- | --- | 
| <a name="shared-config-mqttmaxconnectionretryinterval"></a> mqttMaxConnectionRetryInterval  |  Optional\. The maximum interval \(in seconds\) between MQTT connection retries if the connection is dropped\.  |  Specify this value as an unsigned integer\. The default is `60`\.  | 
| <a name="shared-config-managedrespawn"></a> managedRespawn  |  Optional\. Indicates that the OTA agent needs to run custom code before an update\.  |  Valid values are `true` or `false`\. For more information, see [OTA updates of AWS IoT Greengrass Core software](core-ota-update.md)\.  | 
| <a name="shared-config-writedirectory"></a> writeDirectory  |  Optional\. The write directory where AWS IoT Greengrass creates all read\-write resources\.  |  For more information, see [Configure a write directory for AWS IoT Greengrass](#write-directory)\.  | 

------
#### [ GGC v1\.10 ]

```
{
  "coreThing" : {
    "caPath" : "root.ca.pem",
    "certPath" : "hash.cert.pem",
    "keyPath" : "hash.private.key",
    "thingArn" : "arn:partition:iot:region:account-id:thing/core-thing-name",
    "iotHost" : "host-prefix-ats.iot.region.amazonaws.com",
    "ggHost" : "greengrass-ats.iot.region.amazonaws.com",
    "keepAlive" : 600
  },
  "runtime" : {
    "maxWorkItemCount" : 1024,
    "cgroup" : {
      "useSystemd" : "yes"
    }
  },
  "managedRespawn" : false,
  "crypto" : {
    "principals" : {
      "SecretsManager" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key"
      },
      "IoTCertificate" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key",
        "certificatePath" : "file:///greengrass/certs/hash.cert.pem"
      } 
    },
    "caPath" : "file:///greengrass/certs/root.ca.pem"
  }
}
```

The `config.json` file supports the following properties:

**coreThing**


| Field | Description | Notes | 
| --- | --- | --- | 
| <a name="shared-config-capath"></a>caPath |  The path to the AWS IoT root CA relative to the `/greengrass-root/certs` directory\.  |  For backward compatibility with versions earlier than 1\.7\.0\. This property is ignored when the `crypto` object is present\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\.   | 
| <a name="shared-config-certpath"></a>certPath |  The path to the core device certificate relative to the `/greengrass-root/certs` directory\.  | For backward compatibility with versions earlier than 1\.7\.0\. This property is ignored when the crypto object is present\. | 
| <a name="shared-config-keypath"></a>keyPath | The path to the core private key relative to /greengrass\-root/certs directory\. | For backward compatibility with versions earlier than 1\.7\.0\. This property is ignored when the crypto object is present\. | 
| <a name="shared-config-thingarn"></a>thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS IoT Greengrass core device\. | Find this for your core in the AWS IoT Greengrass console under Cores, or by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-core-definition-version.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-core-definition-version.html) CLI command\. | 
| <a name="shared-config-iothost-v1.9"></a>iotHost | Your AWS IoT endpoint\. |  Find this in the AWS IoT console under **Settings**, or by running the [https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) CLI command\. This command returns the Amazon Trust Services \(ATS\) endpoint\. For more information, see the [Server authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) documentation\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\. Make sure that your [ endpoints correspond to your AWS Region](https://docs.aws.amazon.com/general/latest/gr/greengrass.html)\.    | 
| <a name="shared-config-gghost-v1.9"></a>ggHost | Your AWS IoT Greengrass endpoint\. |  This is your `iotHost` endpoint with the host prefix replaced by *greengrass* \(for example, `greengrass-ats.iot.region.amazonaws.com`\)\. Use the same AWS Region as `iotHost`\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\. Make sure that your [ endpoints correspond to your AWS Region](https://docs.aws.amazon.com/general/latest/gr/greengrass.html)\.    | 
| <a name="shared-config-iotmqttport"></a>iotMqttPort | Optional\. The port number to use for MQTT communication with AWS IoT\. | Valid values are 8883 or 443\. The default value is 8883\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="shared-config-iothttpport"></a>iotHttpPort | Optional\. The port number used to create HTTPS connections to AWS IoT\. | Valid values are 8443 or 443\. The default value is 8443\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="shared-config-ggmqttport"></a>ggMqttPort | Optional\. The port number to use for MQTT communication over the local network\. | Valid values are 1024 through 65535\. The default value is 8883\. For more information, see [Configure the MQTT port for local messaging](#config-local-mqtt-port)\. | 
| <a name="shared-config-gghttpport"></a>ggHttpPort | Optional\. The port number used to create HTTPS connections to the AWS IoT Greengrass service\. | Valid values are 8443 or 443\. The default value is 8443\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="shared-config-keepalive"></a>keepAlive | Optional\. The MQTT KeepAlive period, in seconds\. | Valid range is between 30 and 1200 seconds\. The default value is 600\. | 
| <a name="shared-config-networkproxy"></a>networkProxy | Optional\. An object that defines a proxy server to connect to\. | This can be an HTTP or HTTPS proxy\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="config-mqttOperationTimeout-v1.10.2"></a>mqttOperationTimeout | Optional\. The amount of time \(in seconds\) to allow the Greengrass core to complete a publish, subscribe, or unsubscribe operation in MQTT connections to AWS IoT Core\. |  This property is available starting in AWS IoT Greengrass v1\.10\.2\. The default value is 5\. The minimum value is 5\.  | 

**runtime**


| Field | Description | Notes | 
| --- |--- |--- |
| maxWorkItemCount | Optional\. The maximum number of work items that the Greengrass daemon can process at a time\. Work items that exceed this limit are ignored\. The work item queue is shared by system components, user\-defined Lambda functions, and connectors\. | The default value is 1024\. The maximum value is limited by your device hardware\. Increasing this value increases the memory that AWS IoT Greengrass uses\. You can increase this value if you expect your core to receive heavy MQTT message traffic\. | 
| postStartHealthCheckTimeout | Optional\. The time \(in milliseconds\) after starting that the Greengrass daemon waits for the health check to finish\. | The default timeout is 30 seconds \(30000 ms\)\. | 
| `cgroup` | 
| --- |
| useSystemd | Indicates whether your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Valid values are yes or no\. Run the check\_ggc\_dependencies script in [Module 1](module1.md) to see if your device uses systemd\. | 

**crypto**

The `crypto` contains properties that support private key storage on a hardware security module \(HSM\) through PKCS\#11 and local secret storage\. For more information, see [AWS IoT Greengrass core security principals](gg-sec.md#gg-principals), [Hardware security integration](hardware-security.md), and [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\. Configurations for private key storage on HSMs or in the file system are supported\.


| Field | Description | Notes | 
| --- |--- |--- |
| caPath |  The absolute path to the AWS IoT root CA\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\.   | 
| `PKCS11` | 
| --- |
| OpenSSLEngine |  Optional\. The absolute path to the OpenSSL engine `.so` file to enable PKCS\#11 support on OpenSSL\.  |  Must be a path to a file on the file system\. This property is required if you're using the Greengrass OTA update agent with hardware security\. For more information, see [Configure support for over\-the\-air updates](hardware-security.md#hardware-security-ota-updates)\.  | 
| P11Provider |  The absolute path to the PKCS\#11 implementation's libdl\-loadable library\.  |  Must be a path to a file on the file system\.  | 
| slotLabel |  The slot label that's used to identify the hardware module\.  |  Must conform to PKCS\#11 label specifications\.  | 
| slotUserPin |  The user pin that's used to authenticate the Greengrass core to the module\.  |  Must have sufficient permissions to perform C\_Sign with the configured private keys\.  | 
| `principals` | 
| --- |
| IoTCertificate | The certificate and private key that the core uses to make requests to AWS IoT\. | 
| IoTCertificate  \.privateKeyPath  |  The path to the core private key\.  |  For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\.  | 
| IoTCertificate  \.certificatePath |  The absolute path to the core device certificate\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\.  | 
| MQTTServerCertificate |  Optional\. The private key that the core uses in combination with the certificate to act as an MQTT server or gateway\.  | 
| MQTTServerCertificate  \.privateKeyPath |  The path to the local MQTT server private key\.  |  Use this value to specify your own private key for the local MQTT server\. For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. If this property is omitted, AWS IoT Greengrass rotates the key based your rotation settings\. If specified, the customer is responsible for rotating the key\.  | 
| SecretsManager | The private key that secures the data key used for encryption\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\. | 
| SecretsManager  \.privateKeyPath |  The path to the local secrets manager private key\.  |  Only an RSA key is supported\. For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. The private key must be generated using the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism\.  | 

The following configuration properties are also supported:


****  

| Field | Description | Notes | 
| --- | --- | --- | 
| <a name="shared-config-mqttmaxconnectionretryinterval"></a> mqttMaxConnectionRetryInterval  |  Optional\. The maximum interval \(in seconds\) between MQTT connection retries if the connection is dropped\.  |  Specify this value as an unsigned integer\. The default is `60`\.  | 
| <a name="shared-config-managedrespawn"></a> managedRespawn  |  Optional\. Indicates that the OTA agent needs to run custom code before an update\.  |  Valid values are `true` or `false`\. For more information, see [OTA updates of AWS IoT Greengrass Core software](core-ota-update.md)\.  | 
| <a name="shared-config-writedirectory"></a> writeDirectory  |  Optional\. The write directory where AWS IoT Greengrass creates all read\-write resources\.  |  For more information, see [Configure a write directory for AWS IoT Greengrass](#write-directory)\.  | 

------
#### [ GGC v1\.9 ]

```
{
  "coreThing" : {
    "caPath" : "root.ca.pem",
    "certPath" : "hash.cert.pem",
    "keyPath" : "hash.private.key",
    "thingArn" : "arn:partition:iot:region:account-id:thing/core-thing-name",
    "iotHost" : "host-prefix-ats.iot.region.amazonaws.com",
    "ggHost" : "greengrass-ats.iot.region.amazonaws.com",
    "keepAlive" : 600
  },
  "runtime" : {
    "cgroup" : {
      "useSystemd" : "yes"
    }
  },
  "managedRespawn" : false,
  "crypto" : {
    "principals" : {
      "SecretsManager" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key"
      },
      "IoTCertificate" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key",
        "certificatePath" : "file:///greengrass/certs/hash.cert.pem"
      } 
    },
    "caPath" : "file:///greengrass/certs/root.ca.pem"
  }
}
```

The `config.json` file supports the following properties:

**coreThing**


| Field | Description | Notes | 
| --- | --- | --- | 
| <a name="shared-config-capath"></a>caPath |  The path to the AWS IoT root CA relative to the `/greengrass-root/certs` directory\.  |  For backward compatibility with versions earlier than 1\.7\.0\. This property is ignored when the `crypto` object is present\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\.   | 
| <a name="shared-config-certpath"></a>certPath |  The path to the core device certificate relative to the `/greengrass-root/certs` directory\.  | For backward compatibility with versions earlier than 1\.7\.0\. This property is ignored when the crypto object is present\. | 
| <a name="shared-config-keypath"></a>keyPath | The path to the core private key relative to /greengrass\-root/certs directory\. | For backward compatibility with versions earlier than 1\.7\.0\. This property is ignored when the crypto object is present\. | 
| <a name="shared-config-thingarn"></a>thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS IoT Greengrass core device\. | Find this for your core in the AWS IoT Greengrass console under Cores, or by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-core-definition-version.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-core-definition-version.html) CLI command\. | 
| <a name="shared-config-iothost-v1.9"></a>iotHost | Your AWS IoT endpoint\. |  Find this in the AWS IoT console under **Settings**, or by running the [https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) CLI command\. This command returns the Amazon Trust Services \(ATS\) endpoint\. For more information, see the [Server authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) documentation\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\. Make sure that your [ endpoints correspond to your AWS Region](https://docs.aws.amazon.com/general/latest/gr/greengrass.html)\.    | 
| <a name="shared-config-gghost-v1.9"></a>ggHost | Your AWS IoT Greengrass endpoint\. |  This is your `iotHost` endpoint with the host prefix replaced by *greengrass* \(for example, `greengrass-ats.iot.region.amazonaws.com`\)\. Use the same AWS Region as `iotHost`\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\. Make sure that your [ endpoints correspond to your AWS Region](https://docs.aws.amazon.com/general/latest/gr/greengrass.html)\.    | 
| <a name="shared-config-iotmqttport"></a>iotMqttPort | Optional\. The port number to use for MQTT communication with AWS IoT\. | Valid values are 8883 or 443\. The default value is 8883\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="shared-config-iothttpport"></a>iotHttpPort | Optional\. The port number used to create HTTPS connections to AWS IoT\. | Valid values are 8443 or 443\. The default value is 8443\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="shared-config-gghttpport"></a>ggHttpPort | Optional\. The port number used to create HTTPS connections to the AWS IoT Greengrass service\. | Valid values are 8443 or 443\. The default value is 8443\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 
| <a name="shared-config-keepalive"></a>keepAlive | Optional\. The MQTT KeepAlive period, in seconds\. | Valid range is between 30 and 1200 seconds\. The default value is 600\. | 
| <a name="shared-config-networkproxy"></a>networkProxy | Optional\. An object that defines a proxy server to connect to\. | This can be an HTTP or HTTPS proxy\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\. | 

**runtime**


| Field | Description | Notes | 
| --- |--- |--- |
| postStartHealthCheckTimeout | Optional\. The time \(in milliseconds\) after starting that the Greengrass daemon waits for the health check to finish\. | The default timeout is 30 seconds \(30000 ms\)\. | 
| `cgroup` | 
| --- |
| useSystemd | Indicates whether your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Valid values are yes or no\. Run the check\_ggc\_dependencies script in [Module 1](module1.md) to see if your device uses systemd\. | 

**crypto**

The `crypto` object is added in v1\.7\.0\. It introduces properties that support private key storage on a hardware security module \(HSM\) through PKCS\#11 and local secret storage\. For more information, see [AWS IoT Greengrass core security principals](gg-sec.md#gg-principals), [Hardware security integration](hardware-security.md), and [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\. Configurations for private key storage on HSMs or in the file system are supported\.


| Field | Description | Notes | 
| --- |--- |--- |
| caPath |  The absolute path to the AWS IoT root CA\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\.  Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\.   | 
| `PKCS11` | 
| --- |
| OpenSSLEngine |  Optional\. The absolute path to the OpenSSL engine `.so` file to enable PKCS\#11 support on OpenSSL\.  |  Must be a path to a file on the file system\. This property is required if you're using the Greengrass OTA update agent with hardware security\. For more information, see [Configure support for over\-the\-air updates](hardware-security.md#hardware-security-ota-updates)\.  | 
| P11Provider |  The absolute path to the PKCS\#11 implementation's libdl\-loadable library\.  |  Must be a path to a file on the file system\.  | 
| slotLabel |  The slot label that's used to identify the hardware module\.  |  Must conform to PKCS\#11 label specifications\.  | 
| slotUserPin |  The user pin that's used to authenticate the Greengrass core to the module\.  |  Must have sufficient permissions to perform C\_Sign with the configured private keys\.  | 
| `principals` | 
| --- |
| IoTCertificate | The certificate and private key that the core uses to make requests to AWS IoT\. | 
| IoTCertificate  \.privateKeyPath  |  The path to the core private key\.  |  For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\.  | 
| IoTCertificate  \.certificatePath |  The absolute path to the core device certificate\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\.  | 
| MQTTServerCertificate |  Optional\. The private key that the core uses in combination with the certificate to act as an MQTT server or gateway\.  | 
| MQTTServerCertificate  \.privateKeyPath |  The path to the local MQTT server private key\.  |  Use this value to specify your own private key for the local MQTT server\. For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. If this property is omitted, AWS IoT Greengrass rotates the key based your rotation settings\. If specified, the customer is responsible for rotating the key\.  | 
| SecretsManager | The private key that secures the data key used for encryption\. For more information, see [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\. | 
| SecretsManager  \.privateKeyPath |  The path to the local secrets manager private key\.  |  Only an RSA key is supported\. For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. The private key must be generated using the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism\.  | 

The following configuration properties are also supported:


****  

| Field | Description | Notes | 
| --- | --- | --- | 
| <a name="shared-config-mqttmaxconnectionretryinterval"></a> mqttMaxConnectionRetryInterval  |  Optional\. The maximum interval \(in seconds\) between MQTT connection retries if the connection is dropped\.  |  Specify this value as an unsigned integer\. The default is `60`\.  | 
| <a name="shared-config-managedrespawn"></a> managedRespawn  |  Optional\. Indicates that the OTA agent needs to run custom code before an update\.  |  Valid values are `true` or `false`\. For more information, see [OTA updates of AWS IoT Greengrass Core software](core-ota-update.md)\.  | 
| <a name="shared-config-writedirectory"></a> writeDirectory  |  Optional\. The write directory where AWS IoT Greengrass creates all read\-write resources\.  |  For more information, see [Configure a write directory for AWS IoT Greengrass](#write-directory)\.  | 

------
#### [ Deprecated versions ]

The following versions of the AWS IoT Greengrass Core software are not supported\. This information is included for reference purposes only\.

**GGC v1\.8**  

```
{
  "coreThing" : {
    "caPath" : "root.ca.pem",
    "certPath" : "hash.cert.pem",
    "keyPath" : "hash.private.key",
    "thingArn" : "arn:aws:iot:region:account-id:thing/core-thing-name",
    "iotHost" : "host-prefix-ats.iot.region.amazonaws.com",
    "ggHost" : "greengrass-ats.iot.region.amazonaws.com",
    "keepAlive" : 600
  },
  "runtime" : {
    "cgroup" : {
      "useSystemd" : "yes"
    }
  },
  "managedRespawn" : false,
  "crypto" : {
    "principals" : {
      "SecretsManager" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key"
      },
      "IoTCertificate" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key",
        "certificatePath" : "file:///greengrass/certs/hash.cert.pem"
      } 
    },
    "caPath" : "file:///greengrass/certs/root.ca.pem"
  }
}
```
The `config.json` file supports the following properties:  
**coreThing**      
<a name="config-json-properties-corething-v1.8"></a>[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)
**runtime**      
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)
**crypto**  
The `crypto` object is added in v1\.7\.0\. It introduces properties that support private key storage on a hardware security module \(HSM\) through PKCS\#11 and local secret storage\. For more information, see [AWS IoT Greengrass core security principals](gg-sec.md#gg-principals), [Hardware security integration](hardware-security.md), and [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\. Configurations for private key storage on HSMs or in the file system are supported\.      
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)
The following configuration properties are also supported:    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)

**GGC v1\.7**  

```
{
  "coreThing" : {
    "caPath" : "root.ca.pem",
    "certPath" : "hash.cert.pem",
    "keyPath" : "hash.private.key",
    "thingArn" : "arn:aws:iot:region:account-id:thing/core-thing-name",
    "iotHost" : "host-prefix-ats.iot.region.amazonaws.com",
    "ggHost" : "greengrass-ats.iot.region.amazonaws.com",
    "keepAlive" : 600
  },
  "runtime" : {
    "cgroup" : {
      "useSystemd" : "yes"
    }
  },
  "managedRespawn" : false,
  "crypto" : {
    "principals" : {
      "SecretsManager" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key"
      },
      "IoTCertificate" : {
        "privateKeyPath" : "file:///greengrass/certs/hash.private.key",
        "certificatePath" : "file:///greengrass/certs/hash.cert.pem"
      } 
    },
    "caPath" : "file:///greengrass/certs/root.ca.pem"
  }
}
```
The `config.json` file supports the following properties:  
**coreThing**      
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)
**runtime**      
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)
**crypto**  
The `crypto` object, added in v1\.7\.0, introduces properties that support private key storage on a hardware security module \(HSM\) through PKCS\#11 and local secret storage\. For more information, see [Hardware security integration](hardware-security.md) and [Deploy secrets to the AWS IoT Greengrass core](secrets.md)\. Configurations for private key storage on HSMs or in the file system are supported\.      
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)
The following configuration properties are also supported:    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)

**GGC v1\.6**  

```
{
   "coreThing": {
       "caPath": "root-ca-pem",
       "certPath": "cloud-pem-crt",
       "keyPath": "cloud-pem-key",
       "thingArn": "arn:aws:iot:region:account-id:thing/core-thing-name",
       "iotHost": "host-prefix.iot.region.amazonaws.com",
       "ggHost": "greengrass.iot.region.amazonaws.com",
       "keepAlive": 600,
       "mqttMaxConnectionRetryInterval": 60
   },
   "runtime": {
       "cgroup": {
           "useSystemd": "yes|no"
       }
   },
   "managedRespawn": true,
   "writeDirectory": "/write-directory"
}
```
If you use the **Default Group creation** option from the AWS IoT Greengrass console, then the `config.json` file is deployed to the core device in a working state that specifies the default configuration\.
The `config.json` file supports the following properties:    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)

**GGC v1\.5**  

```
{
   "coreThing": {
       "caPath": "root-ca-pem",
       "certPath": "cloud-pem-crt",
       "keyPath": "cloud-pem-key",
       "thingArn": "arn:aws:iot:region:account-id:thing/core-thing-name",
       "iotHost": "host-prefix.iot.region.amazonaws.com",
       "ggHost": "greengrass.iot.region.amazonaws.com",
       "keepAlive": 600
   },
   "runtime": {
       "cgroup": {
           "useSystemd": "yes|no"
       }
   },
   "managedRespawn": true
}
```
The `config.json` file exists in `/greengrass-root/config` and contains the following parameters:    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)

**GGC v1\.3**  

```
{
   "coreThing": {
       "caPath": "root-ca-pem",
       "certPath": "cloud-pem-crt",
       "keyPath": "cloud-pem-key",
       "thingArn": "arn:aws:iot:region:account-id:thing/core-thing-name",
       "iotHost": "host-prefix.iot.region.amazonaws.com",
       "ggHost": "greengrass.iot.region.amazonaws.com",
       "keepAlive": 600
   },
   "runtime": {
       "cgroup": {
           "useSystemd": "yes|no"
       }
   },
   "managedRespawn": true
}
```
The `config.json` file exists in `/greengrass-root/config` and contains the following parameters:    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)

**GGC v1\.1**  

```
{
   "coreThing": {
       "caPath": "root-ca-pem",
       "certPath": "cloud-pem-crt",
       "keyPath": "cloud-pem-key",
       "thingArn": "arn:aws:iot:region:account-id:thing/core-thing-name",
       "iotHost": "host-prefix.iot.region.amazonaws.com",
       "ggHost": "greengrass.iot.region.amazonaws.com",
       "keepAlive": 600
   },
   "runtime": {
       "cgroup": {
           "useSystemd": "yes|no"
       }
   }
}
```
The `config.json` file exists in `/greengrass-root/config` and contains the following parameters:    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)

**GGC v1\.0**  
In AWS IoT Greengrass Core v1\.0, `config.json` is deployed to `greengrass-root/configuration`\.  

```
{
   "coreThing": {
       "caPath": "root-ca-pem",
       "certPath": "cloud-pem-crt",
       "keyPath": "cloud-pem-key",
       "thingArn": "arn:aws:iot:region:account-id:thing/core-thing-name",
       "iotHost": "host-prefix.iot.region.amazonaws.com",
       "ggHost": "greengrass.iot.region.amazonaws.com",
       "keepAlive": 600
   },
   "runtime": {
       "cgroup": {
           "useSystemd": "yes|no"
       }
   }
}
```
The `config.json` file exists in `/greengrass-root/configuration` and contains the following parameters:    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)

------

## Service endpoints must match the root CA certificate type<a name="certificate-endpoints"></a>

Your AWS IoT Core and AWS IoT Greengrass endpoints must correspond to the certificate type of the root CA certificate on your device\. If the endpoints and certificate type do not match, authentication attempts fail between the device and AWS IoT Core or AWS IoT Greengrass\. For more information, see [Server authentication](https://docs.aws.amazon.com/iot/latest/developerguide/server-authentication.html) in the *AWS IoT Developer Guide*\.

If your device uses an Amazon Trust Services \(ATS\) root CA certificate, which is the preferred method, it must also use ATS endpoints for device management and discovery data plane operations\. ATS endpoints include the `ats` segment, as shown in the following syntax for the AWS IoT Core endpoint\.

```
prefix-ats.iot.region.amazonaws.com
```

**Note**  
For backward compatibility, AWS IoT Greengrass currently supports legacy VeriSign root CA certificates and endpoints in some AWS Regions\. If you're using a legacy VeriSign root CA certificate, we recommend that you create an ATS endpoint and use an ATS root CA certificate instead\. Otherwise, make sure to use the corresponding legacy endpoints\. For more information, see [Supported legacy endpoints](https://docs.aws.amazon.com/general/latest/gr/greengrass.html#greengrass-legacy-endpoints) in the *Amazon Web Services General Reference*\.

### Endpoints in config\.json<a name="certificate-endpoints-config"></a>

On a Greengrass core device, endpoints are specified in the `coreThing` object in the [`config.json`](#config-json) file\. The `iotHost` property represents the AWS IoT Core endpoint\. The `ggHost` property represents the AWS IoT Greengrass endpoint\. In the following example snippet, these properties specify ATS endpoints\.

```
{
  "coreThing" : {
    ...
    "iotHost" : "abcde1234uwxyz-ats.iot.us-west-2.amazonaws.com",
    "ggHost" : "greengrass-ats.iot.us-west-2.amazonaws.com",
    ...
  },
```

**AWS IoT Core endpoint**  
You can get your AWS IoT Core endpoint by running the [https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) CLI command with the appropriate `--endpoint-type` parameter\.  
+ To return an ATS signed endpoint, run:

  ```
  aws iot describe-endpoint --endpoint-type iot:Data-ATS
  ```
+ To return a legacy VeriSign signed endpoint, run:

  ```
  aws iot describe-endpoint --endpoint-type iot:Data
  ```

**AWS IoT Greengrass endpoint**  
Your AWS IoT Greengrass endpoint is your `iotHost` endpoint with the host prefix replaced by *greengrass*\. For example, the ATS signed endpoint is `greengrass-ats.iot.region.amazonaws.com`\. This uses the same Region as your AWS IoT Core endpoint\.

## Connect on port 443 or through a network proxy<a name="alpn-network-proxy"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 and later\.

Greengrass cores communicate with AWS IoT Core using the MQTT messaging protocol with TLS client authentication\. By convention, MQTT over TLS uses port 8883\. However, as a security measure, restrictive environments might limit inbound and outbound traffic to a small range of TCP ports\. For example, a corporate firewall might open port 443 for HTTPS traffic, but close other ports that are used for less common protocols, such as port 8883 for MQTT traffic\. Other restrictive environments might require all traffic to go through an HTTP proxy before connecting to the internet\.

To enable communication in these scenarios, AWS IoT Greengrass allows the following configurations:
+ **MQTT with TLS client authentication over port 443**\. If your network allows connections to port 443, you can configure the core to use port 443 for MQTT traffic instead of the default port 8883\. This can be a direct connection to port 443 or a connection through a network proxy server\.

  AWS IoT Greengrass uses the [ Application Layer Protocol Network](https://tools.ietf.org/html/rfc7301) \(ALPN\) TLS extension to enable this connection\. As with the default configuration, MQTT over TLS on port 443 uses certificate\-based client authentication\.

  When configured to use a direct connection to port 443, the core supports [over\-the\-air \(OTA\) updates](core-ota-update.md) for AWS IoT Greengrass software\. This support requires AWS IoT Greengrass Core v1\.9\.3 or later\.
+ **HTTPS communication over port 443**\. AWS IoT Greengrass sends HTTPS traffic over port 8443 by default, but you can configure it to use port 443\.
+ **Connection through a network proxy**\. You can configure a network proxy server to act as an intermediary for connecting to the Greengrass core\. Only basic authentication and HTTP and HTTPS proxies are supported\.

  The proxy configuration is passed to user\-defined Lambda functions through the `http_proxy`, `https_proxy`, and `no_proxy` environment variables\. User\-defined Lambda functions must use these passed\-in settings to connect through the proxy\. Common libraries used by Lambda functions to make connections \(such as boto3 or cURL and python `requests` packages\) typically use these environment variables by default\. If a Lambda function also specifies these same environment variables, AWS IoT Greengrass doesn't override them\.
**Important**  
Greengrass cores that are configured to use a network proxy don't support [OTA updates](core-ota-update.md)\.<a name="config-mqtt-port"></a>

**To configure MQTT over port 443**

This feature requires AWS IoT Greengrass Core v1\.7 or later\.

This procedure allows the Greengrass core to use port 443 for MQTT messaging with AWS IoT Core\.

1. Run the following command to stop the Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd stop
   ```

1. Open `greengrass-root/config/config.json` for editing as the su user\.

1. In the `coreThing` object, add the `iotMqttPort` property and set the value to **443**, as shown in the following example\.

   ```
   {
       "coreThing" : {
           "caPath" : "root.ca.pem",
           "certPath" : "12345abcde.cert.pem",
           "keyPath" : "12345abcde.private.key",
           "thingArn" : "arn:aws:iot:us-west-2:123456789012:thing/core-thing-name",
           "iotHost" : "abcd123456wxyz-ats.iot.us-west-2.amazonaws.com",
           "iotMqttPort" : 443,
           "ggHost" : "greengrass-ats.iot.us-west-2.amazonaws.com",
           "keepAlive" : 600
       },
       ...
   }
   ```

1. Start the daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

 <a name="config-http-port"></a>

**To configure HTTPS over port 443**

This feature requires AWS IoT Greengrass Core v1\.8 or later\.

This procedure configures the core to use port 443 for HTTPS communication\.

1. Run the following command to stop the Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd stop
   ```

1. Open `greengrass-root/config/config.json` for editing as the su user\.

1. In the `coreThing` object, add the `iotHttpPort` and `ggHttpPort` properties, as shown in the following example\.

   ```
   {
       "coreThing" : {
           "caPath" : "root.ca.pem",
           "certPath" : "12345abcde.cert.pem",
           "keyPath" : "12345abcde.private.key",
           "thingArn" : "arn:aws:iot:us-west-2:123456789012:thing/core-thing-name",
           "iotHost" : "abcd123456wxyz-ats.iot.us-west-2.amazonaws.com",
           "iotHttpPort" : 443,
           "ggHost" : "greengrass-ats.iot.us-west-2.amazonaws.com",
           "ggHttpPort" : 443,
           "keepAlive" : 600
       },
       ...
   }
   ```

1. Start the daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

 <a name="config-network-proxy"></a>

**To configure a network proxy**

This feature requires AWS IoT Greengrass Core v1\.7 or later\.

This procedure allows AWS IoT Greengrass to connect to the internet through an HTTP or HTTPS network proxy\.

1. Run the following command to stop the Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd stop
   ```

1. Open `greengrass-root/config/config.json` for editing as the su user\.

1. In the `coreThing` object, add the [networkProxy](#networkProxy-object) object, as shown in the following example\.

   ```
   {
       "coreThing" : {
           "caPath" : "root.ca.pem",
           "certPath" : "12345abcde.cert.pem",
           "keyPath" : "12345abcde.private.key",
           "thingArn" : "arn:aws:iot:us-west-2:123456789012:thing/core-thing-name",
           "iotHost" : "abcd123456wxyz-ats.iot.us-west-2.amazonaws.com",
           "ggHost" : "greengrass-ats.iot.us-west-2.amazonaws.com",
           "keepAlive" : 600,
           "networkProxy": {
               "noProxyAddresses" : "http://128.12.34.56,www.mywebsite.com",
               "proxy" : {
                   "url" : "https://my-proxy-server:1100",
                   "username" : "Mary_Major",
                   "password" : "pass@word1357"
               }
           }
       },
       ...
   }
   ```

1. Start the daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

**networkProxy object**

Use the `networkProxy` object to specify information about the network proxy\. This object has the following properties\.


| Field | Description | 
| --- | --- | 
| noProxyAddresses |  Optional\. A comma\-separated list of IP addresses or host names that are exempt from the proxy\.  | 
| proxy |  The proxy to connect to\. A proxy has the following properties\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)  | 

### Allowing endpoints<a name="allow-endpoints-proxy"></a>

Communication between Greengrass devices and AWS IoT Core or AWS IoT Greengrass must be authenticated\. This authentication is based on registered X\.509 device certificates and cryptographic keys\. To allow authenticated requests to pass through proxies without additional encryption, allow the following endpoints\.


| Endpoint | Port | Description | 
| --- | --- | --- | 
| greengrass\.region\.amazonaws\.com | 443 |  Used for control plane operations for group management\.  | 
| `prefix-ats.iot.region.amazonaws.com` or `prefix.iot.region.amazonaws.com` | MQTT: 8883 or 443 HTTPS: 8443 or 443 |  Used for data plane operations for device management, such as shadow sync\. Allow the use of one or both endpoints, depending on whether your core and connected devices use Amazon Trust Services \(preferred\) root CA certificates, legacy root CA certificates, or both\. For more information, see [Service endpoints must match the root CA certificate type](#certificate-endpoints)\.  | 
| `greengrass-ats.iot.region.amazonaws.com` or `greengrass.iot.region.amazonaws.com` | 8443 or 443 |  Used for device discovery operations\. Allow the use of one or both endpoints, depending on whether your core and connected devices use Amazon Trust Services \(preferred\) root CA certificates, legacy root CA certificates, or both\. For more information, see [Service endpoints must match the root CA certificate type](#certificate-endpoints)\.  Clients that connect on port 443 must implement the [ Application Layer Protocol Negotiation \(ALPN\)](https://tools.ietf.org/html/rfc7301) TLS extension and pass `x-amzn-http-ca` as the `ProtocolName` in the `ProtocolNameList`\. For more information, see [Protocols](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) in the *AWS IoT Developer Guide*\.   | 
| \*\.s3\.amazonaws\.com | 443 |  Used for deployment operations and over\-the\-air updates\. This format includes the `*` character because endpoint prefixes are controlled internally and might change at any time\.  | 
| logs\.region\.amazonaws\.com | 443 |  Required if the Greengrass group is configured to write logs to CloudWatch\.  | 

## Configure a write directory for AWS IoT Greengrass<a name="write-directory"></a>

This feature is available for AWS IoT Greengrass Core v1\.6 and later\.

By default, the AWS IoT Greengrass Core software is deployed under a single root directory where AWS IoT Greengrass performs all read and write operations\. However, you can configure AWS IoT Greengrass to use a separate directory for all write operations, including creating directories and files\. In this case, AWS IoT Greengrass uses two top\-level directories:
+ The *greengrass\-root* directory, which you can leave as read\-write or optionally make read\-only\. This contains the AWS IoT Greengrass Core software and other critical components that should remain immutable during runtime, such as certificates and `config.json`\.
+ The specified write directory\. This contains writable content, such as logs, state information, and deployed user\-defined Lambda functions\.

This configuration results in the following directory structure\.

**Greengrass root directory**  

```
greengrass-root/
|-- certs/
|   |-- root.ca.pem
|   |-- hash.cert.pem
|   |-- hash.private.key
|   |-- hash.public.key
|-- config/
|   |-- config.json
|-- ggc/
|   |-- packages/
|       |-- package-version/
|           |-- bin/
|               |-- daemon 
|           |-- greengrassd
|           |-- lambda/
|           |-- LICENSE/
|           |-- release_notes_package-version.html
|               |-- runtime/
|                   |-- java8/
|                   |-- nodejs8.10/
|                   |-- python3.8/
|   |-- core/
```

**Write Directory**  

```
write-directory/
|-- packages/
|   |-- package-version/
|       |-- ggc_root/
|       |-- rootfs_nosys/
|       |-- rootfs_sys/
|       |-- var/
|-- deployment/
|   |-- group/
|       |-- group.json
|   |-- lambda/
|   |-- mlmodel/
|-- var/
|   |-- log/
|   |-- state/
```

 

**To configure a write directory**

1. Run the following command to stop the AWS IoT Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd stop
   ```

1. Open `greengrass-root/config/config.json` for editing as the su user\.

1. Add `writeDirectory` as a parameter and specify the path to the target directory, as shown in the following example\.

   ```
   {
       "coreThing": {
           "caPath": "root-CA.pem",
           "certPath": "hash.pem.crt",
           ...
       },
       ...
       "writeDirectory" : "/write-directory"
   }
   ```
**Note**  
You can update the `writeDirectory` setting as often as you want\. After the setting is updated, AWS IoT Greengrass uses the newly specified write directory at the next start, but doesn't migrate content from the previous write directory\.

1. Now that your write directory is configured, you can optionally make the *greengrass\-root* directory read\-only\. For instructions, see [To Make the Greengrass Root Directory Read\-Only](#configure-ro-directory)\.

   Otherwise, start the AWS IoT Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

 <a name="configure-ro-directory"></a>

**To make the Greengrass root directory read\-only**

Follow these steps only if you want to make the Greengrass root directory read\-only\. The write directory must be configured before you begin\.

1. Grant access permissions to required directories:

   1. Give read and write permissions to the `config.json` owner\.

      ```
      sudo chmod 0600 /greengrass-root/config/config.json
      ```

   1. Make ggc\_user the owner of the certs and system Lambda directories\.

      ```
      sudo chown -R ggc_user:ggc_group /greengrass-root/certs/
      sudo chown -R ggc_user:ggc_group /greengrass-root/ggc/packages/1.11.0/lambda/
      ```
**Note**  
The ggc\_user and ggc\_group accounts are used by default to run system Lambda functions\. If you configured the group\-level [default access identity](lambda-group-config.md#lambda-access-identity-groupsettings) to use different accounts, you should give permissions to that user \(UID\) and group \(GID\) instead\.

1. Make the *greengrass\-root* directory read\-only by using your preferred mechanism\.
**Note**  
One way to make the *greengrass\-root* directory read\-only is to mount the directory as read\-only\. However, to apply over\-the\-air \(OTA\) updates to the AWS IoT Greengrass Core software in a mounted directory, the directory must first be unmounted, and then remounted after the update\. You can add these `umount` and `mount` operations to the `ota_pre_update` and `ota_post_update` scripts\. For more information about OTA updates, see [Greengrass OTA update agent](core-ota-update.md#ota-agent) and [Managed respawn with OTA updates](core-ota-update.md#ota-managed-respawn)\.

1. Start the daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

   If the permissions from step 1 aren't set correctly, tthe daemon won't start\.

## Configure MQTT settings<a name="configure-mqtt"></a>

In the AWS IoT Greengrass environment, local devices, Lambda functions, connectors, and system components can communicate with each other and with AWS IoT Core\. All communication goes through the core, which manages the [subscriptions](gg-sec.md#gg-msg-workflow) that authorize MQTT communication between entities\.

For information about MQTT settings you can configure for AWS IoT Greengrass, see the following sections:
+ [Message quality of service](#message-quality-of-service)
+ [MQTT message queue for cloud targets](#mqtt-message-queue)
+ [MQTT persistent sessions with AWS IoT Core](#mqtt-persistent-sessions)
+ [Client IDs for MQTT connections with AWS IoT](#connection-client-id)
+ [MQTT port for local messaging](#config-local-mqtt-port)
+ [Timeout for publish, subscribe, unsubscribe operations in MQTT connections with the AWS Cloud](#mqtt-operation-timeout)

**Note**  
<a name="sitewise-connector-opcua-support"></a>OPC\-UA is an information exchange standard for industrial communication\. To implement support for OPC\-UA on the Greengrass core, you can use the [IoT SiteWise connector](iot-sitewise-connector.md)\. The connector sends industrial device data from OPC\-UA servers to asset properties in AWS IoT SiteWise\.

### Message quality of service<a name="message-quality-of-service"></a>

AWS IoT Greengrass supports quality of service \(QoS\) levels 0 or 1, depending on your configuration and the target and direction of the communication\. The Greengrass core acts as a client for communication with AWS IoT Core and a message broker for communication on the local network\.

![\[The core as client and local message broker.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/mqtt-qos.png)

For more information about MQTT and QoS, see [Getting Started](https://mqtt.org/getting-started/) on the MQTT website\.

**Communication with the AWS Cloud**  
+ **Outbound messages use QoS 1**

  The core sends messages destined for AWS Cloud targets using QoS 1\. AWS IoT Greengrass uses an MQTT message queue to process these messages\. If message delivery isn't confirmed by AWS IoT Core, the message is spooled to be retried later \(unless the queue is full\)\. This can help minimize data loss from intermittent connectivity\.

  For more information, including how to configure a local storage cache that can persist messages destined for AWS Cloud targets, see [MQTT message queue for cloud targets](#mqtt-message-queue)\.

   
+ **Inbound messages use QoS 0 \(default\) or QoS 1**

  By default, the core subscribes with QoS 0 to messages from AWS Cloud sources\. If you enable persistent sessions, the core subscribes with QoS 1\. This can help minimize data loss from intermittent connectivity\. To manage the QoS for these subscriptions, you configure persistence settings on the local spooler system component\.

  For more information, including how to enable the core to establish a persistent session with AWS Cloud targets, see [MQTT persistent sessions with AWS IoT Core](#mqtt-persistent-sessions)\.

**Communication with local targets**  
All local communication uses QoS 0\. The core makes one attempt to send a message to a local target, which can be a Greengrass Lambda function, connector, or [connected device](what-is-gg.md#greengrass-devices)\. The core doesn't store messages or confirm delivery\. Messages can be dropped anywhere between components\.  
Although direct communication between Lambda functions doesn't use MQTT messaging, the behavior is the same\.

### MQTT message queue for cloud targets<a name="mqtt-message-queue"></a>

MQTT messages that are destined for AWS Cloud targets are queued to await processing\. Queued messages are processed in first in, first out \(FIFO\) order\. After a message is processed and published to AWS IoT Core, the message is removed from the queue\.

By default, the Greengrass core stores unprocessed messages destined for AWS Cloud targets in memory\. You can configure the core to store unprocessed messages in a local storage cache instead\. Unlike in\-memory storage, the local storage cache has the ability to persist across core restarts \(for example, after a group deployment or a device reboot\), so AWS IoT Greengrass can continue to process the messages\. You can also configure the storage size\.

AWS IoT Greengrass uses the spooler system component \(the `GGCloudSpooler` Lambda function\) to manage the message queue\. You can use the following `GGCloudSpooler` environment variables to configure storage settings\.
+ **GG\_CONFIG\_STORAGE\_TYPE**\. The location of the message queue\. The following are valid values:
  + `FileSystem`\. Store unprocessed messages in the local storage cache on the disk of the physical core device\. When the core restarts, queued messages are retained for processing\. Messages are removed after they are processed\.
  + `Memory` \(default\)\. Store unprocessed messages in memory\. When the core restarts, queued messages are lost\.

    This option is optimized for devices with restricted hardware capabilities\. When using this configuration, we recommend that you deploy groups or restart the device when the service disruption is the lowest\.
+ **GG\_CONFIG\_MAX\_SIZE\_BYTES**\. The storage size, in bytes\. This value can be any non\-negative integer **greater than or equal to 262144** \(256 KB\); a smaller size prevents the AWS IoT Greengrass Core software from starting\. The default size is 2\.5 MB\. When the size limit is reached, the oldest queued messages are replaced by new messages\.

**Note**  
This feature is available for AWS IoT Greengrass Core v1\.6 and later\. Earlier versions use in\-memory storage with a queue size of 2\.5 MB\. You cannot configure storage settings for earlier versions\.

#### To cache messages in local storage<a name="configure-local-storage-cache"></a>

You can configure AWS IoT Greengrass to cache messages to the file system so they persist across core restarts\. To do this, you deploy a function definition version where the `GGCloudSpooler` function sets the storage type to `FileSystem`\. You must use the AWS IoT Greengrass API to configure the local storage cache\. You can't do this in the console\.

The following procedure uses the [ `create-function-definition-version`](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition-version.html) CLI command to configure the spooler to save queued messages to the file system\. It also configures a 2\.6 MB queue size\.

1. <a name="get-group-id-latestversion"></a>Get the IDs of the target Greengrass group and group version\. This procedure assumes that this is the latest group and group version\. The following query returns the most recently created group\.

   ```
   aws greengrass list-groups --query "reverse(sort_by(Groups, &CreationTimestamp))[0]"
   ```

   Or, you can query by name\. Group names are not required to be unique, so multiple groups might be returned\.

   ```
   aws greengrass list-groups --query "Groups[?Name=='MyGroup']"
   ```
**Note**  
<a name="find-group-ids-console"></a>You can also find these values in the AWS IoT console\. The group ID is displayed on the group's **Settings** page\. Group version IDs are displayed on the group's **Deployments** page\.

1. <a name="copy-group-id-latestversion"></a>Copy the `Id` and `LatestVersion` values from the target group in the output\.

1. <a name="get-latest-group-version"></a>Get the latest group version\.
   + Replace *group\-id* with the `Id` that you copied\.
   + Replace *latest\-group\-version\-id* with the `LatestVersion` that you copied\.

   ```
   aws greengrass get-group-version \
   --group-id group-id \
   --group-version-id latest-group-version-id
   ```

1. <a name="copy-group-component-arns-except-function"></a>From the `Definition` object in the output, copy the `CoreDefinitionVersionArn` and the ARNs of all other group components except `FunctionDefinitionVersionArn`\. You use these values when you create a new group version\.

1. <a name="parse-function-def-id"></a>From the `FunctionDefinitionVersionArn` in the output, copy the ID of the function definition\. The ID is the GUID that follows the `functions` segment in the ARN, as shown in the following example\.

   ```
   arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/functions/bcfc6b49-beb0-4396-b703-6dEXAMPLEcu5/versions/0f7337b4-922b-45c5-856f-1aEXAMPLEsf6
   ```
**Note**  
Or, you can create a function definition by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html) command, and then copying the ID from the output\.

1. Add a function definition version to the function definition\.
   + Replace *function\-definition\-id* with the `Id` that you copied for the function definition\.
   + Replace *arbitrary\-function\-id* with a name for the function, such as **spooler\-function**\.
   + Add any Lambda functions that you want to include in this version to the `functions` array\. You can use the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html) command to get the Greengrass Lambda functions from an existing function definition version\.
**Warning**  
Make sure that you specify a value for `GG_CONFIG_MAX_SIZE_BYTES` that's **greater than or equal to 262144**\. A smaller size prevents the AWS IoT Greengrass Core software from starting\.

   ```
   aws greengrass create-function-definition-version \
   --function-definition-id function-definition-id \
   --functions '[{"FunctionArn": "arn:aws:lambda:::function:GGCloudSpooler:1","FunctionConfiguration": {"Environment": {"Variables":{"GG_CONFIG_MAX_SIZE_BYTES":"2621440","GG_CONFIG_STORAGE_TYPE":"FileSystem"}},"Executable": "spooler","MemorySize": 32768,"Pinned": true,"Timeout": 3},"Id": "arbitrary-function-id"}]'
   ```
**Note**  
If you previously set the `GG_CONFIG_SUBSCRIPTION_QUALITY` environment variable to [support persistent sessions with AWS IoT Core](#mqtt-persistent-sessions), include it in this function instance\.

1. <a name="copy-function-def-version-arn"></a>Copy the `Arn` of the function definition version from the output\.

1. <a name="create-group-version-with-sys-lambda"></a>Create a group version that contains the system Lambda function\.
   + Replace *group\-id* with the `Id` for the group\.
   + Replace *core\-definition\-version\-arn* with the `CoreDefinitionVersionArn` that you copied from the latest group version\.
   + Replace *function\-definition\-version\-arn* with the `Arn` that you copied for the new function definition version\.
   + Replace the ARNs for other group components \(for example, `SubscriptionDefinitionVersionArn` or `DeviceDefinitionVersionArn`\) that you copied from the latest group version\.
   + Remove any unused parameters\. For example, remove the `--resource-definition-version-arn` if your group version doesn't contain any resources\.

   ```
   aws greengrass create-group-version \
   --group-id group-id \
   --core-definition-version-arn core-definition-version-arn \
   --function-definition-version-arn function-definition-version-arn \
   --device-definition-version-arn device-definition-version-arn \
   --logger-definition-version-arn logger-definition-version-arn \
   --resource-definition-version-arn resource-definition-version-arn \
   --subscription-definition-version-arn subscription-definition-version-arn
   ```

1. <a name="copy-group-version-id"></a>Copy the `Version` from the output\. This is the ID of the new group version\.

1. <a name="create-group-deployment"></a>Deploy the group with the new group version\.
   + Replace *group\-id* with the `Id` that you copied for the group\.
   + Replace *group\-version\-id* with the `Version` that you copied for the new group version\.

   ```
   aws greengrass create-deployment \
   --group-id group-id \
   --group-version-id group-version-id \
   --deployment-type NewDeployment
   ```

 To update the storage settings, you use the AWS IoT Greengrass API to create a new function definition version that contains the `GGCloudSpooler` function with the updated configuration\. Then add the function definition version to a new group version \(along with your other group components\) and deploy the group version\. If you want to restore the default configuration, you can deploy a function definition version that doesn't include the `GGCloudSpooler` function\. 

 This system Lambda function isn't visible in the console\. However, after the function is added to the latest group version, it's included in deployments that you make from the console, unless you use the API to replace or remove it\. 

### MQTT persistent sessions with AWS IoT Core<a name="mqtt-persistent-sessions"></a>

This feature is available for AWS IoT Greengrass Core v1\.10 and later\.

A Greengrass core can establish a persistent session with the AWS IoT message broker\. A persistent session is an ongoing connection that allows the core to receive messages sent while the core is offline\. The core is the client in the connection\.

In a persistent session, the AWS IoT message broker saves all subscriptions the core makes during the connection\. If the core disconnects, the AWS IoT message broker stores unacknowledged and new messages published as QoS 1 and destined for local targets, such as Lambda functions and [connected devices](what-is-gg.md#greengrass-devices)\. When the core reconnects, the persistent session is resumed and the AWS IoT message broker sends stored messages to the core at a maximum rate of 10 messages per second\. Persistent sessions have a default expiry period of 1 hour, which begins when the message broker detects that the core disconnects\. For more information, see [MQTT persistent sessions](https://docs.aws.amazon.com/iot/latest/developerguide/mqtt-persistent-sessions.html) in the *AWS IoT Developer Guide*\.

AWS IoT Greengrass uses the spooler system component \(the `GGCloudSpooler` Lambda function\) to create subscriptions that have AWS IoT as the source\. You can use the following `GGCloudSpooler` environment variable to configure persistent sessions\.
+ **GG\_CONFIG\_SUBSCRIPTION\_QUALITY**\. The quality of subscriptions that have AWS IoT as the source\. The following are valid values:
  + `AtMostOnce` \(default\)\. Disables persistent sessions\. Subscriptions use QoS 0\.
  + `AtLeastOncePersistent`\. Enables persistent sessions\. Sets the `cleanSession` flag to `0` in `CONNECT` messages and subscribes with QoS 1\.

    Messages published with QoS 1 that the core receives are guaranteed to reach the Greengrass daemon's in\-memory work queue\. The core acknowledges the message after it's added to the queue\. Subsequent communication from the queue to the local target \(for example, Greengrass Lambda function, connector, or device\) is sent as QoS 0\. AWS IoT Greengrass doesn't guarantee delivery to local targets\.
**Note**  
You can use the [maxWorkItemCount](#config-json-runtime) configuration property to control the size of the work item queue\. For example, you can increase the queue size if your workload requires heavy MQTT traffic\.

    When persistent sessions are enabled, the core opens at least one additional connection for MQTT message exchange with AWS IoT\. For more information, see [Client IDs for MQTT connections with AWS IoT](#connection-client-id)\.

#### To configure MQTT persistent sessions<a name="configure-persistent-sessions"></a>

You can configure AWS IoT Greengrass to use persistent sessions with AWS IoT Core\. To do this, you deploy a function definition version where the `GGCloudSpooler` function sets the subscription quality to `AtLeastOncePersistent`\. This setting applies to all your subscriptions that have AWS IoT Core \(`cloud`\) as the source\. You must use the AWS IoT Greengrass API to configure persistent sessions\. You can't do this in the console\.

The following procedure uses the [ `create-function-definition-version`](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition-version.html) CLI command to configure the spooler to use persistent sessions\. In this procedure, we assume that you're updating the configuration of the latest group version of an existing group\.

1. <a name="get-group-id-latestversion"></a>Get the IDs of the target Greengrass group and group version\. This procedure assumes that this is the latest group and group version\. The following query returns the most recently created group\.

   ```
   aws greengrass list-groups --query "reverse(sort_by(Groups, &CreationTimestamp))[0]"
   ```

   Or, you can query by name\. Group names are not required to be unique, so multiple groups might be returned\.

   ```
   aws greengrass list-groups --query "Groups[?Name=='MyGroup']"
   ```
**Note**  
<a name="find-group-ids-console"></a>You can also find these values in the AWS IoT console\. The group ID is displayed on the group's **Settings** page\. Group version IDs are displayed on the group's **Deployments** page\.

1. <a name="copy-group-id-latestversion"></a>Copy the `Id` and `LatestVersion` values from the target group in the output\.

1. <a name="get-latest-group-version"></a>Get the latest group version\.
   + Replace *group\-id* with the `Id` that you copied\.
   + Replace *latest\-group\-version\-id* with the `LatestVersion` that you copied\.

   ```
   aws greengrass get-group-version \
   --group-id group-id \
   --group-version-id latest-group-version-id
   ```

1. <a name="copy-group-component-arns-except-function"></a>From the `Definition` object in the output, copy the `CoreDefinitionVersionArn` and the ARNs of all other group components except `FunctionDefinitionVersionArn`\. You use these values when you create a new group version\.

1. <a name="parse-function-def-id"></a>From the `FunctionDefinitionVersionArn` in the output, copy the ID of the function definition\. The ID is the GUID that follows the `functions` segment in the ARN, as shown in the following example\.

   ```
   arn:aws:greengrass:us-west-2:123456789012:/greengrass/definition/functions/bcfc6b49-beb0-4396-b703-6dEXAMPLEcu5/versions/0f7337b4-922b-45c5-856f-1aEXAMPLEsf6
   ```
**Note**  
Or, you can create a function definition by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html) command, and then copying the ID from the output\.

1. Add a function definition version to the function definition\.
   + Replace *function\-definition\-id* with the `Id` that you copied for the function definition\.
   + Replace *arbitrary\-function\-id* with a name for the function, such as **spooler\-function**\.
   + Add any Lambda functions that you want to include in this version to the `functions` array\. You can use the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html) command to get the Greengrass Lambda functions from an existing function definition version\.

   ```
   aws greengrass create-function-definition-version \
   --function-definition-id function-definition-id \
   --functions '[{"FunctionArn": "arn:aws:lambda:::function:GGCloudSpooler:1","FunctionConfiguration": {"Environment": {"Variables":{"GG_CONFIG_SUBSCRIPTION_QUALITY":"AtLeastOncePersistent"}},"Executable": "spooler","MemorySize": 32768,"Pinned": true,"Timeout": 3},"Id": "arbitrary-function-id"}]'
   ```
**Note**  
If you previously set the `GG_CONFIG_STORAGE_TYPE` or `GG_CONFIG_MAX_SIZE_BYTES` environment variables to [define storage settings](#mqtt-message-queue), include them in this function instance\.

1. <a name="copy-function-def-version-arn"></a>Copy the `Arn` of the function definition version from the output\.

1. <a name="create-group-version-with-sys-lambda"></a>Create a group version that contains the system Lambda function\.
   + Replace *group\-id* with the `Id` for the group\.
   + Replace *core\-definition\-version\-arn* with the `CoreDefinitionVersionArn` that you copied from the latest group version\.
   + Replace *function\-definition\-version\-arn* with the `Arn` that you copied for the new function definition version\.
   + Replace the ARNs for other group components \(for example, `SubscriptionDefinitionVersionArn` or `DeviceDefinitionVersionArn`\) that you copied from the latest group version\.
   + Remove any unused parameters\. For example, remove the `--resource-definition-version-arn` if your group version doesn't contain any resources\.

   ```
   aws greengrass create-group-version \
   --group-id group-id \
   --core-definition-version-arn core-definition-version-arn \
   --function-definition-version-arn function-definition-version-arn \
   --device-definition-version-arn device-definition-version-arn \
   --logger-definition-version-arn logger-definition-version-arn \
   --resource-definition-version-arn resource-definition-version-arn \
   --subscription-definition-version-arn subscription-definition-version-arn
   ```

1. <a name="copy-group-version-id"></a>Copy the `Version` from the output\. This is the ID of the new group version\.

1. <a name="create-group-deployment"></a>Deploy the group with the new group version\.
   + Replace *group\-id* with the `Id` that you copied for the group\.
   + Replace *group\-version\-id* with the `Version` that you copied for the new group version\.

   ```
   aws greengrass create-deployment \
   --group-id group-id \
   --group-version-id group-version-id \
   --deployment-type NewDeployment
   ```

1. \(Optional\) Increase the [maxWorkItemCount](#config-json-runtime) property in the core configuration file\. This can help the core handle increased MQTT traffic and communication with local targets\.

 To update the core with these configuration changes, you use the AWS IoT Greengrass API to create a new function definition version that contains the `GGCloudSpooler` function with the updated configuration\. Then add the function definition version to a new group version \(along with your other group components\) and deploy the group version\. If you want to restore the default configuration, you can create a function definition version that doesn't include the `GGCloudSpooler` function\. 

 This system Lambda function isn't visible in the console\. However, after the function is added to the latest group version, it's included in deployments that you make from the console, unless you use the API to replace or remove it\. 

### Client IDs for MQTT connections with AWS IoT<a name="connection-client-id"></a>

This feature is available for AWS IoT Greengrass Core v1\.8 and later\.

The Greengrass core opens MQTT connections with AWS IoT Core for operations such as shadow sync and certificate management\. For these connections, the core generates predictable client IDs based on the core thing name\. Predictable client IDs can be used with monitoring, auditing, and pricing features, including AWS IoT Device Defender and [ AWS IoT lifecycle events](https://docs.aws.amazon.com/iot/latest/developerguide/life-cycle-events.html)\. You can also create logic around predictable client IDs \(for example, [subscribe policy](https://docs.aws.amazon.com/iot/latest/developerguide/pub-sub-policy.html#pub-sub-policy-cert) templates based on certificate attributes\)\.

------
#### [ GGC v1\.9 and later ]

Two Greengrass system components open MQTT connections with AWS IoT Core\. These components use the following patterns to generate the client IDs for the connections\.


| Operation | Client ID pattern | 
| --- | --- | 
| Deployments | `core-thing-name` Example: `MyCoreThing` Use this client ID for connect, disconnect, subscribe, and unsubscribe lifecycle event notifications\. | 
| MQTT message exchange with AWS IoT Core |  `core-thing-name-cn` Example: `MyCoreThing-c01` `n` is an integer that starts at 00 and increments with each new connection to a maximum number of 250\. The number of connections is determined by the number of devices that sync their shadow state with AWS IoT Core \(maximum 2500 per group\) and the number of subscriptions with `cloud` as their source in the group \(maximum 10000 per group\)\. You can use the following formula to calculate the number of MQTT connections per account: `number of connections per account = (1 + (2 + number of MQTT topics in AWS IoT Core + number of device shadows synced) / 50) * Greengrass cores` Where, [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html) To reduce the number of MQTT connections and help reduce costs, you can use local Lambda functions to aggregate data at the edge\. Then you send the aggregated data to the AWS Cloud\. As a result, you use fewer MQTT topics in AWS IoT Core\. The spooler system component connects with AWS IoT Core to exchange messages for subscriptions with a cloud source or target\. The spooler also acts as proxy for message exchange between AWS IoT Core and the local shadow service and device certificate manager\.  If you enable [persistent sessions](#mqtt-persistent-sessions) for subscription with AWS IoT Core, the core opens at least one additional connection to use in a persistent session\. The system components don't support persistent sessions, so they can't share that connection\.   | 

------
#### [ GGC v1\.8 ]

Several Greengrass system components open MQTT connections with AWS IoT Core\. These components use the following patterns to generate the client IDs for the connections\.


| Operation | Client ID pattern | 
| --- | --- | 
| Deployments | `core-thing-name` Example: `MyCoreThing` Use this client ID for connect, disconnect, subscribe, and unsubscribe lifecycle event notifications\. | 
| MQTT message exchange with AWS IoT Core | `core-thing-name-spr` Example: `MyCoreThing-spr` | 
| Shadow sync | `core-thing-name-snn` Example: `MyCoreThing-s01` `nn` is an integer that starts at 00 and increments with each new connection to a maximum of 03\. The number of connections is determined by the number of devices \(maximum 200 devices per group\) that sync their shadow state with AWS IoT Core \(maximum 50 subscriptions per connection\)\. | 
| Device certificate management | `core-thing-name-dcm` Example: `MyCoreThing-dcm` | 

------

**Note**  
Duplicate client IDs used in simultaneous connections can cause an infinite connect\-disconnect loop\. This can happen if another device is hardcoded to use the core device name as the client ID in connections\. For more information, see this [troubleshooting step](gg-troubleshooting.md#config-client-id)\.

Greengrass devices are also fully integrated with the Fleet Indexing service of AWS IoT Device Management\. This allows you to index and search for devices based on device attributes, shadow state, and connection state in the cloud\. For example, Greengrass devices establish at least one connection that uses the thing name as the client ID, so you can use device connectivity indexing to discover which Greengrass devices are currently connected or disconnected to AWS IoT Core\. For more information, see [Fleet indexing service](https://docs.aws.amazon.com/iot/latest/developerguide/iot-indexing.html) in the *AWS IoT Developer Guide*\.

### Configure the MQTT port for local messaging<a name="config-local-mqtt-port"></a>

This feature requires AWS IoT Greengrass Core v1\.10 or later\.

The Greengrass core acts as the local message broker for MQTT messaging between local Lambda functions, connectors, and [Greengrass devices](what-is-gg.md#greengrass-devices)\. By default, the core uses port 8883 for MQTT traffic on the local network\. You might want to change the port to avoid a conflict with other software that runs on port 8883\.

**To configure the port number that the core uses for local MQTT traffic**

1. Run the following command to stop the Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd stop
   ```

1. Open `greengrass-root/config/config.json` for editing as the su user\.

1. In the `coreThing` object, add the `ggMqttPort` property and set the value to the port number you want to use\. Valid values are 1024 to 65535\. The following example sets the port number to `9000`\.

   ```
   {
       "coreThing" : {
           "caPath" : "root.ca.pem",
           "certPath" : "12345abcde.cert.pem",
           "keyPath" : "12345abcde.private.key",
           "thingArn" : "arn:aws:iot:us-west-2:123456789012:thing/core-thing-name",
           "iotHost" : "abcd123456wxyz-ats.iot.us-west-2.amazonaws.com",
           "ggHost" : "greengrass-ats.iot.us-west-2.amazonaws.com",
           "ggMqttPort" : 9000,
           "keepAlive" : 600
       },
       ...
   }
   ```

1. Start the daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

1. If [automatic IP detection](#ip-auto-detect) is enabled for the core, the configuration is complete\.

   If automatic IP detection is not enabled, you must update the connectivity information for the core\. This allows Greengrass devices to receive the correct port number during discovery operations to acquire core connectivity information\. You can use the AWS IoT console or AWS IoT Greengrass API to update the core connectivity information\. For this procedure, you update the port number only\. The local IP address for the core remains the same\.  
**To update the connectivity information for the core \(console\)**  

   1. On the group configuration page, choose **Cores**, and then choose the core\.

   1. On the core details page, choose **Connectivity**, and then choose **Edit**\.

   1. Choose **Add another connection**, enter your current local IP address and the new port number\. The following example sets the port number `9000` for the IP address `192.168.1.8`\.  
![\[The core as client and local message broker.\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-group-cores-connectivity-edit.png)

   1. Remove the obsolete endpoint, and then choose **Update**  
**To update the connectivity information for the core \(API\)**  
   + Use the [UpdateConnectivityInfo](https://docs.aws.amazon.com/greengrass/latest/apireference/updateconnectivityinfo-put.html) action\. The following example uses `update-connectivity-info` in the AWS CLI to set the port number `9000` for the IP address `192.168.1.8`\.

     ```
     aws greengrass update-connectivity-info \
         --thing-name "MyGroup_Core" \
         --connectivity-info "[{\"Metadata\":\"\",\"PortNumber\":9000,\"HostAddress\":\"192.168.1.8\",\"Id\":\"localIP_192.168.1.8\"},{\"Metadata\":\"\",\"PortNumber\":8883,\"HostAddress\":\"127.0.0.1\",\"Id\":\"localhost_127.0.0.1_0\"}]"
     ```
**Note**  
You can also configure the port that the core uses for MQTT messaging with AWS IoT Core\. For more information, see [Connect on port 443 or through a network proxy](#alpn-network-proxy)\.

### Timeout for publish, subscribe, unsubscribe operations in MQTT connections with the AWS Cloud<a name="mqtt-operation-timeout"></a>

This feature is available in AWS IoT Greengrass v1\.10\.2 or later\.

You can configure the amount of time \(in seconds\) to allow the Greengrass core to complete a publish, subscribe, or unsubscribe operation in MQTT connections to AWS IoT Core\. You might want to adjust this setting if the operations time out because of bandwidth constraints or high latency\. To configure this setting in the [config\.json](#config-json) file, add or change the `mqttOperationTimeout` property in the `coreThing` object\. For example:

```
{
  "coreThing": {
    "mqttOperationTimeout": 10,
    "caPath": "root-ca.pem",
    "certPath": "hash.cert.pem",
    "keyPath": "hash.private.key",
    ...
  },
  ...
}
```

The default timeout is 5 seconds\. The minimum timeout is 5 seconds\.

## Activate automatic IP detection<a name="ip-auto-detect"></a>

You can configure AWS IoT Greengrass to enable devices in a Greengrass group to automatically discover the Greengrass core\. When enabled, the core watches for changes to its IP addresses\. If an address changes, the core publishes an updated list of addresses\. These addresses are made available to devices that are in the same Greengrass group as the core\.

**Note**  
The AWS IoT policy for connected devices must grant the `greengrass:Discover` permission to allow devices to retrieve connectivity information for the core\. For more information about the policy statement, see [Discovery authorization](gg-discover-api.md#gg-discover-auth)\.

To enable this feature from the AWS IoT Greengrass console, choose **Automatic detection** when you deploy your Greengrass group for the first time\. You can also enable or disable this feature on the group's **Settings** page under **Core connectivity information**\. Automatic IP detection is enabled if **Automatically detect and override connection information** is selected\.

To manage automatic discovery with the AWS IoT Greengrass API, you must configure the `IPDetector` system Lambda function\. The following procedure shows how to use the [ create\-function\-definition\-version](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition-version.html) CLI command to configure automatic discovery of the Greengrass core\.

1. <a name="get-group-id-latestversion"></a>Get the IDs of the target Greengrass group and group version\. This procedure assumes that this is the latest group and group version\. The following query returns the most recently created group\.

   ```
   aws greengrass list-groups --query "reverse(sort_by(Groups, &CreationTimestamp))[0]"
   ```

   Or, you can query by name\. Group names are not required to be unique, so multiple groups might be returned\.

   ```
   aws greengrass list-groups --query "Groups[?Name=='MyGroup']"
   ```
**Note**  
<a name="find-group-ids-console"></a>You can also find these values in the AWS IoT console\. The group ID is displayed on the group's **Settings** page\. Group version IDs are displayed on the group's **Deployments** page\.

1. <a name="copy-group-id-latestversion"></a>Copy the `Id` and `LatestVersion` values from the target group in the output\.

1. <a name="get-latest-group-version"></a>Get the latest group version\.
   + Replace *group\-id* with the `Id` that you copied\.
   + Replace *latest\-group\-version\-id* with the `LatestVersion` that you copied\.

   ```
   aws greengrass get-group-version \
   --group-id group-id \
   --group-version-id latest-group-version-id
   ```

1. <a name="copy-group-component-arns-except-function"></a>From the `Definition` object in the output, copy the `CoreDefinitionVersionArn` and the ARNs of all other group components except `FunctionDefinitionVersionArn`\. You use these values when you create a new group version\.

1. From the `FunctionDefinitionVersionArn` in the output, copy the ID of the function definition and the function definition version:

   ```
   arn:aws:greengrass:region:account-id:/greengrass/groups/function-definition-id/versions/function-definition-version-id
   ```
**Note**  
You can optionally create a function definition by running the [ `create-function-definition`](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html) command, and then copy the ID from the output\.

1.  Use the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html) command to get the current definition state\. Use the *function\-definition\-id* you copied for the function definiton\. For example, *4d941bc7\-92a1\-4f45\-8d64\-EXAMPLEf76c3*\. 

   ```
   aws greengrass get-function-definition-version
   --function-definition-id function-definition-id
   --function-definition-version-id function-definition-version-id
   ```

    Make a note of the listed function configurations\. You will need to include these when creating a new function definition version in order to prevent loss of your current definition settings\. 

1.  Add a function definition version to the function definition\. 
   + Replace *function\-definition\-id* with the `Id` that you copied for the function definition\. For example, *4d941bc7\-92a1\-4f45\-8d64\-EXAMPLEf76c3*\.
   + Replace *arbitrary\-function\-id* with a name for the function, such as **auto\-detection\-function**\.
   + Add all Lambda functions that you want to include in this version to the `functions` array, such as any listed in the previous step\.

   ```
   aws greengrass create-function-definition-version \
   --function-definition-id function-definition-id \
   --functions '[{"FunctionArn":"arn:aws:lambda:::function:GGIPDetector:1","Id":"arbitrary-function-id","FunctionConfiguration":{"Pinned":true,"MemorySize":32768,"Timeout":3}}]'\
   --region us-west-2
   ```

1. <a name="copy-function-def-version-arn"></a>Copy the `Arn` of the function definition version from the output\.

1. <a name="create-group-version-with-sys-lambda"></a>Create a group version that contains the system Lambda function\.
   + Replace *group\-id* with the `Id` for the group\.
   + Replace *core\-definition\-version\-arn* with the `CoreDefinitionVersionArn` that you copied from the latest group version\.
   + Replace *function\-definition\-version\-arn* with the `Arn` that you copied for the new function definition version\.
   + Replace the ARNs for other group components \(for example, `SubscriptionDefinitionVersionArn` or `DeviceDefinitionVersionArn`\) that you copied from the latest group version\.
   + Remove any unused parameters\. For example, remove the `--resource-definition-version-arn` if your group version doesn't contain any resources\.

   ```
   aws greengrass create-group-version \
   --group-id group-id \
   --core-definition-version-arn core-definition-version-arn \
   --function-definition-version-arn function-definition-version-arn \
   --device-definition-version-arn device-definition-version-arn \
   --logger-definition-version-arn logger-definition-version-arn \
   --resource-definition-version-arn resource-definition-version-arn \
   --subscription-definition-version-arn subscription-definition-version-arn
   ```

1. <a name="copy-group-version-id"></a>Copy the `Version` from the output\. This is the ID of the new group version\.

1. <a name="create-group-deployment"></a>Deploy the group with the new group version\.
   + Replace *group\-id* with the `Id` that you copied for the group\.
   + Replace *group\-version\-id* with the `Version` that you copied for the new group version\.

   ```
   aws greengrass create-deployment \
   --group-id group-id \
   --group-version-id group-version-id \
   --deployment-type NewDeployment
   ```

 If you want to manually input the IP address of your Greengrass core, you can complete this tutorial with a different function definition that does not include the `IPDetector` function\. This will prevent the detection function from locating and automatically inputting your Greengrass core IP address\.  

 This system Lambda function isn't visible in the Lambda console\. After the function is added to the latest group version, it's included in deployments that you make from the console, unless you use the API to replace or remove it\. 

## Configure the init system to start the Greengrass daemon<a name="start-on-boot"></a>

It's a good practice to set up your init system to start the Greengrass daemon during boot, especially when managing large fleets of devices\.

**Note**  
If you used `apt` to install the AWS IoT Greengrass Core software, you can use the systemd scripts to enable start on boot\. For more information, see [Use systemd scripts to manage the Greengrass daemon lifecycle](install-ggc.md#ggc-package-manager-systemd)\.

There are different types of init system, such as initd, systemd, and SystemV, and they use similar configuration parameters\. The following example is a service file for systemd\. The `Type` parameter is set to `forking` because greengrassd \(which is used to start Greengrass\) forks the Greengrass daemon process, and the `Restart` parameter is set to `on-failure` to direct systemd to restart Greengrass if Greengrass enters a failed state\.

**Note**  
To see if your device uses systemd, run the `check_ggc_dependencies` script as described in [Module 1](module1.md)\. Then to use systemd, make sure that the `useSystemd` parameter in [`config.json`](#config-json) is set to `yes`\.

```
[Unit]
Description=Greengrass Daemon

[Service]
Type=forking
PIDFile=/var/run/greengrassd.pid
Restart=on-failure
ExecStart=/greengrass/ggc/core/greengrassd start
ExecReload=/greengrass/ggc/core/greengrassd restart
ExecStop=/greengrass/ggc/core/greengrassd stop

[Install]
WantedBy=multi-user.target
```

For information about how to create and enable a service file for systemd on a Raspberry Pi, see [SYSTEMD](https://www.raspberrypi.org/documentation/linux/usage/systemd.md) in the Raspberry Pi documentation\.

## See also<a name="cores-see-also"></a>
+ [What is AWS IoT Greengrass?](what-is-gg.md)
+ [Supported platforms and requirements](what-is-gg.md#gg-platforms)
+ [Getting started with AWS IoT Greengrass](gg-gs.md)
+ [Overview of the AWS IoT Greengrass group object model](deployments.md#api-overview)
+ [Hardware security integration](hardware-security.md)