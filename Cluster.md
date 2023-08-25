# Security group of cluster
* https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html

```
vpcid=vpc-0e5dcdc77bf5fcc86
region=$(aws configure get region  )
echo $vpcid
echo $region

```
* create-security-group
```
aws ec2 create-security-group --description "Security group of Payments Cluster" --group-name "eks-cluster-sg-payments" --vpc-id $vpcid --output json | jq '.[]'

```

* authorize-security-group-ingress
```
aws ec2 authorize-security-group-ingress \
    --group-id $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text  ) \
    --protocol -1 --port -1 \
    --source-group $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text  ) \
    --output json | jq '.[]'

```
* authorize-security-group-ingress
```
aws ec2 authorize-security-group-ingress \
    --group-id $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text  ) \
    --protocol tcp \
    --port 443 \
    --cidr 10.0.0.0/8 \
    --output json | jq '.[]'

```

# Creating an Amazon EKS cluster
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-cluster.html
* describe-security-groups
```
aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text
```

* describe-subnets
```
aws ec2 describe-subnets --filters "Name=vpc-id, Values=$vpcid"   --query "Subnets[*].{id:SubnetId,az:AvailabilityZone,subnet:Tags[?Key=='Name']|[0].Value}" --output text
```

* ClusterConfig
* https://eksctl.io/usage/schema/
```
cat << EOF > cluster-payments.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: payments
  region: ap-northeast-2
  version: "1.27"
privateCluster:
  enabled: true
  skipEndpointCreation: true
vpc:
#eks-cluster-sg-[Cluster Name]
  securityGroup: sg-0eb70e4b38716fba7
  subnets:
    private:
      EKS-subnet-private2-ap-northeast-2a:
        id: "subnet-00936ff6a3197cfaf"
      EKS-subnet-private5-ap-northeast-2c:
        id: "subnet-0be3341072b0df062"
EOF

```

* get-caller-identity
```
aws sts get-caller-identity
```

* AWS Organizations
* IAM Identity Center(SSO)/Permission sets/EKSClusterAdminAccess/inline policy
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

* IAM/Identity providers/AWSSO_xxxxxx_DO_NOT_DELETE
* IAM/Roles/AWSReservedSSO_EKSClusterAdminAccess_xxxxxx
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

```
aws configure sso
```
```
SSO session name (Recommended): sso-eks-admin
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
CLI profile name [EKSClusterAdminAccess-xxxxxxxxxxxx]:

To use this profile, specify the profile name using --profile, as shown:

aws s3 ls --profile EKSClusterAdminAccess-xxxxxxxxxxxx
An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied
```
```
aws sts get-caller-identity
```
```
aws sts get-caller-identity --query Arn --output text --profile EKSClusterAdminAccess-xxxxxxxxxxxx
```
```
account=$(aws sts get-caller-identity --query Account --output text --profile EKSClusterAdminAccess-xxxxxxxxxxxx)
rolearn="arn:aws:iam::${account}:role/EKSClusterCreator"
```
```
creds=$(aws sts assume-role --role-arn ${rolearn} \
   --role-session-name "create-eks-cluster" \
   --query 'Credentials.[{AWS_ACCESS_KEY_ID: AccessKeyId}, {AWS_SESSION_TOKEN: SessionToken}, {AWS_SECRET_ACCESS_KEY: SecretAccessKey}]' \
   --profile EKSClusterAdminAccess-xxxxxxxxxxxx | jq -r '.[]' | sed 's/{//;s/}//;s/"//g;s/ AWS_ACCESS_KEY_ID: /export AWS_ACCESS_KEY_ID=/;s/ AWS_SESSION_TOKEN: /export AWS_SESSION_TOKEN=/;s/ AWS_SECRET_ACCESS_KEY: /export AWS_SECRET_ACCESS_KEY=/')
echo $creds
eval ${creds}
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
* (Optional) AWS_PROFILE
```
export AWS_PROFILE=k8sAdmin
echo ${AWS_PROFILE}
unset AWS_PROFILE

```

* dry-run
* https://eksctl.io/usage/dry-run/
```
eksctl create cluster -f cluster-payments.yaml --dry-run
```

* create cluster
```
eksctl create cluster -f cluster-payments.yaml
```

* kube/config
```
kubectl config view
```
