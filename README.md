AWS Elastic Kubernetes Service (EKS) EFS Provisioner  
===================================================

This solution shows you how to create Persistent Storage for an AWS EKS Cluster using AWS EFS. This readme updates an article "efs-provisioner" referenced below and provides a more basic step by step process.  

Steps:  
  Create Amazon EKS Cluster  
  Configure AWS EFS Security Group  
  Create AWS EFS  
  Checkout aws-eks-efs-provisioner project from github  
  Configure EKS EFS Provisioner  
  Test EKS EFS Provisioner  
  Remove AWS EFS Provisioner  
  
  
## Create Amazon EKS Cluster
This assumes you already have a EKS Cluster and Dashboard up and running with kubectl configured.
Please see "AWS Elastic Kubernetes Service (EKS) QuickStart" link below for setting up a cluster.    
```
https://github.com/kskalvar/aws-eks-cluster-quickstart
```

Wait for EKS Cluster configuration to complete before proceeding    

## Configure AWS EFS Security Group
### AWS EC2 Dashboard
On the left hand side bar  
Click on "Security Groups"  

Click on "Create Security Group"
```
Security group name: eks-cluster-demo-efs-security-group
Description: eks-cluster-demo-efs-security-group
VPC: eks-cluster-demo-EKSVpc-*-VPC
```
Click on "Create"  

Select "eks-cluster-demo-efs-security-group"  
Click on "Actions"  
Click on "Edit inbound rules"  
```
Type        Protocol  Port Range     Source 
All Traffic All       0 - 65535      Custom <eks-cluster-demo-efs-security-group Security GROUP ID>
NFS         TCP       2049           Custom <eks-cluster-demo*EKSNodeGroup* Security GROUP ID> 
```
Click on "Save"  

## Create AWS EFS
Be sure your EKS Cluster is up and running as you'll be putting a mount target in each of your Cluster's Availability Zones.

### AWS EFS Dashboard
Click on "Create file system"  
```
Configure file system access:
VPC: eks-cluster-demo-EKSVpc-*-VPC

Create mount targets:
Availability Zone: It will auto-populate Availability Zones based on your VPC selection
Subnet: It will auto-populate Subnets based on your VPC selection
IP Address:  Automatic
Security groups: It will auto-populate, changed to <eks-cluster-demo-efs-security-group> you created above
```
Click on "Next Step"  
Click on "Next Step"  
Click on "Create File System"  

Scroll down to "Mount targets" section  
Wait for all Mount target state to appear as "Available" before proceeding  

## Checkout aws-eks-efs-provisioner project from github
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
Deployments should show "efs-provisioner" green

Click on "Workloads/Pods"  
Pods should show "efs-provisioner" green  

Click on "Config and Storage/Persistent Volume Claims"  
Persistent Volume Claims should show "efs" green


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

### Remove AWS EFS File System
#### AWS EFS Dashboard
File systems:  
Select "file system id"   
Click on "Actions"  
Select "Delete file system"  

### Remove EFS Security Group
#### AWS EC2 Dashboard
On the left hand side bar  
Click on "Security Groups"  

Select "eks-cluster-demo-efs-security-group"   
Click on "Actions" button  
Select "Delete Security Group"  


## References
AWS Elastic Kubernetes Service (EKS) QuickStart  
https://github.com/kskalvar/aws-eks-cluster-quickstart

efs-provisioner  
https://github.com/kubernetes-incubator/external-storage/tree/master/aws/efs

