# KinD
Quick Start to run and deploy sample project by using kind
Just in 6 Steps:
# 1.Installation
```bash 
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
chmod +x ./kind
mv ./kind /usr/local/bin/
```
# 2.Creating a Cluster
Creating a Kubernetes cluster is as simple as
```bash 
kind create cluster
```
# Checking Clusters
```bash
kind get clusters
ls ~/.kube
##Docker images Added node docker ps Added  kind-control-plane
kubectl cluster-info --context kind-kind 
alias k='kubectl' added to /root/.bashrc then exec bash 

k cluster-info
k get nodes
k get pods -A
grep server ~/.kube/config    #same as container API server
docker exec -it kind-control-plane bash
INNER CONTAINER: {
crictl ps
ls /etc/kubernetes/
ls /etc/kubernetes/manifests/
}
k get nodes -o wide
k config get-contexts
k get nodes --context kind-kind     #USE WHEN U HAVE MORE THAN ONE CLUSTER
#TO SWITCH BETWWEN CLUSTERS: k config use-context kind-kind

```
# 3.Creating cluster with a worker
Delete old cluster and run :
```bash 
vim /tmp/kind.yml
# a cluster with a control-plane nodes and a worker
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
```
Then:
```bash
kind create cluster --config  /tmp/kind.yml
```
# Checking Clusters
```diff
docker ps
docker exec -it 
INNER CONTAINER:
{ ps -ef
cat /usr/local/etc/haproxy/haproxy.cfg
ping kind-control-plane
ping kind-worker
}
docker network ls    #THEY'R IN THE SAME CLUSTER THEN SAME NETWORK 
#IF YOU INSPECT FOR EXAMPLE external load blancer / kind-control-pplane you can see network

k get nodes 
k get pods -A
##### if you want specific verion of kuberneties , you have to :
### kind create cluster --image kindest/node:v1.19.7@sha256:dsssssfssbal balah
### or you can write image: imagename in your config.yml file /tmp/kind.yml WHAT image for you worker or control-planes

k get sc 
```
# 4.Add a Loadblancer (Metallb)
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/master/manifests/namespace.yaml
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)" 
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/master/manifests/metallb.yaml
```
Wait for Ready state:
```bash
kubectl get pods -n metallb-system --watch
```
# 5.Configure Metallb
Setup address pool used by loadbalancers:
To complete layer2 configuration, we need to provide metallb a range of IP addresses it controls. We want this range to be on the docker kind network.
```bash
docker network inspect -f '{{.IPAM.Config}}' kind
```
##When you finding out docker network range : my range was 172.18.0.1/16
```bash
vim /tmp/metallb-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.18.255.200-172.18.255.250
```
Then:
```bash
k create -f /tmp/metallb-configmap.yaml
```
Wait for metallb pods to have a status of Running
```bash
kubectl get pods -n metallb-system --watch
```
# 6.TESTINNG TIME
```bash
k create deploy nginx --image nginx
k get pods  --watch
k get deployments --watch
k describe deployment nginx

k expose deploy nginx --port 80 --type LoadBalancer
```
Checking with
```bash
k get svc
k get all
curl 172.18.255.200
```
If you want to access to your pod (deployment) run this command to port-forwarding your app :
```bash 
kubectl port-forward service/nginx --address 0.0.0.0 80:80
```
Here you go ;)


