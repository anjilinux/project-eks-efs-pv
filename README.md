AWS Elastic Kubernetes Service (EKS) EFS Provisioner QuickStart  
===============================================

This solution shows you how to create Persistent Storage for an AWS EKS Cluster using AWS EFS. This readme updates an article "efs-provisioner" referenced below and provides a more basic step by step process.  

Steps:  
  Create Amazon EKS Cluster  
  Create AWS EFS  
  Configure AWS EFS Security Group  
  Checkout aws-eks-efs-provisioner from github  
  Configure EKS EFS Provisioner  
  Test EKS EFS Provisioner  
  Remove Your AWS EFS Provisioner  
  
  
## Create Amazon EKS Cluster
This assumes you already have a EKS Cluster and Dashboard up and running with kubectl configured.
Please see "AWS Elastic Kubernetes Service (EKS) QuickStart" link below for setting up a cluster.    
```
https://github.com/kskalvar/aws-eks-cluster-quickstart
```

## Create AWS EFS
Be sure your EKS Cluster is up and running as you'll be putting a mount target in each of your VPC's Availability Zones.

### AWS EFS Dashboard
Click on "Create file system"  
```
VPC: Select VPC created for your EKS Cluster
Create mount targets:  It will auto-populate Availability Zones based on your VPC selection
IP Address:  Automatic
Security groups: It will auto-populate your default security groups based on your VPC
```
Click on "Next Step"  
Click on "Next Step"  
Click on "Create File System"  

Scroll down to "Mount targets" section  
Wait for all Mount target state to appear as "Available" before proceeding  

## Configure AWS EFS Security Group
### AWS VPC Dashboard
On the left hand side bar  
Click on "Security Groups"  

Identify the "Group ID" for both the "default" "VPC ID" for your EKS Cluster and the "NodeSecurityGroup"    
for your EKS Cluster.  Note:  Both should appear within the same "VPC ID"
```
*-EKSNodeGroup-*-NodeSecurityGroup-*
default
```
Copy the <Group ID> for the "NodeSecurityGroup"  

Select the "default" security group for the "VPC ID" where your EKS Cluster resides  
Click on "Inbound Rules"
Click on Edit Rules
Click on "Add Rule"
```
Type: NFS
Protocol: TCP
Port Range: 2049
Source: Custom
Text Field: <Group ID> for the "NodeSecurityGroup"
```

## Checkout aws-eks-efs-provisioner from github
You will need to ssh into the AWS EC2 Instance you created above.  This is a step by step process.    

On the instance you have kubectl configured, checkout the provisioner repo from github.  
```
git clone https://github.com/kskalvar/aws-eks-cluster-efs-provisioner
cp ~/aws-eks-cluster-efs-provisioner/scripts/configure-kube-efs-provisioner ~
chmod 777 ~/configure-kube-efs-provisioner
cp ~/aws-eks-cluster-efs-provisioner/scripts/manifest.yaml.txt ~
cp ~/aws-eks-cluster-efs-provisioner/kube-config/special-rbac.yaml ~
```

## Configure EKS EFS Provisioner
Use kubectl to create the EFS Provisioner
```
NOTE:  There is a script in /home/ec2-user called "configure-kube-efs-provisioner".  
       You may run this script to automate the creation and population of environment 
       variables in manifest.yaml and automatically kubectl apply special-rebac.yaml
       and manifest.yaml.  It uses the AWS EFS I specified in this HOW-TO.  So if you
       have more than one EFS it won't work.  If you do use the script then all
       you need to do is check the Kubernetes Dashboard to see everything is working.
```

### Configure RBAC Cluster Role
You will need to ssh into the AWS EC2 Instance you created above. This is a step by step process.  
``` 
kubectl apply -f special-rbac.yaml  
```

### Test from Kubernetes Dashboard



## Test EKS EFS Provisioner
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
kubectl exec -it web-<replica set>-<random string> bash
cd /mnt
ls
touch web-<replica set>-<random string>
exit
```

## Remove Your AWS EFS Provisioner

### Remove web-deployment-service
kubectl delete -f web-deployment-service.yaml


### Remove EKS EFS Provisioner
kubectl delete -f manifest.yaml

### Remove NFS Inbound Rule from default VPC Security Group


### Remove AWS EFS File System


## References
AWS Elastic Kubernetes Service (EKS) QuickStart  
https://github.com/kskalvar/aws-eks-cluster-quickstart

efs-provisioner  
https://github.com/kubernetes-incubator/external-storage/tree/master/aws/efs

