# Install Tools
* https://kubernetes.io/docs/tasks/tools/
* https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/install-kubectl.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/eksctl.html

```
sudo yum install jq
```

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
<!-- sudo ./aws/install -->
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
ls -l /usr/local/bin/aws
echo "complete -C '/usr/local/bin/aws_completer' aws" >> ~/.bash_profile
source ~/.bashrc

```

```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chmod +x /usr/local/bin/kubectl
kubectl version

```

```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin

```

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
echo "source <(eksctl completion bash)" >> ~/.bashrc
source ~/.bashrc

```

# Security group of cluster
* https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html

```
vpcid=vpc-0e5dcdc77bf5fcc86
region=$(aws configure get region  )
echo $vpcid
echo $region

```

```
aws ec2 create-security-group --description "Security group of Payments Cluster" --group-name "eks-cluster-sg-payments" --vpc-id $vpcid --output json | jq '.[]'
```

```
aws ec2 authorize-security-group-ingress \
    --group-id $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text  ) \
    --protocol -1 --port -1 \
    --source-group $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text  ) \
    --output json | jq '.[]'
```

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
* https://eksctl.io/usage/schema/

```
aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text
```

```
aws ec2 describe-subnets --filters "Name=vpc-id, Values=$vpcid"   --query "Subnets[*].{id:SubnetId,az:AvailabilityZone,subnet:Tags[?Key=='Name']|[0].Value}" --output text
```

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

