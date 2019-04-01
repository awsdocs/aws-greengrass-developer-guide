# Permissions Policy Template<a name="policy-template"></a>

The following is a policy template that grants the permissions required for AWS IoT Device Tester:

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
                "arn:aws:iam::*:role/idt-*"
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