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

✅ **AWS CLI Configuration:**  
`aws configure --profile aws-personal`  

✅ **Creating the EKS Cluster with eksctl:**  
`eksctl create cluster --name home-task-eks --region eu-west-1 --nodes 5 --node-type t3.small --profile aws-personal`  

![EKS Cluster overview](screenshots/eks_cluster_overview.png)

---