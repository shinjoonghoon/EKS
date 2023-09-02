# EKS using SSO
* https://aws.amazon.com/blogs/containers/a-quick-path-to-amazon-eks-single-sign-on-using-aws-sso/

# AWS Organizations - Management account
* SSO Group: EKSAdmins
* SSO User: eksadmin
* IAM Identity Center(SSO)/Permission sets/EKSAdminAccess/inline policy
>Permission set name: `EKSAdminAccess`
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
# AWS Organizations - OU account
* IAM/Identity providers - SAML
>AWSSO_xxxxxx_DO_NOT_DELETE
* IAM/Roles/
>AWSReservedSSO_EKSAdminAccess_xxxxxx
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

# Create IAM EKSClusterCreator role
>When you create an EKS cluster, the user or role you use will inherit cluster administrator permissions. This identity principal retains cluster admin permissions for the life of the cluster and should be kept secure. It is best practice to create additional roles to delegate routine operational tasks while keeping the cluster creator role separate and secure. If you forget the name of that principal, it can be retrieved or changed by AWS Support for a limited time, or you can recreate the IAM principal if you remember the user or role name.
* IAM/Roles/EKSClusterCreator
>`EKSClusterCreator`
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
>`EKSClusterCreatorPolicy`
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
# SSO Login
```
aws configure sso
```
```
SSO session name (Recommended): eksadmin
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
The only role available to you is: EKSAdminAccess
Using the role name "EKSAdminAccess"
CLI default client Region [None]: ap-northeast-2
CLI default output format [None]: json
CLI profile name [EKSAdminAccess-xxxxxxxxxxxx]: eksadmin

To use this profile, specify the profile name using --profile, as shown:

aws s3 ls --profile eksadmin
An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied
```
# Assume role: EKSClusterCreator
```
aws sts get-caller-identity
```
```
aws sts get-caller-identity --query Arn --output text --profile eksadmin
```
```
account=$(aws sts get-caller-identity --query Account --output text --profile eksadmin)
rolearn="arn:aws:iam::${account}:role/EKSClusterCreator"
echo $account
echo $rolearn
```
```
creds=$(aws sts assume-role --role-arn ${rolearn} \
   --role-session-name "eks-cluster-creator" \
   --query 'Credentials.[{AWS_ACCESS_KEY_ID: AccessKeyId}, {AWS_SESSION_TOKEN: SessionToken}, {AWS_SECRET_ACCESS_KEY: SecretAccessKey}]' \
   --profile eksadmin | jq -r '.[]' | sed 's/{//;s/}//;s/"//g;s/ AWS_ACCESS_KEY_ID: /export AWS_ACCESS_KEY_ID=/;s/ AWS_SESSION_TOKEN: /export AWS_SESSION_TOKEN=/;s/ AWS_SECRET_ACCESS_KEY: /export AWS_SECRET_ACCESS_KEY=/')
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
    "UserId": "AROAxxxxxxxxxxxxxxxxx:eks-cluster-creator",
    "Account": "xxxxxxxxxxxx",
    "Arn": "arn:aws:sts::xxxxxxxxxxxx:assumed-role/EKSClusterCreator/eks-cluster-creator"
}

```
