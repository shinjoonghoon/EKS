# EKS using SSO
* get-caller-identity
```
aws sts get-caller-identity
```

* AWS Organizations/management account
* IAM Identity Center(SSO)/Permission sets/EKSClusterAdminAccess/inline policy
>EKSClusterAdminAccess
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
* AWS Organizations/OU account
* IAM/Identity providers/
>AWSSO_xxxxxx_DO_NOT_DELETE
* IAM/Roles/
>AWSReservedSSO_EKSClusterAdminAccess_xxxxxx
* Trusted entities
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<accountid>:saml-provider/AWSSO_xxxxxx_DO_NOT_DELETE"
            },
            "Action": [
                "sts:AssumeRoleWithSAML",
                "sts:TagSession"
            ],
            "Condition": {
                "StringEquals": {
                    "SAML:aud": "https://signin.aws.amazon.com/saml"
                }
            }
        }
    ]
}
```
* AwsSSOInlinePolicy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

* IAM/Roles/EKSClusterCreator
>EKSClusterCreator
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::xxxxxxxxxxxx:role/aws-reserved/sso.amazonaws.com/ap-northeast-2/AWSReservedSSO_EKSClusterAdminAccess_xxxxxx"
                ]
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
* IAM/Roles/EKSClusterCreator/EKSAdminPolicy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "eks:*",
                "iam:*",
                "cloudformation:*",
                "ec2:*",
                "vpc:*",
                "autoscaling:*",
                "ssm:*",
                "kms:*",
                "s3:*"
            ],
            "Resource": "*"
        }
    ]
}
```
* sso login
```
aws configure sso
```
```
SSO session name (Recommended): sso1
SSO start URL [None]: https://d-xxxxxx.awsapps.com/start
SSO region [None]: ap-northeast-2
SSO registration scopes [sso:account:access]: sso:account:access
Attempting to automatically open the SSO authorization page in your default browser.
If the browser does not open or you wish to use a different device to authorize this request, open the following URL:

https://device.sso.ap-northeast-2.amazonaws.com/

Then enter the code:

XXXX-XXXX
The only AWS account available to you is: xxxxxxxxxxxx
Using the account ID xxxxxxxxxxxx
The only role available to you is: EKSClusterAdminAccess
Using the role name "EKSClusterAdminAccess"
CLI default client Region [ap-northeast-2]:
CLI default output format [None]: json
CLI profile name [EKSClusterAdminAccess-xxxxxxxxxxxx]: eks-admin

To use this profile, specify the profile name using --profile, as shown:

aws s3 ls --profile eks-admin
An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied
```
* assume-role
```
aws sts get-caller-identity
```
```
aws sts get-caller-identity --query Arn --output text --profile eks-admin
```
```
account=$(aws sts get-caller-identity --query Account --output text --profile eks-admin)
rolearn="arn:aws:iam::${account}:role/EKSClusterCreator"
echo $account
echo $rolearn
```
```
creds=$(aws sts assume-role --role-arn ${rolearn} \
   --role-session-name "create-eks-cluster" \
   --query 'Credentials.[{AWS_ACCESS_KEY_ID: AccessKeyId}, {AWS_SESSION_TOKEN: SessionToken}, {AWS_SECRET_ACCESS_KEY: SecretAccessKey}]' \
   --profile eks-admin | jq -r '.[]' | sed 's/{//;s/}//;s/"//g;s/ AWS_ACCESS_KEY_ID: /export AWS_ACCESS_KEY_ID=/;s/ AWS_SESSION_TOKEN: /export AWS_SESSION_TOKEN=/;s/ AWS_SECRET_ACCESS_KEY: /export AWS_SECRET_ACCESS_KEY=/')
echo $creds
```
```
eval $creds
```
```
aws sts get-caller-identity
```
```
{
    "UserId": "ARxxxxxxxxxxxxxxxxxxx:create-eks-cluster",
    "Account": "xxxxxxxxxxxx",
    "Arn": "arn:aws:sts::xxxxxxxxxxxx:assumed-role/EKSClusterCreator/create-eks-cluster"
}

```
---

* (Optional)
* AWS_PROFILE
```
export AWS_PROFILE=k8sAdmin
echo ${AWS_PROFILE}
unset AWS_PROFILE

```
* re-login
```
account=$(aws sts get-caller-identity --query Account --output text --profile EKSClusterAdminAccess-xxxxxxxxxxxx)
rolearn="arn:aws:iam::${account}:role/EKSClusterCreator"
echo $account
echo $rolearn

creds=$(aws sts assume-role --role-arn ${rolearn} \
   --role-session-name "create-eks-cluster" \
   --query 'Credentials.[{AWS_ACCESS_KEY_ID: AccessKeyId}, {AWS_SESSION_TOKEN: SessionToken}, {AWS_SECRET_ACCESS_KEY: SecretAccessKey}]' \
   --profile EKSClusterAdminAccess-xxxxxxxxxxxx | jq -r '.[]' | sed 's/{//;s/}//;s/"//g;s/ AWS_ACCESS_KEY_ID: /export AWS_ACCESS_KEY_ID=/;s/ AWS_SESSION_TOKEN: /export AWS_SESSION_TOKEN=/;s/ AWS_SECRET_ACCESS_KEY: /export AWS_SECRET_ACCESS_KEY=/')
echo $creds

eval $creds

aws sts get-caller-identity

```

* unset
* https://repost.aws/ko/knowledge-center/iam-assume-role-cli
```
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN

```
