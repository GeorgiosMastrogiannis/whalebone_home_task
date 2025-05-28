# Whalebone Home Task
### by Georgios Mastrogiannis

## Overview
This repository contains my solution to the **Whalebone Home Task**, where I:
- **Deployed** a Kubernetes cluster on **AWS EKS**


---

## Infrastructure Setup
- **Decision:** Use **AWS EKS** to create the kubernetes cluster for real-world experience.
- Since Elastic Search will need 3 dedicated nodes, I will create 2 additional nodes for any infra tools.
- **Tools Used:** AWS CLI, eksctl, kubectl, Helm.

**AWS CLI Configuration:**  
`aws configure --profile aws-personal`  

**Creating the EKS Cluster with eksctl:**  
`eksctl create cluster -f wb-task-cluster.yaml --profile aws-personal`  

![EKS Cluster overview](screenshots/eks_cluster_overview.png)

---

## Apps Installation
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

**Install Elastic Search**
```
georgiosmastrogiannis@Evas-MacBook-Pro k8s % helm install elasticsearch elastic/elasticsearch \
  -n elastic \
  -f elastic-values.yaml
W0528 15:34:27.328870   59985 warnings.go:70] spec.template.spec.containers[0].env[18]: hides previous definition of "ELASTIC_PASSWORD", which may be dropped when using apply
NAME: elasticsearch
LAST DEPLOYED: Wed May 28 15:34:25 2025
NAMESPACE: elastic
STATUS: deployed
REVISION: 1
NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=elastic -l app=elasticsearch-master -w
2. Retrieve elastic user's password.
  $ kubectl get secrets --namespace=elastic elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
3. Test cluster health using Helm test.
  $ helm --namespace=elastic test elasticsearch
```

```
georgiosmastrogiannis@Evas-MacBook-Pro k8s % kubectl get pods -n elastic -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP               NODE                                           NOMINATED NODE   READINESS GATES
elasticsearch-master-0   1/1     Running   0          9m53s   192.168.36.154   ip-192-168-39-185.eu-west-1.compute.internal   <none>           <none>
elasticsearch-master-1   1/1     Running   0          10m     192.168.82.44    ip-192-168-78-72.eu-west-1.compute.internal    <none>           <none>
elasticsearch-master-2   1/1     Running   0          10m     192.168.1.60     ip-192-168-17-61.eu-west-1.compute.internal    <none>           <none>
georgiosmastrogiannis@Evas-MacBook-Pro k8s % 
georgiosmastrogiannis@Evas-MacBook-Pro k8s % 
georgiosmastrogiannis@Evas-MacBook-Pro k8s % 
georgiosmastrogiannis@Evas-MacBook-Pro k8s % 
georgiosmastrogiannis@Evas-MacBook-Pro k8s % kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints

NAME                                           TAINTS
ip-192-168-1-123.eu-west-1.compute.internal    <none>
ip-192-168-17-61.eu-west-1.compute.internal    [map[effect:NoSchedule key:dedicated value:elasticsearch]]
ip-192-168-39-185.eu-west-1.compute.internal   [map[effect:NoSchedule key:dedicated value:elasticsearch]]
ip-192-168-68-117.eu-west-1.compute.internal   <none>
ip-192-168-78-72.eu-west-1.compute.internal    [map[effect:NoSchedule key:dedicated value:elasticsearch]]
georgiosmastrogiannis@Evas-MacBook-Pro k8s % 
georgiosmastrogiannis@Evas-MacBook-Pro k8s % 
georgiosmastrogiannis@Evas-MacBook-Pro k8s % kubectl get nodes -l dedicated=elasticsearch --show-labels

NAME                                           STATUS   ROLES    AGE     VERSION                LABELS
ip-192-168-17-61.eu-west-1.compute.internal    Ready    <none>   5h44m   v1.30.11-eks-473151a   alpha.eksctl.io/cluster-name=wb-task-cluster,alpha.eksctl.io/instance-id=i-069e41c6b73892868,alpha.eksctl.io/nodegroup-name=es-nodes,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=t3.medium,beta.kubernetes.io/os=linux,dedicated=elasticsearch,failure-domain.beta.kubernetes.io/region=eu-west-1,failure-domain.beta.kubernetes.io/zone=eu-west-1a,k8s.io/cloud-provider-aws=ccf8b604b8038cc5aec106e14e7c564c,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-192-168-17-61.eu-west-1.compute.internal,kubernetes.io/os=linux,node-lifecycle=on-demand,node.kubernetes.io/instance-type=t3.medium,topology.ebs.csi.aws.com/zone=eu-west-1a,topology.k8s.aws/zone-id=euw1-az2,topology.kubernetes.io/region=eu-west-1,topology.kubernetes.io/zone=eu-west-1a
ip-192-168-39-185.eu-west-1.compute.internal   Ready    <none>   5h43m   v1.30.11-eks-473151a   alpha.eksctl.io/cluster-name=wb-task-cluster,alpha.eksctl.io/instance-id=i-0e57f6ca978e516a1,alpha.eksctl.io/nodegroup-name=es-nodes,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=t3.medium,beta.kubernetes.io/os=linux,dedicated=elasticsearch,failure-domain.beta.kubernetes.io/region=eu-west-1,failure-domain.beta.kubernetes.io/zone=eu-west-1b,k8s.io/cloud-provider-aws=ccf8b604b8038cc5aec106e14e7c564c,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-192-168-39-185.eu-west-1.compute.internal,kubernetes.io/os=linux,node-lifecycle=on-demand,node.kubernetes.io/instance-type=t3.medium,topology.ebs.csi.aws.com/zone=eu-west-1b,topology.k8s.aws/zone-id=euw1-az3,topology.kubernetes.io/region=eu-west-1,topology.kubernetes.io/zone=eu-west-1b
ip-192-168-78-72.eu-west-1.compute.internal    Ready    <none>   5h44m   v1.30.11-eks-473151a   alpha.eksctl.io/cluster-name=wb-task-cluster,alpha.eksctl.io/instance-id=i-04bf2ae2197427ac8,alpha.eksctl.io/nodegroup-name=es-nodes,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=t3.medium,beta.kubernetes.io/os=linux,dedicated=elasticsearch,failure-domain.beta.kubernetes.io/region=eu-west-1,failure-domain.beta.kubernetes.io/zone=eu-west-1c,k8s.io/cloud-provider-aws=ccf8b604b8038cc5aec106e14e7c564c,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-192-168-78-72.eu-west-1.compute.internal,kubernetes.io/os=linux,node-lifecycle=on-demand,node.kubernetes.io/instance-type=t3.medium,topology.ebs.csi.aws.com/zone=eu-west-1c,topology.k8s.aws/zone-id=euw1-az1,topology.kubernetes.io/region=eu-west-1,topology.kubernetes.io/zone=eu-west-1c
```

