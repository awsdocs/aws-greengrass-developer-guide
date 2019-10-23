# Permissions Policy Template<a name="policy-template"></a>

The following is a policy template that grants the permissions required for IDT for AWS IoT Greengrass to run tests\.

**Important**  
The following policy template grants permission to create roles, create policies, and attach policies to roles\. IDT for AWS IoT Greengrass uses these permissions for tests that need to create roles\. Although the policy template does not provide administrator privileges to the user, the permissions can be potentially used to gain administrator access to your AWS account\.

------
#### [ IDT v2\.0\.0 ]

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