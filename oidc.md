# Creating an IAM OIDC provider for cluster
* https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
```
cluster=payments
oidc_id=$(aws eks describe-cluster --name $cluster --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
echo $oidc_id
```
* assume-role: EKSClusterCreator
```
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```

```
eksctl utils associate-iam-oidc-provider --cluster $cluster --approve --region ap-northeast-2
```
>Error: operation error IAM: GetOpenIDConnectProvider, https response error StatusCode: 0, RequestID: , request send failed, Post "https://iam.amazonaws.com/": dial tcp x.x.x.x:443: i/o timeout
```
aws iam list-open-id-connect-providers
```
>Connect timeout on endpoint URL: "https://iam.amazonaws.com/"
```
aws eks describe-cluster --name $cluster --query 'cluster.identity.oidc.issuer' --output text
```

