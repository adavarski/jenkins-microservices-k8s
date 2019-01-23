# microservises app: bluepost (ui, post, mongodb)

```
Bluepost: Reddit-like application written in Ruby (Sinatra) created for learning purposes

The functionality is simple. The application allows you to share interesting links with the other, as well as vote and comment on them.

Application requirements:

Ruby v2.3
Bundler
MongoDB v3.2

```

We will build, test, release, deploy with Jenkins CI/CD to k8s (minikube):

Jenkins Setup

minikube ssh ->  get /etc/kubernetes/admin.conf and change localhost to 192.168.99.110 (minikube ip) 
kubectl --kubeconfig=./admin.conf get po

## Test microservices locally:
```
$ mv docker-compose.yml docker-compose.yml.dockerhub
$ cp docker-compose.yml.ORIG docker-compose.yml
$ docker-compose up --build
```
Browser: http://localhost:9292

$ docker rm $(docker ps -a -q)

## Deploy to minikube using kubectl:

```
$ mkdir tmp
$ cp docker-compose.yml tmp

Setup dockerhub images and setup docker-compose version: "2":

$ cat docker-compose.yml
version: '2'

services:
  mongo:
    image: mongo:3.4

  post:
    image: davarski/post
    environment:
      - POST_DATABASE_HOST=mongo
    depends_on:
      - mongo

  ui:
    image: davarski/ui
    environment:
      - POST_SERVICE_HOST=post
      - POST_SERVICE_PORT=5000
    ports:
      - 9292:9292
    depends_on:
      - post

Generete k8s kubectl files:

$ rm docker-compose.yml
$ kompose convert
WARN Unsupported depends_on key - ignoring        
INFO Kubernetes file "mongo-service.yaml" created 
INFO Kubernetes file "post-service.yaml" created  
INFO Kubernetes file "ui-service.yaml" created    
INFO Kubernetes file "mongo-deployment.yaml" created 
INFO Kubernetes file "post-deployment.yaml" created 
INFO Kubernetes file "ui-deployment.yaml" created 

If you want you can setup images based on GitLab build:
Example: davarski/post:master_43e751df70a228f3f6ceee088b1db975aa06b275

DockerHub Tags for image davarski/post:
latest
2.0.0
master_43e751df70a228f3f6ceee088b1db975aa06b275
master_0439d6744484ad8cd0862e69a5022c54ec735564
master_14b278978c4e5122678de0dd5efb791eac100d40

Setup nodeport for UI

Add NodePort and nodePort

$ cat ui-service.yaml 
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.1.0 (36652f6)
  creationTimestamp: null
  labels:
    io.kompose.service: ui
  name: ui
spec:
  type: NodePort
  ports:
  - name: "9292"
    port: 9292
    targetPort: 9292
    nodePort: 30623
  selector:
    io.kompose.service: ui
status:
  loadBalancer: {}
  
  kubectl apply :
  
$ export KUBECONFIG=~/.kube/config
$ kubectl create -f .
deployment.extensions/mongo created
service/mongo created
deployment.extensions/post created
service/post created
deployment.extensions/ui created
service/ui created

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
mongo-6bc479dbf6-2867b   1/1     Running   0          19m
post-659cbd6d8b-rdqxh    1/1     Running   0          19m
ui-7774c64655-fqp5c      1/1     Running   0          19m

$ minikube ip
192.168.99.100

Browser: http://192.168.99.100:30623

Clean:
$ kubectl delete --all svc --namespace=default
$ kubectl delete --all deployments --namespace=default
```

## Deploy to minikube using Helm:

```
$ helm init
$ helm --kube-context=minikube install --set usePassword=false --name mongodb  stable/mongodb
$ helm upgrade post post/charts/post --install --kube-context=minikube --set image.tag=latest
$ helm upgrade ui ui/charts/ui --install --kube-context=minikube --set image.tag=latest

$ kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
mongodb-69fc4cddff-zqdkk   1/1     Running   0          11m
post-65c8d5c598-gjlpb      1/1     Running   0          10m
ui-7d95c8ff49-dsjvr        1/1     Running   0          9m
ui-7d95c8ff49-hg7kb        1/1     Running   0          9m

$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          1h
mongodb      ClusterIP   10.101.221.251   <none>        27017/TCP        3m
post         ClusterIP   10.104.186.193   <none>        5000/TCP         1m
ui           NodePort    10.106.85.96     <none>        9292:32299/TCP   1m

$ minikube ip
192.168.99.100

Browser: http://192.168.99.100:32299/

Clean:

helm del --purge post
helm del --purge ui
helm del --purge mongodb
```

### Deploy with Jenkins and Helm, Jenkins is running under k8s 

```
Setup jenkins https://github.com/adavarski/K8S-with-jenkins-and-helm

 $ cd ./jenkins    
 $ kubectl create -f jenkins-volume.yml 
 $ helm install --name jenkins --values values.yml stable/jenkins

Get admin user password:
printf $(kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo

 $ kubectl get svc|grep NodePort
jenkins         NodePort    10.100.88.246   <none>        8080:32123/TCP   45m

$ minikube ip
192.168.99.100

Login to jenkins http://192.168.99.100:32123 user admin with password from printf output

Setup k8s RBAC for jenkins:

$ kubectl create clusterrolebinding kube-system-default-admin --clusterrole cluster-admin --serviceaccount=kube-system:default
$ kubectl create clusterrolebinding default-sa-admin --user system:serviceaccount:default:default  --clusterrole cluster-admin

Install stable/mongodb helm chart (we will not build chart for mongodb)

$ helm  install --set usePassword=false --name mongodb  stable/mongodb

Deploy blueprint app

Create a multibranch pipline with a fork of this repository as the Git source for the pipeline using provided Jenkinsfile.

$ kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-9f8486cb5-kfb6s    1/1     Running   0          2h
mongodb-69fc4cddff-b2hqz   1/1     Running   1          59m
post-65c8d5c598-shhhm      1/1     Running   6          31m
ui-7d95c8ff49-4wwl2        1/1     Running   6          29m
ui-7d95c8ff49-fr74x        1/1     Running   4          29m

$ kubectl get svc|grep ui
ui              NodePort    10.103.25.143   <none>        9292:30124/TCP   9m

Browser: http://192.168.99.100:30124

Clean:

helm del --purge post
helm del --purge ui
helm del --purge mongodb


````
### Deploy with Jenkins outside k8s and Helm minikube 

```
