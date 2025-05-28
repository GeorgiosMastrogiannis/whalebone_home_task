# Whalebone Home Task
### by Georgios Mastrogiannis

## Overview
This repository contains my solution to the **Whalebone Home Task**, where I:
- **Deployed** a Kubernetes cluster on **AWS EKS**


---

## Infrastructure Setup
- **Decision:** Use **AWS EKS** to create the kubernetes cluster for real-world experience.
- Since Elastic Search will need 3 dedicated nodes, I will create 2 additional nodes for any infra tools.
- **Tools Used:** AWS CLI, eksctl, Terraform, kubectl, Helm.

**AWS CLI Configuration:**  
`aws configure --profile aws-personal`  

**Creating the EKS Cluster with eksctl:**  
`eksctl create cluster -f wb-task-cluster.yaml --profile aws-personal`  

![EKS Cluster overview](screenshots/eks_cluster_overview.png)

---

✅ **Create new StorageClass**
```
georgiosmastrogiannis@Evas-MacBook-Pro k8s % kubectl apply -f elastic-storageclass.yaml
storageclass.storage.k8s.io/elastic-storageclass created
georgiosmastrogiannis@Evas-MacBook-Pro k8s % 
georgiosmastrogiannis@Evas-MacBook-Pro k8s % 
georgiosmastrogiannis@Evas-MacBook-Pro k8s % kubectl get storageClass               
NAME                             PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
elastic-storageclass (default)   ebs.csi.aws.com         Retain          WaitForFirstConsumer   true                   10s
gp2                              kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  3h42m
```

**Create Elastic Search credentials secret**
`kubectl create secret ...`
This kubernetes secret will be encrypted at rest, meaning in etcd, but still visible with kubectl as per documentation:
https://docs.aws.amazon.com/eks/latest/userguide/envelope-encryption.html
If time allows will introduce sealed secrets

