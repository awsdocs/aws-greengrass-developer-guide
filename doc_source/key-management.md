# Key management for the Greengrass core device<a name="key-management"></a>

It's the responsibility of the customer to guarantee secure storage of cryptographic \(public and private\) keys on the Greengrass core device\. AWS IoT Greengrass uses public and private keys for the following scenarios:
+ The IoT client key is used with the IoT certificate to authenticate the Transport Layer Security \(TLS\) handshake when a Greengrass core connects to AWS IoT Core\. For more information, see [Device authentication and authorization for AWS IoT Greengrass](device-auth.md)\.
**Note**  
The key and certificate are also referred to as the core private key and the core device certificate\.
+ The MQTT server key is used the MQTT server certificate to authenticate TLS connections between core and connected devices\. For more information, see [Device authentication and authorization for AWS IoT Greengrass](device-auth.md)\.
+ The local secrets manager also uses the IoT client key to protect the data key used to encrypt local secrets, but you can provide your own private key\. For more information, see [Secrets encryption](secrets.md#secrets-encryption)\.

A Greengrass core supports private key storage using file system permissions, [hardware security modules](hardware-security.md), or both\. If you use file system\-based private keys, you are responsible for their secure storage on the core device\.

On a Greengrass core, the location of your private keys are specified in the `crypto` section of the `config.json` file\. If you configure the core to use a customer\-provided key for the MQTT server certificate, it is your responsibility to rotate the key\. For more information, see [AWS IoT Greengrass core security principals](gg-sec.md#gg-principals)\.

For connected devices, it's your responsibility to keep the TLS stack up to date and protect private keys\. Private keys are used with device certificates to authenticate TLS connections with the AWS IoT Greengrass service\.