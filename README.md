# Provision Amazon EKS K8s

Two ways to provision EKS Cluster in AWS

- AWS CLI https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html
- EKS CLI (newer) https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

## AWS CLI

### Getting Started:

### 1. Create Free Tier Account

### 2. Configure AWS CLI

```
# Run Amazon CLI

docker run -it --rm -v ${PWD}:/work -w /work --entrypoint /bin/sh amazon/aws-cli:2.0.17

# switch directories

cd ./amazon

# install utilities

yum install jq gzip nano tar git

# Login to AWS
# Access your "My Security Credentials" section in your profile.
# Create an access key

aws configure
```

### 3. Create IAM role & Assign IAM role with Policies

```
# create our role for EKS

# create role and extract ARN

role_arn=$(aws iam create-role --role-name <custom-role-name> --assume-role-policy-document file://assume-policy.json | jq .Role.Arn | sed s/\"//g)

# attach an AWS managed policy (AmazonEKSClusterPolicy) to the IAM role

aws iam attach-role-policy --role-name <custom-role-name> --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
```

### 4. Create VPC with Subnets for Cluster

```
# download VPC CloudFormation template

curl https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-05-08/amazon-eks-vpc-sample.yaml -o vpc.yaml

# deploy VPC with Public subnets using CloudFormation

aws cloudformation deploy --template-file vpc.yaml --stack-name <custom-cf-stack>

# get stack details
# use resource info in stack.json file in next section

aws cloudformation list-stack-resources --stack-name <custom-cf-stack> > stack.json
```

### 5. Create EKS Cluster

```
# create cluster

aws eks create-cluster \
--name <custom-eks-cluster> \
--role-arn $role_arn \
--resources-vpc-config subnetIds=<subnet-Id1>,<subnet-Id2>,<subnet-Id3>,securityGroupIds=<sg-Id>,endpointPublicAccess=true,endpointPrivateAccess=false

aws eks list-clusters
aws eks describe-cluster --name <custom-eks-cluster>
```

### 6. Accessing EKS Cluster

```
# get a kubeconfig for our cluster

aws eks update-kubeconfig --name getting-started-eks --region <eks-cluster-region>

# grab the config if you want it

cp ~/.kube/config .

# download kubectl

curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.23.6/bin/linux/amd64/kubectl

# move to /usr/bin

chmod +x ./kubectl && mv ./kubectl /usr/local/bin/kubectl
```

### 7. Add Nodes to EKS Cluster

```
# create role for nodes

role_arn=$(aws iam create-role --role-name <custom-eks-role-nodes> --assume-role-policy-document file://assume-node-policy.json | jq .Role.Arn | sed s/\"//g)

# assign security, networking & registry policies
aws iam attach-role-policy --role-name <custom-eks-role-nodes> --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
aws iam attach-role-policy --role-name <custom-eks-role-nodes> --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
aws iam attach-role-policy --role-name <custom-eks-role-nodes> --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

# create nodes

aws eks create-nodegroup \
--cluster-name <custom-eks-cluster> \
--nodegroup-name <group-name> \
--node-role $role_arn \
--subnets <subnet-Id> \
--disk-size 30 \
--scaling-config minSize=1,maxSize=1,desiredSize=1 \
--instance-types t2.micro
```
