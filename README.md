<h1> spinnaker-meetup </h1>
<h1> Spinnaker experimental deployment scenario: </h1>

<p> Distributed mode deployment of spinnaker on kubernetes cluster <br>
Using same kubernetes cluster as target cloud provider</p>

<h2>Prerequisite for experimental Spinnaker setup: </h2>

<p> Experimental kubernetes cluster created using either minikube or kubeadm <br>
Kubectl binary and kubeconfig file to access kubernetes cluster <br>
Minio deployment <br>
Halyard deployment </p>

<h2>Steps:</h2>

<h3> A.Setting up required infra:</h3>
<h4> Download kubectl binary from below given link and follow steps:</h4>
<p> 1. curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/linux/amd64/kubectl <br>
2. chmod +x ./kubectl <br>
3. sudo mv ./kubectl /usr/local/bin/kubectl </p>

<h3> Place kubeconfig file in ~/.kube folder named as "config" to access kubernetes cluster</h3>

<h4> Download following files from this repo: </h4>
<p> 1. halyard-deployment.yml <br>
2. minio-deployment.yml </p>

kubectl create namespace <spinnaker-namespace>

kubectl config set-context $(kubectl config current-context) --namespace=<spinnaker-namespace>

kubectl apply -f minio-deployment.yml

kubectl apply -f halyard-deployment.yml

<p> kubectl get pods <br>
(you will notice minio and halyard pods are running. It may be possible that halyard pod will take time to show running as status) </p>

<p> Copy kubeconfig file to halyard pod for making same kubernetes cluster as target cloud provider: <br>
kubectl cp ~/.kube/config <halyard-pod-name>:/tmp/kubeconfig </p>

<h3>  B. Configuring spinnaker services using hal commands provided by halyard service: </h3>

<p> Enter inside halyard pod: <br>
kubectl exec -it <halyard-pod-name> /bin/bash </p>

<p> Try running hal command: <br>
hal config 

Execute hal commands to kick start basic spinnaker services:

<p> hal config version edit --version 1.16.4 <br>
(configuring spinnaker version) </p>

<p> hal config storage s3 edit --endpoint http://minio-service:9000 --access-key-id testingtest --secret-access-key --bucket <bucket-name> <br> 
(will prompt for secret-access-key, provide "testingtest") <br>
hal config storage s3 edit --path-style-access=true <br>
(enabling s3 compatible storage) </p>

<p> echo "spinnaker.s3.versioning: false" > ~/.hal/default/profile/front50-local.yml <br>
(Minio doesn’t support versioning objects, we need to disable it in Spinnaker) </p>

<p> hal config provider kubernetes account add k8s-v2-spin --provider-version v2  --kubeconfig-file /tmp/kubeconfig <br>
hal config provider kubernetes enable <br>
(configuring target cloud provider) </p>


<p> hal config deploy edit --type Distributed --account-name k8s-v2-spin --location <spinnaker-namespace> <br>
(configuring mode of spinnaker deployment) </p>

<p> hal deploy apply <br>
(kick start spinnaker services) </p>

Come out of pod

Verify spinnaker service started:

kubectl get pods

(you will notice pods are coming up of various spinnaker services. It will take sometime for all pods to be in running state)

<h3> C. Exposing deck and gate endpoints locally to access spinnaker: </h3>

kubectl port-forward spin-deck 9000:9000

kubectl port-forward spin-gate 8084:8084

<p> Now check spinnaker UI on browser: <br>
http://localhost:9000 </p>
