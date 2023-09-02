
# Create a VPC
Using [YAML](https://github.com/shinjoonghoon/eks-basic/blob/main/eks-vpc-2AZ-private-subnet-only.yml).

>a) region: 1)Command line --profile option, 2)Value of AWS_PROFILE environment variable, 3)Otherwise profile is [default]

>b) configure transit gateway route table
```
vpcid=vpc-xxxxxxxxxxxxxxxxx
```
```
region=$(aws configure get region --profile eksadmin)
echo $vpcid
echo $region
```


# Create VPC endpoint
> Your cluster's VPC subnets must have a VPC interface endpoint for any AWS services that your Pods need access to. For more information, see Access an AWS service using an interface VPC endpoint. Some commonly-used services and endpoints are listed in the following table. For a complete list of endpoints, see AWS services that integrate with AWS PrivateLink in the AWS PrivateLink Guide.
* https://docs.aws.amazon.com/eks/latest/userguide/private-clusters.html
* https://docs.aws.amazon.com/vpc/latest/privatelink/create-interface-endpoint.html
* https://docs.aws.amazon.com/vpc/latest/privatelink/interface-endpoints.html

# create-security-group
```
aws ec2 create-security-group --description "Security group of endpoints" --group-name "Security group of endpoints" --vpc-id $vpcid --output json --region $region  | jq '.[]'
```

* authorize-security-group-ingress
```
aws ec2 authorize-security-group-ingress \
    --group-id $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`Security group of endpoints`)].GroupId' --output text --region $region ) \
    --protocol -1 --port -1 --cidr 10.0.0.0/8 \
    --region $region
    --output json | jq '.[]'

```
* Tag VPC subnets
>Name=tag:Endpoints,Values=true

* describe-subnets
```
sbs=$(aws ec2 describe-subnets   --filters "Name=vpc-id,Values=$vpcid" "Name=tag:Endpoints,Values=true" --region ap-northeast-2 --query 'Subnets[].SubnetId' --output text)
echo $sbs

```

* describe-route-tables
```
rts=$(aws ec2 describe-route-tables   --filters "Name=vpc-id,Values=$vpcid" "Name=tag:Endpoints,Values=true" --region ap-northeast-2 --query 'RouteTables[].RouteTableId' --output text)
echo $rts

```

* describe-vpc-endpoints
```
aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=$vpcid" --region ap-northeast-2 --query 'VpcEndpoints[].[State,ServiceName]' --output text
```

# s3 vpc endpoint of type **gateway**
>602401143452.dkr.ecr.ap-northeast-2.amazonaws.com/eks/coredns:v1.10.1-eksbuild.1
```
aws ec2 create-vpc-endpoint \
    --region $region \
    --vpc-endpoint-type Gateway \
    --vpc-id  $vpcid \
    --route-table-ids $(aws ec2 describe-route-tables   --filters "Name=vpc-id,Values=$vpcid" "Name=tag:Endpoints,Values=true" --region ap-northeast-2 --query 'RouteTables[].RouteTableId' --output text) \
    --service-name com.amazonaws.$region.s3 \
    --output json | jq '.[]'

```

# s3 vpc endpoint of type **interface**
```
myarray=(
com.amazonaws.$region.s3
)
for v in "${myarray[@]}"; do
    aws ec2 create-vpc-endpoint \
    --region $region \
    --vpc-id  $vpcid \
    --subnet-ids $(aws ec2 describe-subnets   --filters "Name=vpc-id,Values=$vpcid" "Name=tag:Endpoints,Values=true" --region ap-northeast-2 --query 'Subnets[].SubnetId' --output text) \
    --vpc-endpoint-type Interface \
    --service-name  $v \
    --security-group-ids $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`Security group of endpoints`)].GroupId' --output text --region $region ) \
    --output json | jq '.[]'
done

```

# AWS services
```
myarray=(
com.amazonaws.$region.appmesh-envoy-management
com.amazonaws.$region.cloudformation
com.amazonaws.$region.ec2
com.amazonaws.$region.ec2messages
com.amazonaws.$region.ecr.api
com.amazonaws.$region.ecr.dkr
com.amazonaws.$region.eks
com.amazonaws.$region.elasticloadbalancing
com.amazonaws.$region.kms
com.amazonaws.$region.logs
com.amazonaws.$region.ssm
com.amazonaws.$region.ssmmessages
com.amazonaws.$region.sts
com.amazonaws.$region.xray
)
for v in "${myarray[@]}"; do
    aws ec2 create-vpc-endpoint \
    --region $region \
    --vpc-id  $vpcid \
    --subnet-ids $(aws ec2 describe-subnets   --filters "Name=vpc-id,Values=$vpcid" "Name=tag:Endpoints,Values=true" --region ap-northeast-2 --query 'Subnets[].SubnetId' --output text) \
    --vpc-endpoint-type Interface \
    --service-name  $v \
    --security-group-ids $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`Security group of endpoints`)].GroupId' --output text --region $region ) \
    --output json | jq '.[]'
done

```

# Describe VPC endpoints
```
aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=$vpcid" --region ap-northeast-2 --query 'VpcEndpoints[].[State,VpcEndpointType,ServiceName]' --output text
```

