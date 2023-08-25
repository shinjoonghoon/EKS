# Install Tools
* https://kubernetes.io/docs/tasks/tools/
* https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/install-kubectl.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/eksctl.html

* jq
```
sudo yum install jq
```

* aws cli
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
<!-- sudo ./aws/install -->
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
ls -l /usr/local/bin/aws
complete -C '/usr/local/bin/aws_completer' aws
echo "complete -C '/usr/local/bin/aws_completer' aws" >> ~/.bash_profile
source ~/.bashrc

```

* kubectl
```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chmod +x /usr/local/bin/kubectl
kubectl version

```

* eksctl
```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin

```

* autocomplete
```
echo "source <(kubectl completion bash)" >> ~/.bashrc
echo "source <(eksctl completion bash)" >> ~/.bashrc
source ~/.bashrc

```
