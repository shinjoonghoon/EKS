```
cluster=payments
oidc_id=$(aws eks describe-cluster --name $cluster --query "cluster.identity.oidc.issuer" --profile eks-admin --output text | cut -d '/' -f 5)
echo $oidc_id
```
assume-role: EKSClusterCreator
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
