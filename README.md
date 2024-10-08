



# K8S practice in EKS:
first 4 way to practice k8s

-> Minikube(local)

->EKS/AKS/GKE(elastic k8s services/Azure/Google)

->killercoda,k8s playground

## Elastic kuberneties services:
Eks is a platform as a services which is provided by AWS to deploye your application over the kuberneties cluster
EKS manage security,scalability,mentainance every thing.

## let's start:
first step to install and setup AWS CLI

```
sudo apt install unzip
url "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
 ls
 cd aws/
 cat install
 vim install
 aws --version
 cd ..
 aws configure
 aws s3 ls aws-gautam-bocket-g/certificate/
 ```
 # How to create EKS cluster using eksctl utility

## Pre-requisites:
- IAM user with **access keys and secret access keys**
- AWSCLI should be configured 
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

- Install **kubectl** 
```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

- Install **eksctl**
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
#
## (configure with the IAM USER)
### the user must have the permission like (application load balancer,EC2,VPC,cloud formation,etc) otherwise give the permission for AdministratorAccess
```
aws configure
```

#
### Steps to create EKS cluster:
### After this command search cloudformation on dash bord and check create or in pogress at the end K8S cluster is ready atomatically by AWS
- Create EKS Cluster
```bash
eksctl create cluster --name=my-cluster \
                      --region=us-west-2 \
                      --version=1.30 \
                      --without-nodegroup
```

- Associate IAM OIDC Provider(helps to authenticate between all inside the cluster)
```bash
eksctl utils associate-iam-oidc-provider \
    --region us-west-2 \
    --cluster my-cluster \
    --approve
```
#

- Create Nodegroup
-  After this command exixution check EKS on dash board and cluster name will show -->Resources(2node)
```bash
eksctl create nodegroup --cluster=my-cluster \
                       --region=us-west-2 \
                       --name=my-cluster \
                       --node-type=t2.medium \
                       --nodes=2 \
                       --nodes-min=2 \
                       --nodes-max=2 \
                       --node-volume-size=29 \
                       --ssh-access \
                       --ssh-public-key=eks-nodegroup-key (put the ssh key which is use for creating EC2)
```
#### Note: Make sure the ssh-public-key "eks-nodegroup-key is available in your aws account"
#
## Now we talk with cluster by the help of kubectl
- Update Kubectl Context
```bash
aws eks update-kubeconfig --region us-west-2 --name my-cluster        (by the help of this command i connect to my local mechine to my cluster)
```
### run it:
```
kubectl get nodes
kubectl get pods --all-namespaces
```



# pod
```
vim pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
by help of this command we create namespace
```
kubectl create namespace nginx
```
if want to delet the namespace nginx then try it:
```
kubectl delete namespace nginx(for cmd)
```
for see  namespace:
```
kubectl get namespace

```




## oterwise in yaml:
```
vim namespace.yml


apiVersion: v1
kind: Namespace
metadata:
  name: nginx
  labels:
    name: nginx
```
```
kunectl apply -f namespace.yml
```
### now some changes required for pod:
```
vim pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```
```
kubectl apply -f pod.yml
kubectl get pods -n nginx
kubectl delete -f pod.yml(this will delete pod.yml)
```
## for deployment:
```
vim deployment.yml:


apiVersion: apps/v1
kind: deployment
metadata:
   name: nginx-deployment
   Namespace: nginx
   labels:
     app: nginx
spec: 
  replicas: 5
  selector:
    matchlabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80  
```
```
kubectl apply --validate=true --dry-run=client --filename=deployment.yml  

kubectl apply -f deployment.yml
kubectl get pods -n nginx

kubectl describe pod (name-of-the-pod) -n nginx
```
```
kubectl get all -n nginx
```
## as we know the containers are running inside the pod so we can't access over the broweser we need application load balancer or a service:
```
vim service.yml


apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: nginx
  labels:
    app: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
```
kubectl apply -f service.yml
kubectl get deployment -n nginx
kubectl get service -n nginx
```








#

- Delete EKS Cluster
```bash
eksctl delete cluster --name=my-cluster --region=us-west-2
```

