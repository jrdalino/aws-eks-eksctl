# AWS Elastic Kubernetes Service (EKS) using EKSCTL

## Module 3: Install Kubernetes Tools and Launch EKS using EKCTL

### Step 3.1: Create the default ~/.kube directory for storing kubectl configuration
```
$ mkdir -p ~/.kube
```

### Step 3.2: Install kubectl on MAC
```
$ sudo curl -o kubectl "https://amazon-eks.s3-us-west-2.amazonaws.com/1.11.5/2018-12-06/bin/darwin/amd64/kubectl"
$ sudo chmod +x ./kubectl
$ mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
$ echo 'export PATH=$HOME/bin:$PATH' >> ~/.bash_profile
$ kubectl version --short --client
```

### Step 3.3: Install IAM Authenticator
```
$ brew install aws-iam-authenticator
$ aws-iam-authenticator help
```

### Step 3.4: Install JQ and envsubst
```
$ brew install jq
$ brew install gettext
$ brew link --force gettext
```

### Step 3.5: Verify the binaries are in the path and executable
```
$ for command in kubectl aws-iam-authenticator jq envsubst
  do
    which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND"
  done
```

### Step 3.6: Generate an SSH Key for the Worker Nodes and upload the public key to your EC2 region
```
$ ssh-keygen
$ aws ec2 import-key-pair --key-name "eksworkernodes" --public-key-material file://~/.ssh/id_rsa.pub
```

### Step 3.7: Download the eksctl binaries
```
$ brew install weaveworks/tap/eksctl
$ eksctl version
```

### Step 3.8: Create an EKS Cluster (This will take ~15 minutes) and test cluster
```
$ eksctl create cluster \
--name=calculator-eksctl \
--nodes=2 \
--node-ami=auto \
--node-type=t2.small \ 
--region=${AWS_REGION}
$ kubectl get nodes
```

```
[ℹ]  using region us-east-1
[ℹ]  setting availability zones to [us-east-1d us-east-1a]
[ℹ]  subnets for us-east-1d - public:192.168.0.0/19 private:192.168.64.0/19
[ℹ]  subnets for us-east-1a - public:192.168.32.0/19 private:192.168.96.0/19
[ℹ]  nodegroup "ng-2e8e1187" will use "ami-0abcb9f9190e867ab" [AmazonLinux2/1.12]
[ℹ]  creating EKS cluster "calculator-eksctl" in "us-east-1" region
[ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial nodegroup
[ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-1 --name=calculator-eksctl'
[ℹ]  2 sequential tasks: { create cluster control plane "calculator-eksctl", create nodegroup "ng-2e8e1187" }
[ℹ]  building cluster stack "eksctl-calculator-eksctl-cluster"
[ℹ]  deploying stack "eksctl-calculator-eksctl-cluster"
[ℹ]  buildings nodegroup stack "eksctl-calculator-eksctl-nodegroup-ng-2e8e1187"
[ℹ]  --nodes-min=2 was set automatically for nodegroup ng-2e8e1187
[ℹ]  --nodes-max=2 was set automatically for nodegroup ng-2e8e1187
[ℹ]  deploying stack "eksctl-calculator-eksctl-nodegroup-ng-2e8e1187"
[✔]  all EKS cluster resource for "calculator-eksctl" had been created
[✔]  saved kubeconfig as "/Users/jrdalino/.kube/config"
[ℹ]  adding role "arn:aws:iam::707538076348:role/eksctl-calculator-eksctl-nodegrou-NodeInstanceRole-1IGO2PUALGGAC" to auth ConfigMap
[ℹ]  nodegroup "ng-2e8e1187" has 1 node(s)
[ℹ]  node "ip-192-168-24-30.ec2.internal" is not ready
[ℹ]  waiting for at least 2 node(s) to become ready in "ng-2e8e1187"
[ℹ]  nodegroup "ng-2e8e1187" has 2 node(s)
[ℹ]  node "ip-192-168-24-30.ec2.internal" is ready
[ℹ]  node "ip-192-168-35-7.ec2.internal" is ready
[ℹ]  kubectl command should work with "/Users/jrdalino/.kube/config", try 'kubectl get nodes'
[✔]  EKS cluster "calculator-eksctl" in "us-east-1" region is ready
```

### Step 3.9: Export Worker Role name
```
$ INSTANCE_PROFILE_NAME=$(aws iam list-instance-profiles | jq -r '.InstanceProfiles[].InstanceProfileName' | grep nodegroup)
$ ROLE_NAME=$(aws iam get-instance-profile --instance-profile-name $INSTANCE_PROFILE_NAME | jq -r '.InstanceProfile.Roles[] | .RoleName')
$ echo "export ROLE_NAME=${ROLE_NAME}" >> ~/.bash_profile
```

### (Optional) Clean up
```
$ eksctl delete cluster --name=calculator-eksctl
$ Manually delete Worker Nodes SSH Key
```
