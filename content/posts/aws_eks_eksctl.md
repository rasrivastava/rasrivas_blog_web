---
title: "Setup an AWS EKS cluster with eksctl"
date: 2022-05-08T18:52:59+05:30
draft: false
tags: ["linux", "filesysystem", "partition", "lvm"]
categories: ["linux", "operating system"]
---

eksctl is a simple CLI tool for creating and managing clusters on EKS - Amazon's managed Kubernetes service for EC2.

It is written in Go, uses CloudFormation, was created by Weaveworks and it welcomes contributions from the community.


[eksctl](https://eksctl.io/)
[Github project](https://github.com/rasrivastava/aws_eks)

# Details Course Content:

- Setup an AWS EKS cluster with eksctl
  - Architecture & Create EKS Cluster with eksctl

- Developing for EKS
  - Understanding the Application Architecture
  - Building from Source
  - Publishing to ECR
  - Deploying to EKS
  - Deploying an Application to EKS

- AWS EKS operations using eksctl
  - NodeGroups & Spot Instances
  - Cluster AutoScaler Theory
  - CloudWatch Logging for EKS Cluster Services
  - CloudWatch Containers Insights for EKS

- Helm Package Manager on EKS
  - Helm installation

- Managing Users & RBAC in EKS
  - Adding an Admin User in EKS
  - Adding a Read-Only User in EKS

- Deploy the Kubernetes Dashboard
  - What is the K8s dashboard ?
  - Install Kubernetes Metrics Server in EKS
  - Deploy & Explore the Kubernetes Dashboard in EKS

- Deploy a stateless sample app
  - Deploy backend & frontend resources
  - Scaling Pods up and down

- Deploy a stateful app - using Amazon EBS
  - Stateful App Intro & Architecture
  - Create namespace
  - Create physical volume
  - Deploy MySQL backend
  - Deployment vs StatefulSet with persistent volumes
  - Deploy Wordpress via Deployment & StatefulSet

- Deploy a stateful app - using Amazon EFS
  - EFS for Kubernetes
  - Enable EFS
  - Create namespace & prepare storage
  - Deploy MySQL backend & Wordpress frontend/li> 

- Fargate on EKS
  - Fargate on EKS
  - Create a Fargate Cluster on EKS
  - Add the Fargate Capability to an existing EKS cluster

# Install the aws cli command if not installed
- For my case its installed
  ```
  [rasrivas@rasrivas eks]$ aws --version
  aws-cli/1.18.69 Python/2.7.17 Linux/5.3.11-100.fc29.x86_64 botocore/1.16.19
  [rasrivas@rasrivas eks]$
  ```
- if not installed (https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
  ```
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  unzip awscliv2.zip
  sudo ./aws/install
  ```
  
  ```
  pip3 install --upgrade --user awscli
  ```
  
  - then you need to configure your credentials
    ```
    $ aws configure
    AWS Access Key ID [None]: XXXXXXXXXXXXXXXXXXXXXX
    AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    Default region name [None]: region-code
    Default output format [None]: json
    ```

# Let list all the AWS EKS cluster running
- Currently no cluster will be running
  ```
  [rasrivas@rasrivas eks]$ aws eks list-clusters
  {
      "clusters": []
  }
  [rasrivas@rasrivas eks]$
  ```

# Limitations of AWS EKS command
- we can't specify multiple instances types for the slave notes
- we can't use the spot type instance
- many more

# Using eksctl command for AWS EKS management
- Installing **eksctl** package (https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
  ```
  [rasrivas@rasrivas eks]$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
  [rasrivas@rasrivas eks]$ sudo mv /tmp/eksctl /usr/local/bin
  [rasrivas@rasrivas eks]$ eksctl version
  0.23.0
  [rasrivas@rasrivas eks]$
  ```

# Deployming AWS EKS cluster

- For this demo we are creating three slave nodes
  - 2 instances = t2.micro
  - 1 instance = t2.small

- For for creating the above cluster we need to create yaml file
  - for this we will use **ClusterConfig** k8s kind
  - these groups of instance of similar types are called as **nodegroups**
  ```
  [rasrivas@rasrivas k8s]$ cat cluster.yml 
  apiVersion: eksctl.io/v1alpha5
  kind: ClusterConfig

  metadata:
    name: eksclusterdemo
    region: ap-southeast-1

  nodeGroups:
    - name: ng-1
      instanceType: t2.micro
      desiredCapacity: 2
    - name: ng-2 
      instanceType: t2.small
      desiredCapacity: 1
  [rasrivas@rasrivas k8s]$
  ```

- Create the cluster using **cluster.yml** file (generally it takes approx 15 minutes for the configuration)
  ```
  [rasrivas@rasrivas k8s]$ eksctl create cluster -f cluster.yml 
  [ℹ]  eksctl version 0.23.0
  [ℹ]  using region ap-southeast-1
  [ℹ]  setting availability zones to [ap-southeast-1a ap-southeast-1c ap-southeast-1b]
  [ℹ]  subnets for ap-southeast-1a - public:192.168.0.0/19 private:192.168.96.0/19
  [ℹ]  subnets for ap-southeast-1c - public:192.168.32.0/19 private:192.168.128.0/19
  [ℹ]  subnets for ap-southeast-1b - public:192.168.64.0/19 private:192.168.160.0/19
  [ℹ]  nodegroup "ng-1" will use "ami-0eca92b64847fff2a" [AmazonLinux2/1.16]
  [ℹ]  nodegroup "ng-2" will use "ami-0eca92b64847fff2a" [AmazonLinux2/1.16]
  [ℹ]  using Kubernetes version 1.16
  [ℹ]  creating EKS cluster "eksclusterdemo" in "ap-southeast-1" region with un-managed nodes
  [ℹ]  2 nodegroups (ng-1, ng-2) were included (based on the include/exclude rules)
  [ℹ]  will create a CloudFormation stack for cluster itself and 2 nodegroup stack(s)
  [ℹ]  will create a CloudFormation stack for cluster itself and 0 managed nodegroup stack(s)
  [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=ap-southeast-1 --cluster=eksclusterdemo'
  [ℹ]  CloudWatch logging will not be enabled for cluster "eksclusterdemo" in "ap-southeast-1"
  [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --region=ap-southeast-1 --cluster=eksclusterdemo'
  [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "eksclusterdemo" in "ap-southeast-1"
  [ℹ]  2 sequential tasks: { create cluster control plane "eksclusterdemo", 2 sequential sub-tasks: { no tasks, 2 parallel sub-tasks: { create nodegroup "ng-1", create nodegroup "ng-2" } } }
  [ℹ]  building cluster stack "eksctl-eksclusterdemo-cluster"
  [ℹ]  deploying stack "eksctl-eksclusterdemo-cluster"
  [ℹ]  building nodegroup stack "eksctl-eksclusterdemo-nodegroup-ng-2"
  [ℹ]  building nodegroup stack "eksctl-eksclusterdemo-nodegroup-ng-1"
  [ℹ]  --nodes-min=2 was set automatically for nodegroup ng-1
  [ℹ]  --nodes-max=2 was set automatically for nodegroup ng-1
  [ℹ]  --nodes-min=1 was set automatically for nodegroup ng-2
  [ℹ]  --nodes-max=1 was set automatically for nodegroup ng-2
  [ℹ]  deploying stack "eksctl-eksclusterdemo-nodegroup-ng-1"
  [ℹ]  deploying stack "eksctl-eksclusterdemo-nodegroup-ng-2"
  [ℹ]  waiting for the control plane availability...
  [✔]  saved kubeconfig as "/home/rasrivas/.kube/config"
  [ℹ]  no tasks
  [✔]  all EKS cluster resources for "eksclusterdemo" have been created
  [ℹ]  adding identity "arn:aws:iam::350276982418:role/eksctl-eksclusterdemo-nodegroup-n-NodeInstanceRole-1S9H477T7LL0B" to auth ConfigMap
  [ℹ]  nodegroup "ng-1" has 0 node(s)
  [ℹ]  waiting for at least 2 node(s) to become ready in "ng-1"
  [ℹ]  nodegroup "ng-1" has 2 node(s)
  [ℹ]  node "ip-192-168-50-40.ap-southeast-1.compute.internal" is ready
  [ℹ]  node "ip-192-168-70-91.ap-southeast-1.compute.internal" is ready
  [ℹ]  adding identity "arn:aws:iam::350276982418:role/eksctl-eksclusterdemo-nodegroup-n-NodeInstanceRole-JJNSN2MGFW1V" to auth ConfigMap
  [ℹ]  nodegroup "ng-2" has 0 node(s)
  [ℹ]  waiting for at least 1 node(s) to become ready in "ng-2"
  [ℹ]  nodegroup "ng-2" has 1 node(s)
  [ℹ]  node "ip-192-168-42-119.ap-southeast-1.compute.internal" is ready
  [ℹ]  kubectl command should work with "/home/rasrivas/.kube/config", try 'kubectl get nodes'
  [✔]  EKS cluster "eksclusterdemo" in "ap-southeast-1" region is ready
  [rasrivas@rasrivas k8s]$ 
  ```

- we can notice few things from the creating the cluster
  - **eksctl** command deploy the infrastructure using AWS cloudformation
    ```
    ℹ]  will create a CloudFormation stack for cluster itself and 2 nodegroup stack(s)
    [ℹ]  will create a CloudFormation stack for cluster itself and 0 managed nodegroup stack(s)
    -------
    -------
    [ℹ]  deploying stack "eksctl-eksclusterdemo-cluster"
    ```
  - **eksctl** in will launch 3 EC instances during this setup for the slave node and a master node will also be created but it will internally managed by AWS
  - **eksctl** will create some more resources such as **VPC, subnet, users, policies** etc
  
- its created succesfully and we can verify it:
  ```
  [rasrivas@rasrivas k8s]$ aws eks list-clusters --region ap-southeast-1
  {
      "clusters": [
          "eksclusterdemo"
      ]
  }
  [rasrivas@rasrivas k8s]$
  ```
  
# Now, we need to connect to cluster

- we can connect to the cluster using kubectl command
  - kubectl command required a **config** file
- we can generate it if it's not available, but for my case its available
  ```
  $ kubectl config view
  ```
  
  ```
  $ aws eks --region ap-southeast-1 update-kubeconfig --name eksclusterdemo
  ```
  
# Checking the cluster

- we can cluster info
  ```
  [rasrivas@rasrivas k8s]$ kubectl cluster-info 
  Kubernetes master is running at https://7A9EFAED3294394617D9F9DF696FE746.gr7.ap-southeast-1.eks.amazonaws.com
  CoreDNS is running at https://7A9EFAED3294394617D9F9DF696FE746.gr7.ap-southeast-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

  To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
  [rasrivas@rasrivas k8s]$
  ```
- we can check the number of nodes present in the pod
  ```
  [rasrivas@rasrivas k8s]$ kubectl get nodes
  NAME                                                STATUS   ROLES    AGE     VERSION
  ip-192-168-42-119.ap-southeast-1.compute.internal   Ready    <none>   9m30s   v1.16.8-eks-fd1ea7
  ip-192-168-50-40.ap-southeast-1.compute.internal    Ready    <none>   10m     v1.16.8-eks-fd1ea7
  ip-192-168-70-91.ap-southeast-1.compute.internal    Ready    <none>   10m     v1.16.8-eks-fd1ea7
  [rasrivas@rasrivas k8s]$
  ```

# Creating name space

- its always a good a separate name space for the cluster related deployment

- checking the all the namespace
  ```
  [rasrivas@rasrivas k8s]$ kubectl get ns
  NAME              STATUS   AGE
  default           Active   22m
  kube-node-lease   Active   22m
  kube-public       Active   22m
  kube-system       Active   22m
  [rasrivas@rasrivas k8s]$
  ```

- creating a new name space
  ```
  [rasrivas@rasrivas k8s]$ kubectl create namespace eksns
  namespace/eksns created
  [rasrivas@rasrivas k8s]$ 
  ```
  
  ```
  [rasrivas@rasrivas k8s]$ kubectl get ns | grep eksns
  eksns             Active   11s
  [rasrivas@rasrivas k8s]$ 
  ```

- by default, **default** name space will be used, it use **eksns** name space by default
  ```
  [rasrivas@rasrivas k8s]$ kubectl config set-context --current --namespace=eksns
  Context "iam-root-account@eksclusterdemo.ap-southeast-1.eksctl.io" modified.
  [rasrivas@rasrivas k8s]$
  ```

- verifying it
  ```
  rasrivas@rasrivas k8s]$ kubectl config view | grep eksns
    namespace: eksns
  [rasrivas@rasrivas k8s]$ 
  ```

#  Creating deployment

- creating deployment
  ```
  [rasrivas@rasrivas k8s]$ kubectl create deployment myweb --image=vimal13/apache-webserver-php
  deployment.apps/myweb created
  [rasrivas@rasrivas k8s]$
  ```
- verfiying it
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pods
  NAME                     READY   STATUS    RESTARTS   AGE
  myweb-79b48fb9f5-5mgjm   1/1     Running   0          33s
  [rasrivas@rasrivas k8s]$ 
  ```

- Lets scale it to 3 PODS
  ```
  [rasrivas@rasrivas k8s]$ kubectl scale deployment myweb --replicas=3
  deployment.apps/myweb scaled
  [rasrivas@rasrivas k8s]$
  ```

- lets check on which nodes the PODS are running
  - we will the three PODS will be running on all the three nodes
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pods -o wide
  NAME                     READY   STATUS    RESTARTS   AGE     IP               NODE                                                NOMINATED NODE   READINESS GATES
  myweb-79b48fb9f5-5mgjm   1/1     Running   0          2m31s   192.168.36.3     ip-192-168-42-119.ap-southeast-1.compute.internal   <none>           <none>
  myweb-79b48fb9f5-9clvj   1/1     Running   0          69s     192.168.45.132   ip-192-168-42-119.ap-southeast-1.compute.internal   <none>           <none>
  myweb-79b48fb9f5-wm65z   1/1     Running   0          69s     192.168.83.60    ip-192-168-70-91.ap-southeast-1.compute.internal    <none>           <none>
  [rasrivas@rasrivas k8s]$
  ```
  
 # ELB setip
 
 - current cluster is by default using **Node Port** service
   - we know **Node Port** service is only reachable to internal network
   - to avail the external connectivity we need to add an **ELB**
 
- Advantages of ELB:
  - A 
  - B

- creating an ELB for the deployment
  ```
  [rasrivas@rasrivas k8s]$ kubectl expose deployment myweb --type=LoadBalancer --port=80
  service/myweb exposed
  [rasrivas@rasrivas k8s]$
  ```
  - at the ......
  
  - deleting everyting
    ```
    [rasrivas@rasrivas k8s]$ kubectl delete all --all
    pod "myweb-79b48fb9f5-5mgjm" deleted
    pod "myweb-79b48fb9f5-9clvj" deleted
    pod "myweb-79b48fb9f5-wm65z" deleted
    service "myweb" deleted
    deployment.apps "myweb" deleted
    replicaset.apps "myweb-79b48fb9f5" deleted
    [rasrivas@rasrivas k8s]$ 
    ```
    
    - auto AWS elb will be deleted

# PVC

```
[rasrivas@rasrivas k8s]$ kubectl create -f pvc.yml 
persistentvolumeclaim/pv-claim-1 created
[rasrivas@rasrivas k8s]$
```

```
[rasrivas@rasrivas k8s]$ kubectl get pvc
NAME         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pv-claim-1   Pending                                      gp2            77s
[rasrivas@rasrivas k8s]$ kubectl get pv
No resources found in eksns namespace.
[rasrivas@rasrivas k8s]$ 
[rasrivas@rasrivas k8s]$ kubectl get sc
NAME            PROVISIONER             AGE
gp2 (default)   kubernetes.io/aws-ebs   44m
[rasrivas@rasrivas k8s]$
```
  
```
[rasrivas@rasrivas k8s]$ kubectl create deployment myweb --image=vimal13/apache-webserver-php

deployment.apps/myweb created
[rasrivas@rasrivas k8s]$ 
[rasrivas@rasrivas k8s]$ 
[rasrivas@rasrivas k8s]$ 
[rasrivas@rasrivas k8s]$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myweb-79b48fb9f5-xqjzj   1/1     Running   0          8s
[rasrivas@rasrivas k8s]$
```
  
```
[rasrivas@rasrivas k8s]$ eksctl delete cluster -f cluster.yml 
[ℹ]  eksctl version 0.23.0
[ℹ]  using region ap-southeast-1
[ℹ]  deleting EKS cluster "eksclusterdemo"
[ℹ]  deleted 0 Fargate profile(s)
[✔]  kubeconfig has been updated
[ℹ]  cleaning up LoadBalancer services
[ℹ]  2 sequential tasks: { 2 parallel sub-tasks: { delete nodegroup "ng-2", delete nodegroup "ng-1" }, delete cluster control plane "eksclusterdemo" [async] }
[ℹ]  will delete stack "eksctl-eksclusterdemo-nodegroup-ng-1"
[ℹ]  waiting for stack "eksctl-eksclusterdemo-nodegroup-ng-1" to get deleted
[ℹ]  will delete stack "eksctl-eksclusterdemo-nodegroup-ng-2"
[ℹ]  waiting for stack "eksctl-eksclusterdemo-nodegroup-ng-2" to get deleted
[ℹ]  will delete stack "eksctl-eksclusterdemo-cluster"
[✔]  all cluster resources were deleted
[rasrivas@rasrivas k8s]$
```
  
# Testing ELB

- creating a new cluster
  - Three **t2.micro**
  - In **ap-south-1** (Mumbai) region
  ```
  [rasrivas@rasrivas k8s]$ cat cluster.yml 
  apiVersion: eksctl.io/v1alpha5
  kind: ClusterConfig

  metadata:
    name: eksclusterdemo
    region: ap-south-1

  nodeGroups:
    - name: ng-1
      instanceType: t2.micro
      desiredCapacity: 3
  [rasrivas@rasrivas k8s]$
  ```
- https://docs.google.com/document/d/1vAfL0eQCXyIBC260HUMULrf10lVtZL6tqBhplyh1gR8/edit

- we will create a deployment
  ```
  [rasrivas@rasrivas k8s]$ kubectl create deployment myweb --image=vimal13/apache-webserver-php
  deployment.apps/myweb created
  [rasrivas@rasrivas k8s]$
  ```
  
- check the PODS are created
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pods
  NAME                     READY   STATUS    RESTARTS   AGE
  myweb-79b48fb9f5-q4xww   1/1     Running   0          15s
  [rasrivas@rasrivas k8s]$
  ```
- check the IP on which Node the POS is running
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pods -o wide
  NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE                                           NOMINATED NODE   READINESS GATES
  myweb-79b48fb9f5-q4xww   1/1     Running   0          31s   192.168.51.165   ip-192-168-39-58.ap-south-1.compute.internal   <none>           <none>
  [rasrivas@rasrivas k8s]$ 
  ```
- we will scale for the PODS to 3 for HA
  ```
  [rasrivas@rasrivas k8s]$ kubectl scale deployment myweb --replicas=3
  deployment.apps/myweb scaled
  [rasrivas@rasrivas k8s]$ 
  ```

- verfiry it
  ```
  [rasrivas@rasrivas k8s]$ kubectl get all
  NAME                         READY   STATUS              RESTARTS   AGE
  pod/myweb-79b48fb9f5-6dnxf   1/1     Running             0          6s
  pod/myweb-79b48fb9f5-klmxv   0/1     ContainerCreating   0          7s
  pod/myweb-79b48fb9f5-q4xww   1/1     Running             0          2m30s

  NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  service/kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   12m

  NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/myweb   2/3     3            2           2m31s

  NAME                               DESIRED   CURRENT   READY   AGE
  replicaset.apps/myweb-79b48fb9f5   3         3         2       2m31s
  [rasrivas@rasrivas k8s]$ 
  [rasrivas@rasrivas k8s]$
  ```
- creating a ELB for public access [mention the reason and the aws elb screenshot] elb_13.png
  ```
  [rasrivas@rasrivas k8s]$ kubectl expose deployment myweb  --type=LoadBalancer --port=80
  service/myweb exposed
  [rasrivas@rasrivas k8s]$ 
  ```

- verfiy thorugh CLI
  ```
  [rasrivas@rasrivas k8s]$ kubectl get svc
  NAME         TYPE           CLUSTER-IP     EXTERNAL-IP                                                               PORT(S)        AGE
  kubernetes   ClusterIP      10.100.0.1     <none>                                                                    443/TCP        18m
  myweb        LoadBalancer   10.100.9.147   a76256b92a4654631a9daba27eff342c-302986886.ap-south-1.elb.amazonaws.com   80:32062/TCP   4m46s
  [rasrivas@rasrivas k8s]$ 
  ```
  
  - we can see **EXTERNAL-IP **
    - **a76256b92a4654631a9daba27eff342c-302986886.ap-south-1.elb.amazonaws.com**
    
    ```
    [rasrivas@rasrivas k8s]$ kubectl describe service/myweb
    Name:                     myweb
    Namespace:                default
    Labels:                   app=myweb
    Annotations:              <none>
    Selector:                 app=myweb
    Type:                     LoadBalancer
    IP:                       10.100.9.147
    LoadBalancer Ingress:     a76256b92a4654631a9daba27eff342c-302986886.ap-south-1.elb.amazonaws.com
    Port:                     <unset>  80/TCP
    TargetPort:               80/TCP
    NodePort:                 <unset>  32062/TCP
    Endpoints:                192.168.34.88:80,192.168.51.165:80,192.168.95.135:80
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:
      Type    Reason                Age    From                Message
      ----    ------                ----   ----                -------
      Normal  EnsuringLoadBalancer  7m14s  service-controller  Ensuring load balancer
      Normal  EnsuredLoadBalancer   7m12s  service-controller  Ensured load balancer
    [rasrivas@rasrivas k8s]$ 
    ```
 
- also we can verify using this URL
  - elb_14.png [attach]


- deleting this entire deployemnt
  ```
  [rasrivas@rasrivas k8s]$ kubectl delete all --all
  pod "myweb-79b48fb9f5-6dnxf" deleted
  pod "myweb-79b48fb9f5-klmxv" deleted
  pod "myweb-79b48fb9f5-q4xww" deleted
  service "kubernetes" deleted
  service "myweb" deleted
  deployment.apps "myweb" deleted
  replicaset.apps "myweb-79b48fb9f5" deleted
  [rasrivas@rasrivas k8s]$
  ```
  
- check AWS ELB thorugh console
   - attache IMAGE: elb_15.png

# Testing PVC

- creating deplloyment
   ```
   [rasrivas@rasrivas k8s]$ kubectl create deployment myweb --image=vimal13/apache-webserver-php
  deployment.apps/myweb created
  [rasrivas@rasrivas k8s]$ 
  [rasrivas@rasrivas k8s]$ 
  ```
  
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pods -o wide
  NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE                                            NOMINATED NODE   READINESS GATES
  myweb-79b48fb9f5-hzsdg   1/1     Running   0          40s   192.168.95.135   ip-192-168-78-153.ap-south-1.compute.internal   <none>           <none>
  [rasrivas@rasrivas k8s]$
   ```
- creating a ELB
  ```
  [rasrivas@rasrivas k8s]$ kubectl expose deployment myweb  --type=LoadBalancer --port=80
  service/myweb exposed
  [rasrivas@rasrivas k8s]$
  ```

- logging to the container
  ```
  [rasrivas@rasrivas k8s]$ kubectl exec -it myweb-79b48fb9f5-hzsdg bash
  kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
  [root@myweb-79b48fb9f5-hzsdg /]# 
  [root@myweb-79b48fb9f5-hzsdg /]# 
  [root@myweb-79b48fb9f5-hzsdg /]# cat /var/www/html/index.php 
  <body bgcolor='aqua'>
  <pre>

  <?php

  print "welcome to vimal web server for testing";


  print `ifconfig`;

  ?>

  </pre>
  [root@myweb-79b48fb9f5-hzsdg /]#
  ```

- crreating a index.html file
   ```
   [rasrivas@rasrivas k8s]$ cat index.html 
    Hi this persistent data testing EKS cluster[rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$
   ```
   - and then we copy this file to the container, it will basically overwite the index.htm file

- now if use the ELB dns name (http://a9d60e896fdd4472fbc4bcf313bee12d-340911159.ap-south-1.elb.amazonaws.com/)
  - attache 15 image
  - we can see the updated code
  - but once this POD is deleted this updated code will be deleted
  - when the "deployment" with the help of "rs" will launch the new POD it will use the old data
    ```
    [rasrivas@rasrivas k8s]$ kubectl get pods
    NAME                     READY   STATUS    RESTARTS   AGE
    myweb-79b48fb9f5-hzsdg   1/1     Running   0          10m
    [rasrivas@rasrivas k8s]$ kubectl get deployments.apps 
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    myweb   1/1     1            1           11m
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$ kubectl get rs
    NAME               DESIRED   CURRENT   READY   AGE
    myweb-79b48fb9f5   1         1         1       11m
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$ kubectl delete pod myweb-79b48fb9f5-hzsdg
    pod "myweb-79b48fb9f5-hzsdg" deleted
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$
    ```
 
 - so, rs will create new POD
   ```
   [rasrivas@rasrivas k8s]$ kubectl get pods
    NAME                     READY   STATUS    RESTARTS   AGE
    myweb-79b48fb9f5-2jf86   1/1     Running   0          8s
    [rasrivas@rasrivas k8s]$ 
   ```
    
- if we check the ELB url again, it will use the old data again (http://a9d60e896fdd4472fbc4bcf313bee12d-340911159.ap-south-1.elb.amazonaws.com/)
  - attach picture eks_16.png

- [Explain PVC, PV and SC ??????????????????????????]

- checkig SC, by defualt it uses "gp2"
  ```
  [rasrivas@rasrivas k8s]$ kubectl get sc
  NAME            PROVISIONER             AGE
  gp2 (default)   kubernetes.io/aws-ebs   39m
  [rasrivas@rasrivas k8s]$
  ```

- creating pvc
  ```
  [rasrivas@rasrivas k8s]$ cat pvc.yml 
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: pv-claim-1

  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 5Gi
  [rasrivas@rasrivas k8s]$ 
  ```

- checking it
  ```
  [rasrivas@rasrivas k8s]$ kubectl create -f pvc.yml 
  persistentvolumeclaim/pv-claim-1 created
  [rasrivas@rasrivas k8s]$
  ```

- descibing it ==> [Pending] ===> REASON
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pvc
  NAME         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  pv-claim-1   Pending                                      gp2            17s
  [rasrivas@rasrivas k8s]$ 
  ```
    

- decring the gp2
  ```
  [rasrivas@rasrivas k8s]$ kubectl describe sc gp2
  Name:            gp2
  IsDefaultClass:  Yes
  Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"gp2"},"parameters":{"fsType":"ext4","type":"gp2"},"provisioner":"kubernetes.io/aws-ebs","volumeBindingMode":"WaitForFirstConsumer"}
  ,storageclass.kubernetes.io/is-default-class=true
  Provisioner:           kubernetes.io/aws-ebs
  Parameters:            fsType=ext4,type=gp2
  AllowVolumeExpansion:  <unset>
  MountOptions:          <none>
  ReclaimPolicy:         Delete
  VolumeBindingMode:     WaitForFirstConsumer
  Events:                <none>
  [rasrivas@rasrivas k8s]$
  ```
  - its mentioned => **"volumeBindingMode":"WaitForFirstConsumer"**
 
- now, we need to the **myweb** deployment file
  ```
  [rasrivas@rasrivas k8s]$ kubectl edit deploy myweb
  ```
  
  - add two things:
    ```
    - name: web-vol-1
        persistentVolumeClaim:
          claimName: pv-claim-1
    ```
    
    ```
    volumeMounts:
        - mountPath: /var/www/html
          name: web-vol-1
    ```
  
  - where it will be done
    ```
    spec:
      volumes:
      - name: web-vol-1
        persistentVolumeClaim:
          claimName: pv-claim-1
      containers:
      - image: vimal13/apache-webserver-php
        volumeMounts:
        - mountPath: /var/www/html
          name: web-vol-1
        imagePullPolicy: Always
        name: apache-webserver-php
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
    ```
    
- once is above code is completed, save the file and exit, we will not error
  ```
  [rasrivas@rasrivas k8s]$ kubectl edit deploy myweb 
  deployment.apps/myweb edited
  [rasrivas@rasrivas k8s]$
  ```

- Now, if we see the PVS status, we will see as **Bound**
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pvc
  NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
  pv-claim-1   Bound    pvc-3faa7e80-ec31-4630-aa54-42b5be1139ef   5Gi        RWO            gp2            9m48s
  [rasrivas@rasrivas k8s]$ 
  ```
  - also, we not a notice one thing, once this is done, we will see a new EBS volume of 5 gib
    - picture: eks_15.png

- now, if we descibe our pOD will be seeing, its using PVC
  ```
  [rasrivas@rasrivas k8s]$ kubectl describe pods myweb-849b8d9fd7-7k5wn
  Name:         myweb-849b8d9fd7-7k5wn
  Namespace:    default
  Priority:     0
  Node:         ip-192-168-78-153.ap-south-1.compute.internal/192.168.78.153
  Start Time:   Sun, 05 Jul 2020 17:31:38 +0530
  Labels:       app=myweb
                pod-template-hash=849b8d9fd7
  Annotations:  kubernetes.io/psp: eks.privileged
  Status:       Running
  IP:           192.168.95.135
  IPs:
    IP:           192.168.95.135
  Controlled By:  ReplicaSet/myweb-849b8d9fd7
  Containers:
    apache-webserver-php:
      Container ID:   docker://990689f692a1cc607c140c157590eb46c6dc4ace0c054b05a55a18b0f2b981a5
      Image:          vimal13/apache-webserver-php
      Image ID:       docker-pullable://vimal13/apache-webserver-php@sha256:faed0a5afaf9f04b6915d73f7247f6f5a71db9274ca44118d38f4601c0080a91
      Port:           <none>
      Host Port:      <none>
      State:          Running
        Started:      Sun, 05 Jul 2020 17:31:59 +0530
      Ready:          True
      Restart Count:  0
      Environment:    <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from default-token-prrv2 (ro)
        /var/www/html from web-vol-1 (rw)
  Conditions:
    Type              Status
    Initialized       True 
    Ready             True 
    ContainersReady   True 
    PodScheduled      True 
  Volumes:
    web-vol-1:
      Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
      ClaimName:  pv-claim-1
      ReadOnly:   false
    default-token-prrv2:
      Type:        Secret (a volume populated by a Secret)
      SecretName:  default-token-prrv2
      Optional:    false
  QoS Class:       BestEffort
  Node-Selectors:  <none>
  Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                   node.kubernetes.io/unreachable:NoExecute for 300s
  Events:
    Type    Reason                  Age    From                                                    Message
    ----    ------                  ----   ----                                                    -------
    Normal  Scheduled               6m6s   default-scheduler                                       Successfully assigned default/myweb-849b8d9fd7-7k5wn to ip-192-168-78-153.ap-south-1.compute.internal
    Normal  SuccessfulAttachVolume  5m59s  attachdetach-controller                                 AttachVolume.Attach succeeded for volume "pvc-3faa7e80-ec31-4630-aa54-42b5be1139ef"
    Normal  Pulling                 5m47s  kubelet, ip-192-168-78-153.ap-south-1.compute.internal  Pulling image "vimal13/apache-webserver-php"
    Normal  Pulled                  5m45s  kubelet, ip-192-168-78-153.ap-south-1.compute.internal  Successfully pulled image "vimal13/apache-webserver-php"
    Normal  Created                 5m45s  kubelet, ip-192-168-78-153.ap-south-1.compute.internal  Created container apache-webserver-php
    Normal  Started                 5m45s  kubelet, ip-192-168-78-153.ap-south-1.compute.internal  Started container apache-webserver-php
  [rasrivas@rasrivas k8s]$
  ```
  
  ```
  Volumes:
  web-vol-1:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  pv-claim-1
    ReadOnly:   false
  default-token-prrv2:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-prrv2
    Optional:    false
  ```
   
- if we check our ELB nothing will be there now (http://a9d60e896fdd4472fbc4bcf313bee12d-340911159.ap-south-1.elb.amazonaws.com/)
  - PICUTURE eks_17.png

- now, if we copy data to the PODS, it will e permanent as it will be saved on the EBS vol
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pods
  NAME                     READY   STATUS    RESTARTS   AGE
  myweb-849b8d9fd7-7k5wn   1/1     Running   0          9m30s
  [rasrivas@rasrivas k8s]$ 
  [rasrivas@rasrivas k8s]$ 
  [rasrivas@rasrivas k8s]$ 
  [rasrivas@rasrivas k8s]$ kubectl cp index.html myweb-849b8d9fd7-7k5wn:/var/www/html/index.html
  [rasrivas@rasrivas k8s]$ 
  [rasrivas@rasrivas k8s]$ 
  ```

- now, again, we check the ELB dns, we will see the data
  - PICUTE 18
  - also we can verify by CLI using curl
    ```
     [rasrivas@rasrivas k8s]$ curl a9d60e896fdd4472fbc4bcf313bee12d-340911159.ap-south-1.elb.amazonaws.com
      Hi this persistent data testing EKS cluster[rasrivas@rasrivas k8s]$ 
    ```

- now if we delete the POS, and again "rs" using deployemnt create the POD this data will be availabe
  ```
  [rasrivas@rasrivas k8s]$ kubectl delete pods myweb-849b8d9fd7-7k5wn
  pod "myweb-849b8d9fd7-7k5wn" deleted
  [rasrivas@rasrivas k8s]$ 
  ```
  
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pods
  NAME                     READY   STATUS              RESTARTS   AGE
  myweb-849b8d9fd7-p7bch   1/1     Running             0          34s
  ```

- now, we again see the ELB url, we can see the same data
  ```
  [rasrivas@rasrivas k8s]$ curl a9d60e896fdd4472fbc4bcf313bee12d-340911159.ap-south-1.elb.amazonaws.com
  Hi this persistent data testing EKS cluster[rasrivas@rasrivas k8s]$ 
  ```

# we can also change the storage class

- we want to change the storage type from **gp2** to **io1**

- by deault **RECLAIM POLICY** us **Delete** and **STORAGECLASS** is **gp2**
  ```
  [rasrivas@rasrivas k8s]$ kubectl get pv
  NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
  pvc-3faa7e80-ec31-4630-aa54-42b5be1139ef   5Gi        RWO            Delete           Bound    default/pv-claim-1   gp2                     21m
  [rasrivas@rasrivas k8s]$ 
  ```

- deleting old deployment
  ```
  [rasrivas@rasrivas k8s]$ kubectl delete deployments.apps --all
  deployment.apps "myweb" deleted
  [rasrivas@rasrivas k8s]$ 
  ```
 
 - delete the PVC
   ```
   [rasrivas@rasrivas k8s]$ kubectl get pvc
    NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    pv-claim-1   Bound    pvc-3faa7e80-ec31-4630-aa54-42b5be1139ef   5Gi        RWO            gp2            31m
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$ kubectl delete pvc --all
    persistentvolumeclaim "pv-claim-1" deleted
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$ 
    [rasrivas@rasrivas k8s]$ kubectl get pvc
    No resources found in default namespace.
    [rasrivas@rasrivas k8s]$ 
   ```
   
- here I am creating type with **io1** and also we will use **Reclaim policy as retain**, volume will not be deleted when the entire cluster is deleted
  ```
  [rasrivas@rasrivas k8s]$ cat sc.yml 
  apiVersion: storage.k8s.io/v1
  kind: StorageClass

  metadata:
    name: ekssc1
  provisioner: kubernetes.io/aws-eks
  parameters:
    type: io1
  reclaimPolicy: Retain
  [rasrivas@rasrivas k8s]$ 
  ```
- creating it
  ```
  [rasrivas@rasrivas k8s]$ kubectl create -f sc.yml 
  storageclass.storage.k8s.io/ekssc1 created
  [rasrivas@rasrivas k8s]$ 
  [rasrivas@rasrivas k8s]$ 
  ```

- now we will see we will have two SC, but the defualt is **gp2**
  ```
  [rasrivas@rasrivas k8s]$ kubectl get sc
  NAME            PROVISIONER             AGE
  ekssc1          kubernetes.io/aws-eks   6s
  gp2 (default)   kubernetes.io/aws-ebs   74m
  [rasrivas@rasrivas k8s]$ 
  ```

- now, we will say our pvc config file to use ekssc1 SC
  ```
  [rasrivas@rasrivas k8s]$ cat pvc.yml 
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: pv-claim-1

  spec:
    storageClassName: ekssc1
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 5Gi
  [rasrivas@rasrivas k8s]$
  ```
- creating PVC which will use ekssc1 SC
  ```
  [rasrivas@rasrivas k8s]$ kubectl create -f pvc.yml 
  persistentvolumeclaim/pv-claim-1 created
  [rasrivas@rasrivas k8s]$ 
  ```
