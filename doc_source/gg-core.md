# Configure the AWS IoT Greengrass Core<a name="gg-core"></a>

An AWS IoT Greengrass core is an AWS IoT thing \(device\)\. Like other AWS IoT devices, a core exists in the registry, has a device shadow, and uses a device certificate to authenticate with AWS IoT\. The core device runs the AWS IoT Greengrass core software, which enables it to manage local processes for Greengrass groups, such as communication, shadow sync, and token exchange\.

The AWS IoT Greengrass core software provides the following functionality:
+ Deployment and local execution of connectors and Lambda functions\.
+ Secure, encrypted storage of local secrets and controlled access by connectors and Lambda functions\.
+ MQTT messaging over the local network between devices, connectors, and Lambda functions using managed subscriptions\.
+ MQTT messaging between AWS IoT and devices, connectors, and Lambda functions using managed subscriptions\.
+ Secure connections between devices and the cloud using device authentication and authorization\.
+ Local shadow synchronization of devices\. Shadows can be configured to sync with the cloud\.
+ Controlled access to local device and volume resources\.
+ Deployment of cloud\-trained machine learning models for running local inference\.
+ Automatic IP address detection that enables devices to discover the Greengrass core device\.
+ Central deployment of new or updated group configuration\. After the configuration data is downloaded, the core device is restarted automatically\.
+ Secure, over\-the\-air software updates of user\-defined Lambda functions\.

## AWS IoT Greengrass Core Configuration File<a name="config-json"></a>

The configuration file for the AWS IoT Greengrass core software is the `config.json` file, which is located in the `/greengrass-root/config` directory\.

**Note**  
*greengrass\-root* represents the path where the AWS IoT Greengrass core software is installed on your device\. If you installed the software by following the [Getting Started](gg-gs.md) tutorial, then this is the `/greengrass` directory\.  
If you use the **Easy group creation** option from the AWS IoT Greengrass console, then the `config.json` file is deployed to the core device in a working state\.

 You can review the contents of this file by running the following command:

```
cat /greengrass-root/config/config.json
```

The following is an example `config.json` file\.

------
#### [ GGC v1\.7 ]

This is the `config.json` file that's generated when the Greengrass group and core are created with the **Easy group creation** option in the AWS IoT Greengrass console\.

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


| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the AWS IoT root CA relative to the `/greengrass-root/certs` directory\.  |  For backward compatibility with versions prior to 1\.7\.0\. This property is ignored when the `crypto` is present\. Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\.  | 
| certPath |  The path to the core device certificate relative to the `/greengrass-root/certs` directory\.  | For backward compatibility with versions prior to 1\.7\.0\. This property is ignored when the crypto is present\. | 
| keyPath | The path to the core private key relative to /greengrass\-root/certs directory\. | For backward compatibility with versions prior to 1\.7\.0\. This property is ignored when the crypto is present\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS IoT Greengrass core device\. | Find this for your core in the AWS IoT Greengrass console under Cores, or by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html) CLI command\. | 
| iotHost | Your AWS IoT endpoint\. | Find this in the AWS IoT Core console under **Settings**, or by running the [https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) CLI command\. This assumes that you're using an Amazon Trust Services \(ATS\) endpoint\. For more information, see the [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html) documentation\. Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\. | 
| ggHost | Your AWS IoT Greengrass endpoint\. | This is your `iotHost` endpoint with the host prefix replaced by *greengrass* \(for example, `greengrass-ats.iot.region.amazonaws.com`\)\. Use the same region as `iotHost`\. Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\. | 
| iotMqttPort | Optional\. The port number to use for MQTT communication with AWS IoT\. | Valid values are 8883 or 443\. The default value is 8883\. For more information, [Connect on Port 443 or Through a Network Proxy](#alpn-network-proxy)\. | 
| keepAlive | Optional\. The MQTT KeepAlive period, in seconds\. | The default value is 600\. | 
| networkProxy | Optional\. An object that defines a proxy server to connect to\. | This can be an HTTP or HTTPS proxy\. For more information, [Connect on Port 443 or Through a Network Proxy](#alpn-network-proxy)\. | 


**`runtime`**  

| Field | Description | Notes | 
| --- | --- | --- | 
| cgroup | 
| useSystemd | Indicates whether your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Valid values are yes or no\. Run the check\_ggc\_dependencies script in [Module 1](module1.md) to see if your device uses systemd\. | 

**crypto**

The `crypto` object is added in v1\.7\.0\. It introduces properties that support private key storage on a hardware security module \(HSM\) through PKCS\#11 and local secret storage\. For more information, see [Hardware Security Integration](hardware-security.md) and [Deploy Secrets to the AWS IoT Greengrass Core](secrets.md)\. Configurations for private key storage on HSMs or in the file system are supported\.


| Field | Description | Notes | 
| --- | --- | --- | 
| <a name="config-capath"></a>caPath |  The absolute path to the AWS IoT root CA\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\. Make sure that your [endpoints correspond to your certificate type](#certificate-endpoints)\.  | 
| PKCS11 | 
| OpenSSLEngine |  Optional\. The absolute path to the OpenSSL engine `.so` file to enable PKCS\#11 support on OpenSSL\.  |  Must be a path to a file on the file system\. This property is required if you're using the Greengrass OTA update agent with hardware security\. For more information, see [Configure Support for Over\-the\-Air Updates](hardware-security.md#hardware-security-ota-updates)\.  | 
| P11Provider |  The absolute path to the PKCS\#11 implementation's libdl\-loadable library\.  |  Must be a path to a file on the file system\.  | 
| slotLabel |  The slot label that's used to identify the hardware module\.  |  Must conform to PKCS\#11 label specifications\.  | 
| slotUserPin |  The user pin that's used to authenticate the Greengrass core to the module\.  |  Must have sufficient permissions to perform C\_Sign with the configured private keys\.  | 
| principals | 
| IoTCertificate | The certificate and private key that the core uses to make requests to AWS IoT\. | 
| IoTCertificate  \.privateKeyPath  |  The path to the core private key\.  |  For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\.  | 
| IoTCertificate  \.certificatePath |  The absolute path to the core device certificate\.  |  Must be a file URI of the form: `file:///absolute/path/to/file`\.  | 
| MQTTServerCertificate | Optional\. The private key that the core uses in combination with the certificate to act as an MQTT server or gateway\. | 
| MQTTServerCertificate  \.privateKeyPath |  The path to the local MQTT server private key\.  |  Use this value to specify your own private key for the local MQTT server\. For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. If this property is omitted, AWS IoT Greengrass rotates the key based your rotation settings\. If specified, the customer is responsible for rotating the key\.  | 
| SecretsManager | The private key that secures the data key used for encryption\. For more information, see [Deploy Secrets to the AWS IoT Greengrass Core](secrets.md)\. | 
| SecretsManager  \.privateKeyPath |  The path to the local secrets manager private key\.  |  For file system storage, must be a file URI of the form: `file:///absolute/path/to/file`\. For HSM storage, must be an [RFC 7512 PKCS\#11](https://tools.ietf.org/html/rfc7512) path that specifies the object label\. The private key must be generated using the [PKCS\#1 v1\.5](https://tools.ietf.org/html/rfc2313) padding mechanism\.  | 

The following configuration properties are also supported:


****  

| Field | Description | Notes | 
| --- | --- | --- | 
|  `mqttMaxConnectionRetryInterval`  |  Optional\. The maximum interval \(in seconds\) between MQTT connection retries if the connection is dropped\.  |  Specify this value as an unsigned integer\. The default is `60`\.  | 
|  `managedRespawn`  |  Optional\. Indicates that the OTA agent needs to run custom code before an update\.  |  Valid values are `true` or `false`\. For more information, see [OTA Updates of AWS IoT Greengrass Core Software](core-ota-update.md)\.  | 
|  `writeDirectory`  |  Optional\. The write directory where AWS IoT Greengrass creates all read\-write resources\.  |  For more information, see [Configure a Write Directory for AWS IoT Greengrass](#write-directory)\.  | 

------
#### [ GGC v1\.6 ]

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

**Note**  
If you use the **Easy group creation** option from the AWS IoT Greengrass console, then the `config.json` file is deployed to the core device in a working state that specifies the default configuration\.

The `config.json` file supports the following properties:


****  

| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the [AWS IoT root CA](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html) relative to the `/greengrass-root/certs` directory\.  |  Save the file under `/greengrass-root/certs`\.  | 
| certPath |  The path to the AWS IoT Greengrass core certificate relative to the `/greengrass-root/certs` directory\.  | Save the file under /greengrass\-root/certs\. | 
| keyPath | The path to the AWS IoT Greengrass core private key relative to /greengrass\-root/certs directory\. | Save the file under /greengrass\-root/certs\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS IoT Greengrass core device\. | You can find this for your core in the AWS IoT Greengrass console under Cores, or by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html) CLI command\. | 
| iotHost | Your AWS IoT endpoint\. | Find this in the AWS IoT Core console under Settings, or by running the [https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) CLI command\. | 
| ggHost | Your AWS IoT Greengrass endpoint\. | This value uses the format greengrass\.iot\.region\.amazonaws\.com\. Use the same region as iotHost\. | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default is 600\. | 
|  `mqttMaxConnectionRetryInterval`  |  The maximum interval \(in seconds\) between MQTT connection retries if the connection is dropped\.  |  Specify this value as an unsigned integer\. This is an optional value\. The default is `60`\.  | 
| useSystemd | Indicates whether your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Valid values are yes or no\. Run the check\_ggc\_dependencies script in [Module 1](module1.md) to see if your device uses systemd\. | 
|  `managedRespawn`  |  An optional over\-the\-air \(OTA\) updates feature, this indicates that the OTA agent needs to run custom code before an update\.  |  Valid values are `true` or `false`\. For more information, see [OTA Updates of AWS IoT Greengrass Core Software](core-ota-update.md)\.  | 
|  `writeDirectory`  |  The write directory where AWS IoT Greengrass creates all read\-write resources\.  |  This is an optional value\. For more information, see [Configure a Write Directory for AWS IoT Greengrass](#write-directory)\.  | 

------
#### [ GGC v1\.5\.0 ]

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

| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the [AWS IoT root CA](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html) relative to the `/greengrass-root/certs` folder\.  |  Save the file under the `/greengrass-root/certs` folder\.  | 
| certPath |  The path to the AWS IoT Greengrass core certificate relative to the `/greengrass-root/certs` folder\.  | Save the file under the /greengrass\-root/certs folder\. | 
| keyPath | The path to the AWS IoT Greengrass core private key relative to /greengrass\-root/certs folder\. | Save the file under the /greengrass\-root/certs folder\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS IoT Greengrass core\. | Find this for your core in the AWS IoT Greengrass console under Cores, or by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/list-core-definitions.html) command\. | 
| iotHost | Your AWS IoT endpoint\. | Find this in the AWS IoT Core console under Settings, or by running the [https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) command\. | 
| ggHost | Your AWS IoT Greengrass endpoint\. | This value uses the format greengrass\.iot\.region\.amazonaws\.com\. Use the same region as iotHost\. | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default value is 600 seconds\. | 
| useSystemd | Indicates whether your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Values are yes or no\. Use the dependency script in [Module 1](module1.md) to see if your device uses systemd\. | 
|  `managedRespawn`  |  An optional over\-the\-air \(OTA\) updates feature, this indicates that the OTA agent needs to run custom code before an update\.  |  For more information, see [OTA Updates of AWS IoT Greengrass Core Software](core-ota-update.md)\.  | 

------
#### [ GGC v1\.3 ]

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

| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the [AWS IoT root CA](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html) relative to the `/greengrass-root/certs` folder\.  |  Save the file under the `/greengrass-root/certs` folder\.  | 
| certPath |  The path to the AWS IoT Greengrass core certificate relative to the `/greengrass-root/certs` folder\.  | Save the file under the /greengrass\-root/certs folder\. | 
| keyPath | The path to the AWS IoT Greengrass core private key relative to /greengrass\-root/certs folder\. | Save the file under the /greengrass\-root/certs folder\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS IoT Greengrass core\.  | You can find this value in the AWS IoT Greengrass console under the definition for your AWS IoT thing\. | 
| iotHost | Your AWS IoT endpoint\. | You can find this value in the AWS IoT Core console under Settings\. | 
| ggHost | Your AWS IoT Greengrass endpoint\. | You can find this value in the AWS IoT Core console under Settings with greengrass\. prepended\. | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default value is 600 seconds\. | 
| useSystemd | A binary flag, if your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Values are yes or no\. Use the dependency script in [Module 1](module1.md) to see if your device uses systemd\. | 
|  `managedRespawn`  |  An optional over\-the\-air \(OTA\) updates feature, this indicates that the OTA agent needs to run custom code before an update\.  |  For more information, see [OTA Updates of AWS IoT Greengrass Core Software](core-ota-update.md)\.  | 

------
#### [ GGC v1\.1 ]

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

| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the [AWS IoT root CA](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html) relative to the `/greengrass-root/certs` folder\.  |  Save the file under the `/greengrass-root/certs` folder\.  | 
| certPath |  The path to the AWS IoT Greengrass core certificate relative to the `/greengrass-root/certs` folder\.  | Save the file under the /greengrass\-root/certs folder\. | 
| keyPath | The path to the AWS IoT Greengrass core private key relative to the /greengrass\-root/certs folder\. | Save the file under the /greengrass\-root/certs folder\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS IoT Greengrass core\.  | You can find this value in the AWS IoT Greengrass console under the definition for your AWS IoT thing\. | 
| iotHost | Your AWS IoT endpoint\. | You can find this value in the AWS IoT Core console under Settings\. | 
| ggHost | Your AWS IoT Greengrass endpoint\. | You can find this value in the AWS IoT Core console under Settings with greengrass\. prepended\. | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default value is 600 seconds\. | 
| useSystemd | A binary flag, if your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Values are yes or no\. Use the dependency script in [Module 1](module1.md) to see if your device uses systemd\. | 

------
#### [ GGC v1\.0 ]

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

| Field | Description | Notes | 
| --- | --- | --- | 
| caPath |  The path to the [AWS IoT root CA](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html) relative to the `/greengrass-root/configuration/certs` folder\.  |  Save the file under the `/greengrass-root/configuration/certs` folder\.  | 
| certPath |  The path to the AWS IoT Greengrass core certificate relative to the `/greengrass-root/configuration/certs` folder\.  | Save the file under the /greengrass\-root/configuration/certs folder\. | 
| keyPath | The path to the AWS IoT Greengrass core private key relative to the /greengrass\-root/configuration/certs folder\. | Save the file under the /greengrass\-root/configuration/certs folder\. | 
| thingArn | The Amazon Resource Name \(ARN\) of the AWS IoT thing that represents the AWS IoT Greengrass core\.  | You can find this value in the AWS IoT Greengrass console under the definition for your AWS IoT hing\. | 
| iotHost | Your AWS IoT endpoint\. | You can find this value in the AWS IoT Core console under Settings\. | 
| ggHost | Your AWS IoT Greengrass endpoint\. |  You can find this value in the AWS IoT Core console under **Settings** with `greengrass.` prepended\.  | 
| keepAlive | The MQTT KeepAlive period, in seconds\. | This is an optional value\. The default value is 600 seconds\. | 
| useSystemd | A binary flag if your device uses [https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)\. | Values are yes or no\. Use the dependency script in [Module 1](module1.md) to see if your device uses systemd\. | 

------

## Endpoints Must Match the Certificate Type<a name="certificate-endpoints"></a>

Your AWS IoT and AWS IoT Greengrass endpoints must correspond to the certificate type of your root CA\. For example, if you're using an Amazon Trust Services \(ATS\) certificate \(preferred\), the `coreThing.iotHost` and `coreThing.ggHost` properties must include the `ats` segment \(for example, `abcde1234uwxyz-ats.iot.us-west-2.amazonaws.com`\)\.

If you're using a legacy endpoint, either create an ATS endpoint and download an appropriate certificate \(preferred\) or make sure that your endpoints correspond to your certificate\. For more information, see the [Server Authentication in AWS IoT Core](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html) documentation\.

You can find your AWS IoT endpoint in the AWS IoT Core console under **Settings**, or by running the [https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html](https://docs.aws.amazon.com/cli/latest/reference/iot/describe-endpoint.html) CLI command with the appropriate `--endpoint-type` parameter\. For example:
+ To return an ATS signed data endpoint, run `aws iot describe-endpoint --endpoint-type iot:Data-ATS`\.
+ To return a VeriSign signed data endpoint \(legacy\), run `aws iot describe-endpoint --endpoint-type iot:Data`\.

Your AWS IoT Greengrass endpoint is your `iotHost` endpoint with the host prefix replaced by *greengrass* \(for example, `greengrass-ats.iot.region.amazonaws.com`\)\. This uses the same region as `iotHost`\.

If your endpoints and certificate do not match, authentication attempts between AWS IoT and AWS IoT Greengrass fail\.

## Connect on Port 443 or Through a Network Proxy<a name="alpn-network-proxy"></a>

This feature is available for AWS IoT Greengrass Core v1\.7 only\.

AWS IoT Greengrass communicates with AWS IoT using the MQTT messaging protocol with TLS client authentication\. By convention, MQTT over TLS uses port 8883\. However, as a security measure, restrictive environments might limit inbound and outbound traffic to a small range of TCP ports\. For example, a corporate firewall might open port 443 for HTTPS traffic, but close other ports that are used for less common protocols, such as port 8883 for MQTT traffic\. Other restrictive environments might require all traffic to go through an HTTP proxy before connecting to the internet\.

**Note**  
AWS IoT Greengrass sends HTTPS traffic over port 8443, and requires port 8443 to remain open even if MQTT traffic is configured to send over port 443 instead of the default port 8883\.

To enable communication in these scenarios, AWS IoT Greengrass allows the following configurations:
+ **MQTT with TLS client authentication on port 443**\. If your network allows connections to port 443, you can configure the core to use port 443 for MQTT traffic instead of the default port 8883\. This can be a direct connection to port 443 or a connection through a network proxy server\.

  AWS IoT Greengrass uses the [ Application Layer Protocol Network](https://tools.ietf.org/html/rfc7301) \(ALPN\) TLS extension to enable this connection\. As with the default configuration, MQTT over TLS on port 443 uses certificate\-based client authentication\.
+ **Connection through a network proxy**\. You can configure a network proxy server to act as an intermediary for connecting to the AWS IoT Greengrass core\. Only basic authentication and HTTP and HTTPS proxies are supported\.

  The proxy configuration is passed to user\-defined Lambda functions through the `http_proxy`, `https_proxy`, and `no_proxy` environment variables\. If a function specifies these variables, AWS IoT Greengrass doesn't override them\.

**To Configure MQTT on Port 443**

This procedure allows the core to use port 443 for MQTT messaging\. When using this configuration, AWS IoT Greengrass specifies the `x-amzn-mqtt-ca` header in the TLS handshake, which is expected by AWS IoT\. For more information, see [Protocols](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html) in the *AWS IoT Developer Guide*\.

1. Run the following command to stop the AWS IoT Greengrass daemon:

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

 

**To Configure a Network Proxy**

This procedure allows AWS IoT Greengrass to connect to the internet through an HTTP or HTTPS network proxy\.

1. Run the following command to stop the AWS IoT Greengrass daemon:

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd stop
   ```

1. Open `greengrass-root/config/config.json` for editing as the su user\.

1. In the `coreThing` object, add the `networkProxy` object, as shown in the following example\.

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

Use the `networkProxy` object to specify information about the network proxy\. This object has the following properties\.


**networkProxy object**  

| Field | Description | 
| --- | --- | 
| noProxyAddresses |  Optional\. A comma\-separated list of IP addresses or host names that are exempt from the proxy\.  | 
| proxy |  The proxy to connect to\. A proxy has the following properties\. [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html)  | 

## Configure a Write Directory for AWS IoT Greengrass<a name="write-directory"></a>

This feature is available for AWS IoT Greengrass Core v1\.6 and later\.

By default, the AWS IoT Greengrass core software is deployed under a single root directory where AWS IoT Greengrass performs all read and write operations\. However, you can configure AWS IoT Greengrass to use a separate directory for all write operations, including creating directories and files\. In this case, AWS IoT Greengrass uses two top\-level directories:
+ The *greengrass\-root* directory, which you can leave as read\-write or optionally make read\-only\. This contains the AWS IoT Greengrass core software and other critical components that should remain immutable during runtime, such as certificates and `config.json`\.
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
|                   |-- nodejs6.10/
|                   |-- python2.7/
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

 

**To Configure a Write Directory**

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

**To Make the Greengrass Root Directory Read\-Only**

1. Grant required access to the AWS IoT Greengrass:
**Note**  
This step is required only if you want to make the Greengrass root directory read\-only\. The write directory must be configured before you begin this procedure\.

   1. Give read and write permissions to the `config.json` owner\.

      ```
      sudo chmod 0600 /greengrass-root/config/config.json
      ```

   1. Make ggc\_user the owner of the certs and system Lambda directories\.

      ```
      sudo chown -R ggc_user:ggc_group /greengrass-root/certs/
      sudo chown -R ggc_user:ggc_group /greengrass-root/ggc/packages/1.7.1/lambda/
      ```

1. Make the *greengrass\-root* directory read\-only by using your preferred mechanism\.
**Note**  
One way to make the *greengrass\-root* directory read\-only is to mount the directory as read\-only\. However, to apply over\-the\-air \(OTA\) updates to core software in a mounted directory, the directory must first be unmounted, and then remounted after the update\. You can add these `umount` and `mount` operations to the `ota_pre_update` and `ota_post_update` scripts\. For more information about OTA updates, see [Greengrass OTA Agent](core-ota-update.md#ota-agent) and [AWS IoT Greengrass Core Update with Managed Respawn](core-ota-update.md#managed-respawn-ggc)\.

1. Start the daemon\.

   ```
   cd /greengrass-root/ggc/core/
   sudo ./greengrassd start
   ```

   If the permissions from step 1 aren't set correctly, then the daemon won't start\.

## MQTT Message Queue<a name="mqtt-message-queue"></a>

MQTT messages that are destined for cloud targets are queued to await processing\. Queued messages are processed in first in first out \(FIFO\) order\. After a message is processed and published to the cloud, the message is removed from the queue\. AWS IoT Greengrass manages the queue by using a system\-defined `GGCloudSpooler` Lambda function\.

### Configure the MQTT Message Queue<a name="configure-message-queue"></a>

This feature is available for AWS IoT Greengrass Core v1\.6 and later\.

You can configure AWS IoT Greengrass to store unprocessed messages in memory or in a local storage cache\. Unlike in\-memory storage, the local storage cache has the ability to persist across core restarts \(for example, after a group deployment or a device reboot\), so AWS IoT Greengrass can continue to process the messages\. You can also configure the storage size\.

**Note**  
Versions 1\.5\.0 and earlier use in\-memory storage with a queue size of 2\.5 MB\. You cannot configure storage settings for earlier versions\.

The following environment variables for the `GGCloudSpooler` Lambda function are used to define storage settings\.
+ **GG\_CONFIG\_STORAGE\_TYPE**\. The location of the message queue\. The following are valid values:
  + `FileSystem`\. Store unprocessed messages in the local storage cache\. When the core restarts, queued messages are retained for processing\. Messages are removed after they are processed\.
  + `Memory` \(default\)\. Store unprocessed messages in memory\. When the core restarts, queued messages are lost\.

    This option is optimized for devices with restricted hardware capabilities\. When using this configuration, we recommend that you deploy groups or restart the device when the service disruption is the lowest\.
+ **GG\_CONFIG\_MAX\_SIZE\_BYTES**\. The storage size, in bytes\. This value can be any non\-negative integer **greater than or equal to 262144** \(256 KB\); a smaller size prevents the AWS IoT Greengrass core software from starting\. The default size is 2\.5 MB\. When the size limit is reached, the oldest queued messages are replaced by new messages\.

#### To Cache Messages in Local Storage<a name="configure-local-storage-cache"></a>

To configure AWS IoT Greengrass to cache messages to the file system so they persist across core restarts, you create a function definition version where the `GGCloudSpooler` function specifies the `FileSystem` storage type\. You must use the AWS IoT Greengrass API to configure the local storage cache\. You can't do this in the console\.

The following procedure uses the [ `create-function-definition-version`](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition-version.html) CLI command to configure the spooler to save queued messages to the file system\. It also configures a 2\.6 MB queue size\.

**Note**  
This procedure assumes that you're updating the configuration of the latest group version of an existing group\.

1. <a name="get-group-id-latestversion"></a>Get the IDs of the target Greengrass group and group version\.

   ```
   aws greengrass list-groups
   ```
**Note**  
 The list\-groups command only returns 50 groups at a time\. If you have more than 50 groups, specify the max\-results option with the desired number of results \(such as **100**\) to find your group:   

   ```
   aws greengrass list-groups --max-results 100
   ```

1. <a name="copy-group-id-latestversion"></a>Copy the `Id` and `LatestVersion` properties of your target group from the output\.

1. <a name="get-latest-group-version"></a>Get the latest group version\.
   + Replace *group\-id* with the `Id` that you copied\.
   + Replace *latest\-group\-version\-id* with the `LatestVersion` that you copied\.

   ```
   aws greengrass get-group-version \
   --group-id group-id \
   --group-version-id latest-group-version-id
   ```

1. <a name="copy-group-component-arns-except-function"></a>From the `Definition` object in the output, copy the `CoreDefinitionVersionArn` and the ARNs of all other group components except `FunctionDefinitionVersionArn`\. You use these values when you create a new group version\.

1. <a name="parse-function-def-id"></a>From the `FunctionDefinitionVersionArn` in the output, copy the ID of the function definition\. The ID is the GUID that follows the `functions` segment in the ARN\.
**Note**  
You can optionally create a function definition by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html) command, and then copy the ID from the output\.

1. Add a function definition version to the function definition\.
   + Replace *function\-definition\-id* with the `Id` that you copied for the function definition\.
   + Replace *arbitrary\-function\-id* with a name for the function, such as **spooler\-function**\.
   + Add any Lambda functions that you want to include in this version to the `functions` array\. You can use the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/get-function-definition-version.html) command to get the Greengrass Lambda functions from an existing function definition version\.
**Warning**  
Make sure that you specify a value for `GG_CONFIG_MAX_SIZE_BYTES` that's **greater than or equal to 262144**\. A smaller size prevents the AWS IoT Greengrass core software from starting\.

   ```
   aws greengrass create-function-definition-version \
   --function-definition-id function-definition-id \
   --functions '[{"FunctionArn": "arn:aws:lambda:::function:GGCloudSpooler:1","FunctionConfiguration": {"Environment": {"Variables":{"GG_CONFIG_MAX_SIZE_BYTES":"2621440","GG_CONFIG_STORAGE_TYPE":"FileSystem"}},"Executable": "spooler","MemorySize": 32768,"Pinned": true,"Timeout": 3},"Id": "arbitrary-function-id"}]'
   ```

1. <a name="copy-function-def-version-arn"></a>Copy the `Arn` of the function definition version from the output\.

1. <a name="create-group-version-with-sys-lambda"></a>Create a group version that contains the system\-defined Lambda function\.
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

1. <a name="create-group-deployment"></a>Deploy the group\.
   + Replace *group\-id* with the `Id` that you copied for the group\.
   + Replace *group\-version\-id* with the `Version` that you copied for the new group version\.

   ```
   aws greengrass create-deployment \
   --group-id group-id \
   --group-version-id group-version-id \
   --deployment-type NewDeployment
   ```

 To update the storage settings, you use the AWS IoT Greengrass API to create a new function definition version that contains the `GGCloudSpooler` function with the updated configuration\. Then add the function definition version to a new group version \(along with your other group components\) and deploy the group version\. If you want to restore the default configuration, you can create a function definition version that doesn't include the `GGCloudSpooler` function\. 

 This system\-defined Lambda function isn't visible in the console\. However, after the function is added to the latest group version, it's included in deployments that you make from the console \(unless you use the API to replace or remove it\)\. 

## Activate Automatic IP Detection<a name="ip-auto-detect"></a>

 You can configure AWS IoT Greengrass to enable automatic discovery of your AWS IoT Greengrass core using the `IPDetector` system Lambda function\. This feature can also be enabled by choosing **Automatic detection** when deploying your group from the console for the first time, or from the group settings page in the console at any time\. 

 The following procedure uses the [ `create-function-definition-version`](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition-version.html) CLI command to configure automatic discovery of the Greengrass core\. 

**Note**  
 This procedure assumes that you're updating the configuration of the latest group version of an existing group\. 

1. <a name="get-group-id-latestversion"></a>Get the IDs of the target Greengrass group and group version\.

   ```
   aws greengrass list-groups
   ```
**Note**  
 The list\-groups command only returns 50 groups at a time\. If you have more than 50 groups, specify the max\-results option with the desired number of results \(such as **100**\) to find your group:   

   ```
   aws greengrass list-groups --max-results 100
   ```

1. <a name="copy-group-id-latestversion"></a>Copy the `Id` and `LatestVersion` properties of your target group from the output\.

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
You can optionally create a function definition by running the [https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html](https://docs.aws.amazon.com/cli/latest/reference/greengrass/create-function-definition.html) command, and then copy the ID from the output\.

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

1. <a name="create-group-version-with-sys-lambda"></a>Create a group version that contains the system\-defined Lambda function\.
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

1. <a name="create-group-deployment"></a>Deploy the group\.
   + Replace *group\-id* with the `Id` that you copied for the group\.
   + Replace *group\-version\-id* with the `Version` that you copied for the new group version\.

   ```
   aws greengrass create-deployment \
   --group-id group-id \
   --group-version-id group-version-id \
   --deployment-type NewDeployment
   ```

 If you want to manually input the IP address of your AWS IoT Greengrass core, you can complete this tutorial with a different function definition that does not include the `IPDetector` function\. This will prevent the detection function from locating and automatically inputting your AWS IoT Greengrass core IP address\.  

 This system\-defined Lambda function isn't visible in the Lambda console\. After the function is added to the latest group version, it's included in deployments that you make from the console \(unless you use the API to replace or remove it\)\. 

## Configure the Init System to Start the Greengrass Daemon<a name="start-on-boot"></a>

It's a good practice to set up your init system to start the Greengrass daemon during boot, especially when managing large fleets of devices\.

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

## See Also<a name="cores-see-also"></a>
+ [Hardware Security Integration](hardware-security.md)