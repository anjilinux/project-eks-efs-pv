AWS Elastic Kubernetes Service (EKS) EFS Provisioner QuickStart  
===============================================

This solution shows you how to create Persistent Storage for an AWS EKS Cluster. This readme updates an article "efs-provisioner" referenced below and provides a more basic step by step process.  

Steps:  
  Create your Amazon EKS Cluster  
  Checkout aws-eks-efs-provisioner from github  
 

## Create your Amazon EKS Cluster
This assumes you already have a EKS Cluster up and running with kubectl configured.
Please see "AWS Elastic Kubernetes Service (EKS) QuickStart" link below for setting up a cluster.    
```
https://github.com/kskalvar/aws-eks-cluster-quickstart  
```

## Checkout aws-eks-efs-provisioner from github
You will need to ssh into the AWS EC2 Instance you created above.  This is a step by step process.  

On the instance you have kubectl configured, checkout the codesuite repo from github.  
```
git clone https://github.com/kskalvar/aws-eks-pipeline-quickstart
```




## References
AWS Elastic Kubernetes Service (EKS) QuickStart  
https://github.com/kskalvar/aws-eks-cluster-quickstart

efs-provisioner  
https://github.com/kubernetes-incubator/external-storage/tree/master/aws/efs  
