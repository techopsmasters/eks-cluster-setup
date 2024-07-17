# EKS clustercreation using eksctl:

# Step1: Take EC2 Instance with t2.xlarge instance type
# Step2: Create IAM Role with Admin policy for eks-cluster and attach to ec2-instance
# Step3: Install kubectl
	curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.29/2023-01-11/bin/linux/amd64/kubectl
	chmod +x ./kubectl
	mkdir -p $HOME/bin
	cp ./kubectl $HOME/bin/kubectl
	export PATH=$HOME/bin:$PATH
	echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
	source $HOME/.bashrc
	kubectl version --short --client

# Step4: Install eksctl:
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/bin
    eksctl version

# Step5: Cluster creation:
    eksctl create cluster --name=eksdemo \
                      --region=ap-south-1 \
                      --zones=ap-south-1a,ap-south-1b \
                      --without-nodegroup 
   eksctl create cluster --name=eksdemo --region=ap-south-1 --version=1.29 --zones=ap-south-1a,ap-south-1b --without-nodegroup

					  
# Step6: Add Iam-Oidc-Providers:
    eksctl utils associate-iam-oidc-provider \
        --region us-east-1 \
        --cluster eksdemo \
        --approve
					  
# Step7: Create node-group:
    eksctl create nodegroup --cluster=eksdemo \
                       --region=ap-south-1 \
                       --name=eksdemo-ng-public \
                       --node-type=t2.medium \
                       --nodes=2 \
                       --nodes-min=1 \
                       --nodes-max=2 \
                       --node-volume-size=10 \
                       --ssh-access \
                       --ssh-public-key=devops-demo-est \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access	
					   
# CleanUP
Delete node-group:
			   
 eksctl delete nodegroup --cluster=eksdemo-dev --region=ap-south-1 --name=eksdemo-ng-public
Delete Cluster:
				   
    eksctl delete cluster --name=eksdemo                       --region=ap-south-1	
		      
		      
# EKS-Fargate-Setup

## EKS Fargate Cluster Setup
```bash
eksctl create cluster --name eksdemo --region ap-south-1 --fargate
```

## OIDC Provider Creation
```bash
eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster=eksdemo --approve
```

## Deploy application with below
```bash
cat << EOF > nginx-deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: 451060642371.dkr.ecr.us-east-1.amazonaws.com/nginx:latest
        ports:
        - containerPort: 80
EOF       
		
kubectl apply -f nginx-deployment.yaml
```

