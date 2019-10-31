# spinnaker-meetup
Spinnaker experimental deployment scenario:
------------------------------
Distributed mode deployment of spinnaker on kubernetes cluster
Using same kubernetes cluster as target cloud provider

Prerequisite for experimental Spinnaker setup:
-------------------
Experimental kubernetes cluster created using either minikube or kubeadm
Kubectl binary and kubeconfig file to access kubernetes cluster
Minio deployment
Halyard deployment 

Steps:
-------
A.Setting up required infra:
Download kubectl binary from below given link and follow steps:
1. curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/linux/amd64/kubectl
2. chmod +x ./kubectl
3. sudo mv ./kubectl /usr/local/bin/kubectl

Place kubeconfig file in ~/.kube folder named as "config" to access kubernetes cluster

Download following files from given link:
<Link>
1. halyard-deployment.yml
2. minio-deployment.yml

kubectl create namespace <spinnaker-namespace>

kubectl config set-context $(kubectl config current-context) --namespace=<spinnaker-namespace>

kubectl apply -f minio-deployment.yml

kubectl apply -f halyard-deployment.yml

kubectl get pods
(you will notice minio and halyard pods are running. It may be possible that halyard pod will take time to show running as status)

Copy kubeconfig file to halyard pod for making same kubernetes cluster as target cloud provider:
kubectl cp ~/.kube/config <halyard-pod-name>:/tmp/kubeconfig

B. Configuring spinnaker services using hal commands provided by halyard service:

Enter inside halyard pod:
kubectl exec -it <halyard-pod-name> /bin/bash

Try running hal command:
hal --version (or) hal config 

Execute hal commands to kick start basic spinnaker services:

hal config version edit --version 1.16.1
(configuring spinnaker version)

hal config storage s3 edit --endpoint http://minio-service:9000 --access-key-id testingtest --secret-access-key --bucket <bucket-name> 
(will prompt for secret-access-key, provide "testingtest")
hal config storage edit --type s3
(enabling s3 compatible storage)

hal config provider kubernetes account add k8s-v2-spin --provider-version v2  --kubeconfig-file /tmp/kubeconfig
hal config provider kubernetes enable
(configuring target cloud provider)


hal config deploy edit --type Distributed --account-name k8s-v2-spin --location k8s-spin2
(configuring mode of spinnaker deployment)

hal deploy apply
(kick start spinnaker services)

C. Exposing deck and gate endpoints locally to access spinnaker:

kubectl port-forward spin-deck 9000:9000

kubectl port-forward spin-gate 8084:8084

Now check spinnaker UI on browser:
http://localhost:9000