##TSHOOT
At my first attempt I created the cluster with t3.small nodes, to save money, but then noticed in the official docs Elastic Search pods need at least 2GB:
https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/elasticsearch-deployment-quickstart
So I had to recreate the cluster with t3.medium which has 4GB

After installing Elastic Search with helm chart, pods where in Pending:
```
georgiosmastrogiannis@Evas-MacBook-Pro k8s % kubectl get pods --namespace=elastic -l app=elasticsearch-master -w
NAME                     READY   STATUS    RESTARTS   AGE
elasticsearch-master-0   0/1     Pending   0          27s
elasticsearch-master-1   0/1     Pending   0          27s
elasticsearch-master-2   0/1     Pending   0          27s
```
After double checking taints, labels and resources I noticed these messages in the events:
```
4m22s Normal ExternalProvisioning persistentvolumeclaim/elasticsearch-master-elasticsearch-master-0 Waiting for a volume to be created either by the external provisioner 'ebs.csi.aws.com' or manually by the system administrator. If volume creation is delayed, please verify that the provisioner is running and correctly registered.
georgiosmastrogiannis@Evas-MacBook-Pro k8s % kubectl get events -n elastic LAST SEEN TYPE REASON OBJECT MESSAGE 4m13s Warning FailedScheduling pod/elasticsearch-master-0 running PreBind plugin "VolumeBinding": binding volumes: context deadline exceeded 4m13s Warning FailedScheduling pod/elasticsearch-master-1 running PreBind plugin "VolumeBinding": binding volumes: context deadline exceeded 4m13s Warning FailedScheduling pod/elasticsearch-master-2 running PreBind plugin "VolumeBinding": binding volumes: context deadline exceeded
```

This gave me the idea, something was wrong with the provisioner, after some research and since I verified my storageClass had ebs.csi.aws.com as provisioner, it seems I didn't have ebs-csi driver running.

```
eksctl utils associate-iam-oidc-provider \
  --region eu-west-1 \
  --cluster wb-task-cluster \
  --approve \
  --profile aws-personal
```
AWS asked me to run this when I tried to create the service account the first time:
`2025-05-28 16:07:54 [!]  no IAM OIDC provider associated with cluster, try 'eksctl utils associate-iam-oidc-provider --region=eu-west-1 --cluster=wb-task-cluster'`

```
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster wb-task-cluster \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --region eu-west-1 \
  --profile aws-personal

eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster wb-task-cluster \
  --region eu-west-1 \
  --force \
  --profile aws-personal
  ```