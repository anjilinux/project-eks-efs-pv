AWS Elastic Kubernetes Service (EKS) EFS Provisioner  
===================================================

This solution shows you how to create Persistent Storage for an AWS EKS Cluster using AWS EFS. This readme updates an article "efs-provisioner" referenced below and provides a more basic step by step process.  

Steps:  
  Create Amazon EKS Cluster  
  Create AWS EFS  
  Configure AWS EFS Security Group  
  Checkout aws-eks-efs-provisioner from github  
  Configure EKS EFS Provisioner  
  Test EKS EFS Provisioner  
  Remove AWS EFS Provisioner  
  
  
## Create Amazon EKS Cluster
This assumes you already have a EKS Cluster and Dashboard up and running with kubectl configured.
Please see "AWS Elastic Kubernetes Service (EKS) QuickStart" link below for setting up a cluster.    
```
https://github.com/kskalvar/aws-eks-cluster-quickstart
```

## Create AWS EFS
Be sure your EKS Cluster is up and running as you'll be putting a mount target in each of your Cluster's Availability Zones.

### AWS EFS Dashboard
Click on "Create file system"  
```
Configure file system access:
VPC: <Select VPC created for your EKS Cluster>

Create mount targets:
Availability Zone: It will auto-populate Availability Zones based on your VPC selection
Subnet: It will auto-populate Subnets based on your VPC selection
IP Address:  Automatic
Security groups: It will auto-populate your default security group based on your VPC selection
```
Click on "Next Step"  
Click on "Next Step"  
Click on "Create File System"  

Scroll down to "Mount targets" section  
Wait for all Mount target state to appear as "Available" before proceeding  

## Configure AWS EFS Security Group
### AWS EC2 Dashboard
On the left hand side bar  
Click on "Security Groups"  

Identify the "Group ID" for both the "default" "VPC ID" for your EKS Cluster and the "NodeSecurityGroup"    
for your EKS Cluster.  Note:  Both should appear within the same "VPC ID"
```
sg-<group id> eks-cluster-demo-EKSNodeGroup-*-NodeSecurityGroup-*   vpc-<vpc id>
sg-<group id> default                                               vpc-<vpc-id>
```
Copy the "Group ID" for the "NodeSecurityGroup"  

Select the "default" security group for the "VPC ID" where your EKS Cluster resides  
Click on "Inbound Rules"  
Click on Edit Rules  
Click on "Add Rule"  
```
Type: NFS
Protocol: TCP
Port Range: 2049
Source: Custom
Text Field: "sg-<group id>" for the "NodeSecurityGroup"
```

## Checkout aws-eks-efs-provisioner from github
You will need to ssh into the AWS EC2 Instance you created with the EKS Cluster which has kubectl. This is a step by step process.   

On the instance you have kubectl configured, checkout the provisioner repo from github.  
```
git clone https://github.com/kskalvar/aws-eks-cluster-efs-provisioner
cp ~/aws-eks-cluster-efs-provisioner/scripts/configure-kube-efs-provisioner ~
chmod 777 ~/configure-kube-efs-provisioner
cp ~/aws-eks-cluster-efs-provisioner/scripts/manifest.yaml.txt ~
cp ~/aws-eks-cluster-efs-provisioner/kube-config/special-rbac.yaml ~
```

## Configure EKS EFS Provisioner
You will need to ssh into the AWS EC2 Instance you created with the EKS Cluster which has kubectl. This is a step by step process.  
```
NOTE:  There is a script in /home/ec2-user called "configure-kube-efs-provisioner".  
       You may run this script to automate the creation and population of environment 
       variables in the manifest.yaml file and automatically run kubectl apply special-rebac.yaml
       and manifest.yaml.  It uses the AWS EFS I specified in this HOW-TO.  So if you
       have more than one EFS it won't work.  If you do use the script then all
       you need to do is goto "Verify EFS Provisioner from Kubernetes Dashboard".
```

### Configure RBAC
Configure RBAC
``` 
kubectl apply -f special-rbac.yaml  
```
### Configure EFS Provisioner
Get the AWS EFS File System ID using the AWS Cli
```
aws efs describe-file-systems --query FileSystems[].FileSystemId[] --output text
```

Edit and replace manifest.yaml with value above  
cp ~/manifest.yaml.txt ~/manifest.yaml  
```
<myfs>
```

Configure EFS Provisioner
```
kubectl apply -f ~/manifest.yaml
```

### Verify EFS Provisioner from Kubernetes Dashboard
To verify the EFS Provisioner is working correctly do the following:  
#### Kubernetes Dashboard:  

Click on "Cluster/Storage Classes"  
Storage Classes should show "aws-efs"  

Click on "Workloads/Deployments"  
Pods should show "efs-provisioner"  

Click on "Config and Storage/Persistent Volume Claims"  
Persistent Volume Claims should show "efs"


## Test EKS EFS Provisioner
You will need to ssh into the AWS EC2 Instance you created with the EKS Cluster which has kubectl. This is a step by step process.

To Test the efs provisioner we'll create a web service consisting of 3 containers.    
We'll attach to each container, goto the mount point and create a file which should  
be observable in each of the other containers when we attach to them and go to  
the same mount point.  
```
cp ~/aws-eks-cluster-efs-provisioner/sample-app/web-deployment-service.yaml ~
kubectl apply -f web-deployment-service.yaml
kubectl get pods
```

### Inspect each container 
Test each of the containers listed above using the following procedure
```
kubectl exec -it <web container> bash
cd /mnt
ls
touch <unique file name>
exit
```

## Remove AWS EFS Provisioner
You will need to ssh into the AWS EC2 Instance you created with the EKS Cluster which has kubectl. This is a step by step process.

### Delete web-deployment-service
Delete web service
```
kubectl delete -f web-deployment-service.yaml
```
### Delete EKS EFS Provisioner
Delete EFS Provisioner from EKS
```
kubectl delete -f manifest.yaml
```

### Remove NFS Inbound Rule from default VPC Security Group
#### AWS EC2 Dashboard
On the left hand side bar  
Click on "Security Groups"  

Select the "default" security group for the "VPC ID" where your EKS Cluster resides  
Click on "Inbound Rules"  
Click on Edit Rules  
Delete "NFS" Rule  
Save

### Remove AWS EFS File System
#### AWS EFS Dashboard
File systems:  
Select "file system id"   
Click on "Actions"  
Select "Delete file system"  


## References
AWS Elastic Kubernetes Service (EKS) QuickStart  
https://github.com/kskalvar/aws-eks-cluster-quickstart

efs-provisioner  
https://github.com/kubernetes-incubator/external-storage/tree/master/aws/efs

