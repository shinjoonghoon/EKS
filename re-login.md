```
account=$(aws sts get-caller-identity --query Account --output text --profile eks-admin)
rolearn="arn:aws:iam::${account}:role/EKSClusterCreator"
echo $account
echo $rolearn

creds=$(aws sts assume-role --role-arn ${rolearn} \
   --role-session-name "create-eks-cluster" \
   --query 'Credentials.[{AWS_ACCESS_KEY_ID: AccessKeyId}, {AWS_SESSION_TOKEN: SessionToken}, {AWS_SECRET_ACCESS_KEY: SecretAccessKey}]' \
   --profile eks-admin | jq -r '.[]' | sed 's/{//;s/}//;s/"//g;s/ AWS_ACCESS_KEY_ID: /export AWS_ACCESS_KEY_ID=/;s/ AWS_SESSION_TOKEN: /export AWS_SESSION_TOKEN=/;s/ AWS_SECRET_ACCESS_KEY: /export AWS_SECRET_ACCESS_KEY=/')
echo $creds

eval $creds

aws sts get-caller-identity

```

```
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```
