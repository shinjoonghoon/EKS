# Creating an IAM OIDC provider for cluster
* https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
```
cluster=payments
oidc_id=$(aws eks describe-cluster --name $cluster --query "cluster.identity.oidc.issuer" --profile eksadmin --output text | cut -d '/' -f 5)
echo $oidc_id
```
* assume-role: EKSClusterCreator
```
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```

```
eksctl utils associate-iam-oidc-provider --cluster $cluster --approve --region ap-northeast-2
```

```
aws iam list-open-id-connect-providers
```

```
aws eks describe-cluster --name $cluster --query 'cluster.identity.oidc.issuer' --output text
```

