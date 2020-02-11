# Prerequisites<a name="dev-tst-prereqs"></a>

This section describes the prerequisites for using IDT for AWS IoT Greengrass\.

## Download the Latest Version of AWS IoT Device Tester for AWS IoT Greengrass<a name="install-dev-tst-gg"></a>

Download the latest version of IDT from [Supported Versions of AWS IoT Device Tester for AWS IoT Greengrass](dev-test-versions.md)\. Extract the software into a location on your file system where you have read and write permissions\. 

Windows has a path length limitation of 260 characters\. If you are using Windows, extract IDT for AWS IoT Greengrass into a root directory like `C:\ ` or `D:\` to keep your paths under the 260 character limit\.

## Create and Configure an AWS Account<a name="config-aws-account"></a>

Follow these steps to create and configure an AWS account, create an IAM user, and create an IAM policy that grants IDT for AWS IoT Greengrass permission to access resources on your behalf while running tests\.

1. [Create an AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)\.

1. [Create an IAM policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_managed-policies.html) that grants IDT for AWS IoT Greengrass the permissions it needs to run the tests and collect IDT usage data\. Use the policy templates described in [Permissions Policy Template](#policy-template)\.

1. [Create an IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) in your AWS account and attach the policy you just created\.

### Permissions Policy Template<a name="policy-template"></a>

The following is a policy template that grants the permissions required for IDT for AWS IoT Greengrass to run tests\.

**Important**  
The following policy template grants permission to create roles, create policies, and attach policies to roles\. IDT for AWS IoT Greengrass uses these permissions for tests that need to create roles\. Although the policy template does not provide administrator privileges to the user, the permissions can be potentially used to gain administrator access to your AWS account\.

------
#### [ IDT v2\.0\.0 and later ]

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:DeleteRole",
                "iam:AttachRolePolicy",
                "iam:PassRole",
                "iam:DetachRolePolicy"
            ],
            "Resource": [
                "arn:aws:iam::*:role/idt-*",
                "arn:aws:iam::*:role/GreengrassServiceRole"
            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "lambda:DeleteFunction",
                "lambda:CreateFunction"
            ],
            "Resource": ["arn:aws:lambda:*:*:function:idt-*"]
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": [
                "iot:AttachPolicy",
                "iot:DetachPolicy",
                "iot:DeletePolicy"
            ],
            "Resource": [
                "arn:aws:iot:*:*:policy/idt-*",
                "arn:aws:iot:*:*:cert/*"
            ]
        },
        {
            "Sid": "VisualEditor3",
            "Effect": "Allow",
            "Action": [
                "iot:AttachThingPrincipal",
                "iot:DeleteThing",
                "iot:DetachThingPrincipal",
                "iot:CreateThing"
            ],
            "Resource": [
                "arn:aws:iot:*:*:thing/idt-*",
                "arn:aws:iot:*:*:cert/*"
            ]
        },
        {
            "Sid": "VisualEditor4",
            "Effect": "Allow",
            "Action": [
                "iot:DeleteCertificate",
                "iot:UpdateCertificate"
            ],
            "Resource": ["arn:aws:iot:*:*:cert/*"]
        },
        {
            "Sid": "VisualEditor5",
            "Effect": "Allow",
            "Action": [
                "iot:CreateJob",
                "iot:DescribeJob",
                "iot:DescribeJobExecution",
                "iot:DeleteJob"
            ],
            "Resource": [
                "arn:aws:iot:*:*:job/*",
                "arn:aws:iot:*:*:thing/idt-*"
            ]
        },
        {
            "Sid": "VisualEditor6",
            "Effect": "Allow",
            "Action": [
                "iot:CreatePolicy",
                "iot:CreateKeysAndCertificate",
                "iot:UpdateThingShadow",
                "iot:GetThingShadow",
                "iot:DescribeEndpoint",
                "greengrass:*",
                "iam:ListAttachedRolePolicies",
                "iot:CreateCertificateFromCsr",
                "iot:ListThings"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor7",
            "Effect": "Allow",
            "Action": ["execute-api:Invoke"],
            "Resource": "arn:aws:execute-api:*:098862408343:*"
        }
    ]
}
```

------
#### [ IDT v1\.3\.3 and earlier ]

```
{
	"Version": "2012-10-17",
	"Statement": [
	{
		"Sid": "VisualEditor0",
		"Effect": "Allow",
		"Action": [
			"iam:CreateRole",
			"iam:DeleteRole",
			"iam:AttachRolePolicy",
			"iam:PassRole",
			"iam:DetachRolePolicy"
		],
		"Resource": [
			"arn:aws:iam::*:role/idt-*",
			"arn:aws:iam::*:role/GreengrassServiceRole"
		]
	},
	{
		"Sid": "VisualEditor1",
		"Effect": "Allow",
		"Action": [
			"lambda:DeleteFunction",
			"lambda:CreateFunction"
		],
		"Resource": [
			"arn:aws:lambda:*:*:function:idt-*"
		]
	},
	{
		"Sid": "VisualEditor2",
		"Effect": "Allow",
		"Action": [
			"iot:DeleteCertificate",
			"iot:AttachPolicy",
			"iot:DetachPolicy",
			"iot:DeletePolicy",
			"iot:CreateJob",
			"iot:AttachThingPrincipal",
			"iot:DeleteThing",
			"iot:DescribeJob",
			"iot:UpdateCertificate",
			"iot:DescribeJobExecution",
			"iot:DetachThingPrincipal",
			"iot:CreateThing",
			"iot:DeleteJob"
		],
		"Resource": [
			"arn:aws:iot:*:*:thing/idt-*",
			"arn:aws:iot:*:*:policy/idt-*",
			"arn:aws:iot:*:*:cert/*",
			"arn:aws:iot:*:*:job/*"
		]
	},
	{
		"Sid": "VisualEditor3",
		"Effect": "Allow",
		"Action": [
			"iot:CreatePolicy",
			"iot:CreateKeysAndCertificate",
			"iot:UpdateThingShadow",
			"iot:GetThingShadow",
			"iot:DescribeEndpoint",
			"greengrass:*",
			"iam:ListAttachedRolePolicies"
		],
		"Resource": "*"
	}
	]
}
```

------