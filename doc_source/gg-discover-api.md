# Greengrass Discovery RESTful API<a name="gg-discover-api"></a>

All devices that communicate with an AWS Greengrass core must be a member of a Greengrass group\. Each group must have an AWS Greengrass core\. The Discovery API enables devices to retrieve information required to connect to an AWS Greengrass core that is in the same Greengrass group as the device\. When a device first comes online, it can connect to the AWS Greengrass cloud service and use the Discovery API to find:
+ The group to which it belongs\.
+ The IP address and port for the AWS Greengrass core in the group\.
+ The group's root CA certificate, which can be used to authenticate the AWS Greengrass core device\.

To use this API, send HTTP requests to the following URI:

```
https://your-aws-endpoint/greengrass/discover/thing/thing-name
```

The endpoint is specific to your AWS account\. To retrieve your endpoint, use the `aws iot describe-endpoint` CLI command:

```
$ aws iot describe-endpoint
{
    "endpointAddress": "a1b2c3d4e5f6g7.iot.us-west-2.amazonaws.com"
}
```

Use port `8443` when connecting\. For a list of region\-specific endpoints, see [AWS IoT Regions and Endpoints](http://docs.aws.amazon.com/general/latest/gr/rande.html#iot_region) in the *AWS General Reference*\. This is a data plane only API\. The endpoints used for working with rules, certificates, and policies do not support the Discovery API\.

## Request<a name="gg-discover-request"></a>

The request contains the standard HTTP headers and is sent to the following URI:

```
HTTP GET https://your-aws-endpoint/greengrass/discover/thing/thingName
```

## Response<a name="gg-discover-response"></a>

*Response*

Upon success, the response includes the standard HTTP headers plus the following code and body:

```
HTTP 200
BODY: response document
```

For more information see, [Example Discover Response Documents](#gg-discover-response-doc)\.

## Authorization<a name="gg-discover-auth"></a>

Retrieving the connectivity information requires a policy that allows the caller to perform the `greengrass:Discover` action\. TLS mutual authentication with a client certificate is the only accepted form of authentication\. The following is an example policy that allows a caller to perform this action:

```
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": "greengrass:Discover",
        "Resource": ["arn:aws:iot:aws-region:aws-account:thing/thing-name"]
     }]
}
```

## Example Discover Response Documents<a name="gg-discover-response-doc"></a>

The following document shows the response for a device that is a member of a group with one AWS Greengrass core, one endpoint, and one group CA:

```
{
  "GGGroups": [
    {
      "GGGroupId": "gg-group-01-id",
      "Cores": [
        {
          "thingArn": "core-01-thing-arn",
          "Connectivity": [
            {
              "id": "core-01-connection-id",
              "hostAddress": "core-01-address",
              "portNumber": core-01-port,
              "metadata": "core-01-description"
            }
          ]
        }
      ],
      "CAs": [
        "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----"
      ]
    }
  ]
}
```

The following document shows the response for a device that is a member of two groups with one AWS Greengrass core, multiple endpoints, and multiple group CAs:

```
{
  "GGGroups": [
    {
      "GGGroupId": "gg-group-01-id",
      "Cores": [
        {
          "thingArn": "core-01-thing-arn",
          "Connectivity": [
            {
              "id": "core-01-connection-id",
              "hostAddress": "core-01-address",
              "portNumber": core-01-port,
              "metadata": "core-01-connection-1-description"
            },
            {
              "id": "core-01-connection-id-2",
              "hostAddress": "core-01-address-2",
              "portNumber": core-01-port-2,
              "metadata": "core-01-connection-2-description"
            }
          ]
        }
      ],
      "CAs": [
        "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----",
        "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----",
        "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----"
      ]
    },
    {
      "GGGroupId": "gg-group-02-id",
      "Cores": [
         {
            "thingArn":"core-02-thing-arn",
            "Connectivity" : [
            {
              "id": "core-02-connection-id",
              "hostAddress": "core-02-address",
              "portNumber": core-02-port,
              "metadata": "core-02-connection-1-description"
            }
            ],
            "CAs": [
                "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----",
                "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----",
                "-----BEGIN CERTIFICATE-----cert-contents-----END CERTIFICATE-----"
            ]
        }
    ]
    }
}
```

**Note**  
An AWS Greengrass group must define exactly one AWS Greengrass core\. Any response from the AWS Greengrass cloud service that contains a list of AWS Greengrass cores only contains one AWS Greengrass core\. 