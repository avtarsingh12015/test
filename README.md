# Setting up Ansible Service Broker on K8s (Custom image to fix the Secrets binding) – Developer Guide



## Running locally on Minikube



### Prerequisite:



### 1 ) Hyperkit on mac OS/VirtualBox on windows & Docker



Setup:



### 2) Kubernetes Setup command:
```
brew install kubectl minikube
```


### 3) Run Kube command:


```
minikube start –bootstrapper kubeadm —memory 2048 —cpus 2 –vm-driver=hyperkit
```


### 4) Helm Setup command:


```
brew install kubernetes-helm
```


### 5) Run Tiller Command


```
helm init
```


### 6) Making sure tiller is up, command is:


```
until kubectl get pods -n kube-system -l name=tiller | grep 1/1; do sleep 1; done

EX:

tiller-deploy-5cbc8878f7-szjqg   1/1       Running   0          1d
```


### 7) Create cluster role binding: Command:

```
kubectl create clusterrolebinding tiller-cluster-admin --clusterrole=cluster-admin -- serviceaccount=kube-system:default
```

### 8) Patch tiller for automountServiceAccountToken, so that service catalog can get installed: Command:
```
kubectl -n kube-system patch deployment tiller-deploy -p '{"spec": {"template": {"spec": {"automountServiceAccountToken": true}}}}'
```

### 9) Check if tiller is running or not: command:
```
until kubectl get pods -n catalog -l app=catalog-catalog-apiserver | grep 2/2; do sleep 1; done

EX:

catalog-catalog-apiserver-68c47c9d6-hn9fh   2/2       Running   2          1d
```


### 10) Adding repo for Service catalog Command:

```
helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
```

### 11) Installing Service Catalog Command:

```
helm install svc-cat/catalog --name catalog --namespace catalog
```

### 12) Install ansible Service broker Command:

```
git clone https://github.com/openshift/ansible-service-broker.git

then cd ansible-service- broker/

./scripts/run_latest_k8s_build.sh
```

### 13) Check Ansible Service broker is running or not. Command

```
kubectl get pods -n ansible-service-broker

EX:

NAME                                      READY     STATUS    RESTARTS   AGE

ansible-service-broker-6d4c49f7b6-77sfr   1/1       Running   0          1d
```


### 14) Svcat installation Command

```

curl -sLO https://download.svcat.sh/cli/latest/darwin/amd64/svcat

chmod +x ./svcat

mv ./svcat /usr/local/bin/

svcat version --client

```

### 15) Clone DIND Yaml and run it inside minikube

```
git clone http://github.com/kashif123456/dind

cd dind

kubectl create –f deploy.yaml

```

### 16) Go inside Dind pod, pull patched image and change that with the image to be used by Ansible Service Broker

```
kubectl exec –it dind-3 bash

docker tag devops95010/rds-postgres-apb:stable3 ansibleplaybookbundle/rds-postgres-apb:latest
```

### 17) Check all the services are running or not. Command:

```
kubectl get po —all-namespaces

EX:

NAMESPACE                        NAME                                                  READY     STATUS    RESTARTS   AGE

ansible-service-broker           ansible-service-broker-6d4c49f7b6-77sfr               1/1       Running   0          1d

catalog                          catalog-catalog-apiserver-68c47c9d6-hn9fh             2/2       Running   2          1d

catalog                          catalog-catalog-controller-manager-674c686ff6-dv4g6   1/1       Running   8          1d

default                          dind-3                                                1/1       Running   0          1d

kube-system                      etcd-minikube                                         1/1       Running   0          1d

kube-system                      kube-addon-manager-minikube                           1/1       Running   0          1d

kube-system                      kube-apiserver-minikube                               1/1       Running   0          1d

kube-system                      kube-controller-manager-minikube                      1/1       Running   0          1d

kube-system                      kube-dns-86f4d74b45-9sbs7                             3/3       Running   0          1d

kube-system                      kube-proxy-hmwwr                                      1/1       Running   0          1d

kube-system                      kube-scheduler-minikube                               1/1       Running   0          1d

kube-system                      kubernetes-dashboard-5498ccf677-649zc                 1/1       Running   0          1d

kube-system                      storage-provisioner                                   1/1       Running   0          1d

kube-system                      tiller-deploy-5cbc8878f7-szjqg                        1/1       Running   0          1d

```

### 18) Provision RDS

Use svcat to provision the rds by providing ‘rds name’, ‘vpc security group’, ‘aws access key’, db password and aws secret key

```
svcat provision rds1234 --class dh-rds-postgres-apb --plan default -p subnet=default -p vpc_security_groups=sg-07fbd32ca76xxxx-p region=us-east-1 -p aws_access_key=xxxxx -p db_password=test1234 -p aws_secret_key=xxxxxxxx



EX:

  Name:        rds33                

  Namespace:   default              

  Status:                           

  Class:       dh-rds-postgres-apb  

  Plan:        default              



Parameters:

  aws_access_key: xxxxxxxxxxxxxx

  aws_secret_key: xxxxxxxxxxxxxxxxxxxxxxxx

  db_password: test1234

  region: us-east-1

  subnet: default

  vpc_security_groups: sg-0xxxxxxxxxxxxxxxxxxxxx



19) Confirm that pod shows as running while instance is being provisioned. command:

kubectl get po —all-namespaces |grep -ie running

Note: a new pod would be there with syntax like ‘dh-rds-postgres-apb-prov-xxxxx’

EX:

NAMESPACE                        NAME                                                  READY     STATUS    RESTARTS   AGE

dh-rds-postgres-apb-prov-kqg8s   apb-1070eb76-0805-40fb-bb7b-a8c54209893b              1/1       Running   0          2m



Command to check the logs:

kubectl get logs -n dh-rds-postgres-apb-prov-kqg8s apb-1070eb76-0805-40fb-bb7b-a8c54209893b

EX:



PLAY [Deploy rds-apb to openshift] *********************************************



TASK [Gathering Facts] *********************************************************

ok: [localhost]



TASK [ansible.kubernetes-modules : Install latest openshift client] ************

skipping: [localhost]



TASK [ansibleplaybookbundle.asb-modules : debug] *******************************

skipping: [localhost]



TASK [rds-apb-openshift : set_fact] ********************************************

ok: [localhost]



TASK [rds-apb-openshift : rds] *************************************************

```

### 20) Bind Credentials

```
svcat bind rds23
```

### 21) check the binding Commands

```
kubectl describe secret rds23

kubectl get secret rds23 –o yaml
```

### 22) To get all the bindings command:

```
svcat get bindings

Ex:

GC02QLB1XG8WPE:~ kashif$ svcat get bindings

  NAME    NAMESPACE   INSTANCE   STATUS  

+-------+-----------+----------+--------+

  rds16   default     rds16      Ready   

  rds17   default     rds17      Ready   

  rds18   default     rds18      Ready   

  rds23   default     rds23      Ready   

```

### 23) To unbind the Secret

```
svcat unbind rds23
```
