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
```
kubectl --kubeconfig=./admin.conf get po
```
Jenkinsfile provided to build and deploy app

Create pipilene in jenkis with this Jenkinsfile

```
Started by user Anastas Dimov
Obtained Jenkinsfile from git https://github.com/adavarski/jenkins-microservices-k8s
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/jenkins-microservices-k8s
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Clone repository)
[Pipeline] checkout
 > git rev-parse --is-inside-work-tree # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/adavarski/jenkins-microservices-k8s # timeout=10
Fetching upstream changes from https://github.com/adavarski/jenkins-microservices-k8s
 > git --version # timeout=10
 > git fetch --tags --progress https://github.com/adavarski/jenkins-microservices-k8s +refs/heads/*:refs/remotes/origin/*
 > git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > git rev-parse refs/remotes/origin/origin/master^{commit} # timeout=10
Checking out Revision 76358599af88094f79b66999b6dfc028bfbe9fe9 (refs/remotes/origin/master)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 76358599af88094f79b66999b6dfc028bfbe9fe9
Commit message: "Merge branch 'master' of https://github.com/adavarski/jenkins-microservices-k8s"
 > git rev-list --no-walk 76358599af88094f79b66999b6dfc028bfbe9fe9 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build image post)
[Pipeline] sh
+ docker build -t davarski/post ./post
Sending build context to Docker daemon   29.7kB

Step 1/7 : FROM python:2.7
 ---> bfa54426aeda
Step 2/7 : WORKDIR /app
 ---> Using cache
 ---> d2bb5ff53816
Step 3/7 : ADD requirements.txt /app
 ---> Using cache
 ---> 789136d9e6e7
Step 4/7 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> 854148ddcdd8
Step 5/7 : ADD . /app
 ---> Using cache
 ---> 2e60a3cfec53
Step 6/7 : EXPOSE  5000
 ---> Using cache
 ---> 2aba203a9b22
Step 7/7 : ENTRYPOINT ["python", "post_app.py"]
 ---> Using cache
 ---> 5f92f3e7abdb
Successfully built 5f92f3e7abdb
Successfully tagged davarski/post:latest
[Pipeline] dockerFingerprintFrom
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Push image post)
[Pipeline] withEnv
[Pipeline] {
[Pipeline] withDockerRegistry
$ docker login -u davarski -p ******** https://index.docker.io/v1/
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /var/lib/jenkins/workspace/jenkins-microservices-k8s@tmp/dac80283-ff46-4042-a3e7-1e8ca348f4b0/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[Pipeline] {
[Pipeline] sh
+ docker tag davarski/post index.docker.io/davarski/post:latest
[Pipeline] sh
+ docker push index.docker.io/davarski/post:latest
The push refers to repository [docker.io/davarski/post]
1cd70ea7f6a3: Preparing
ecd48d39d7aa: Preparing
5cd51c3f902b: Preparing
d522192957b9: Preparing
836bc5486c98: Preparing
cf8fd1b5fc6d: Preparing
dc17e5c1609d: Preparing
1610bcd0f525: Preparing
1a36262221c3: Preparing
d2217ead3a1c: Preparing
b53b57a50746: Preparing
d2518892581f: Preparing
c581f4ede92d: Preparing
dc17e5c1609d: Waiting
1610bcd0f525: Waiting
1a36262221c3: Waiting
d2217ead3a1c: Waiting
b53b57a50746: Waiting
d2518892581f: Waiting
c581f4ede92d: Waiting
cf8fd1b5fc6d: Waiting
1cd70ea7f6a3: Layer already exists
d522192957b9: Layer already exists
836bc5486c98: Layer already exists
ecd48d39d7aa: Layer already exists
5cd51c3f902b: Layer already exists
cf8fd1b5fc6d: Layer already exists
d2217ead3a1c: Layer already exists
1610bcd0f525: Layer already exists
1a36262221c3: Layer already exists
dc17e5c1609d: Layer already exists
d2518892581f: Layer already exists
b53b57a50746: Layer already exists
c581f4ede92d: Layer already exists
latest: digest: sha256:e683a926a744ac8f1d2fefe3fe840a242c31740bb1c1ccfd9e7bdb6c183c6c2b size: 3055
[Pipeline] }
[Pipeline] // withDockerRegistry
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build image ui)
[Pipeline] sh
+ docker build -t davarski/ui ./ui
Sending build context to Docker daemon  45.06kB

Step 1/9 : FROM ruby:2.3.3
 ---> 0e1db669d557
Step 2/9 : RUN apt-get update && apt-get install -y build-essential
 ---> Using cache
 ---> 4c75ea74a82e
Step 3/9 : ENV APP_HOME /app
 ---> Using cache
 ---> 6d6d6633ef2f
Step 4/9 : RUN mkdir $APP_HOME
 ---> Using cache
 ---> f3e4ed3252fb
Step 5/9 : WORKDIR $APP_HOME
 ---> Using cache
 ---> dc2eb5408dd2
Step 6/9 : ADD Gemfile* $APP_HOME/
 ---> Using cache
 ---> 40bd95193873
Step 7/9 : RUN bundle install
 ---> Using cache
 ---> 6fa889b99e13
Step 8/9 : ADD . $APP_HOME
 ---> Using cache
 ---> 2d5c78db0404
Step 9/9 : CMD ["puma"]
 ---> Using cache
 ---> 962db5346bcc
Successfully built 962db5346bcc
Successfully tagged davarski/ui:latest
[Pipeline] dockerFingerprintFrom
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Push image ui)
[Pipeline] withEnv
[Pipeline] {
[Pipeline] withDockerRegistry
$ docker login -u davarski -p ******** https://index.docker.io/v1/
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /var/lib/jenkins/workspace/jenkins-microservices-k8s@tmp/55c8263f-6bb5-4e8b-b203-4be6f15e8285/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[Pipeline] {
[Pipeline] sh
+ docker tag davarski/ui index.docker.io/davarski/ui:latest
[Pipeline] sh
+ docker push index.docker.io/davarski/ui:latest
The push refers to repository [docker.io/davarski/ui]
31e474dd48c5: Preparing
30d2c203a6d7: Preparing
8af639d233b1: Preparing
562a8ca24f35: Preparing
a692f70c5b0f: Preparing
57da7b5b7d37: Preparing
e433c8cc41f9: Preparing
807c0e129d43: Preparing
ac45884e47e6: Preparing
f078de0e3e2b: Preparing
e6562eb04a92: Preparing
596280599f68: Preparing
5d6cbe0dbcf9: Preparing
57da7b5b7d37: Waiting
e433c8cc41f9: Waiting
807c0e129d43: Waiting
e6562eb04a92: Waiting
ac45884e47e6: Waiting
f078de0e3e2b: Waiting
596280599f68: Waiting
5d6cbe0dbcf9: Waiting
a692f70c5b0f: Layer already exists
30d2c203a6d7: Layer already exists
562a8ca24f35: Layer already exists
8af639d233b1: Layer already exists
31e474dd48c5: Layer already exists
e433c8cc41f9: Layer already exists
f078de0e3e2b: Layer already exists
57da7b5b7d37: Layer already exists
ac45884e47e6: Layer already exists
807c0e129d43: Layer already exists
e6562eb04a92: Layer already exists
596280599f68: Layer already exists
5d6cbe0dbcf9: Layer already exists
latest: digest: sha256:400994f88625f1ac018a788ea92a4657572d8782252559adf487e7b0c8f607a8 size: 3048
[Pipeline] }
[Pipeline] // withDockerRegistry
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy)
[Pipeline] sh
+ helm --kubeconfig=./admin.conf init --client-only --skip-refresh
$HELM_HOME has been configured at /var/lib/jenkins/.helm.
Not installing Tiller due to 'client-only' flag having been set
Happy Helming!
[Pipeline] sh
+ helm --kubeconfig=./admin.conf upgrade mongodb mongodb/charts/mongodb --install
Release "mongodb" has been upgraded. Happy Helming!
LAST DEPLOYED: Wed Jan 23 13:25:15 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/PersistentVolumeClaim
NAME     STATUS  VOLUME                                    CAPACITY  ACCESS MODES  STORAGECLASS  AGE
mongodb  Bound   pvc-932a0c7c-1efe-11e9-89f2-0800272e7054  8Gi       RWO           standard      21m

==> v1/Service
NAME     TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)    AGE
mongodb  ClusterIP  10.99.52.190  <none>       27017/TCP  21m

==> v1beta1/Deployment
NAME     DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
mongodb  1        1        1           1          21m

==> v1/Pod(related)
NAME                      READY  STATUS   RESTARTS  AGE
mongodb-69fc4cddff-n247p  1/1    Running  0         21m


NOTES:


** Please be patient while the chart is being deployed **

MongoDB can be accessed via port 27017 on the following DNS name from within your cluster:

    mongodb.default.svc.cluster.local



To connect to your database run the following command:

    kubectl run --namespace default mongodb-client --rm --tty -i --image bitnami/mongodb --command -- mongo admin --host mongodb

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/mongodb 27017:27017 &
    mongo --host 127.0.0.1

[Pipeline] sh
+ helm --kubeconfig=./admin.conf upgrade post post/charts/post --install --set image.tag=latest
Release "post" has been upgraded. Happy Helming!
LAST DEPLOYED: Wed Jan 23 13:25:16 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME  TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)   AGE
post  ClusterIP  10.96.188.84  <none>       5000/TCP  21m

==> v1beta1/Deployment
NAME  DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
post  1        1        1           1          21m

==> v1/Pod(related)
NAME                   READY  STATUS   RESTARTS  AGE
post-65c8d5c598-2vqnq  1/1    Running  0         21m


NOTES:

  export POD_NAME=$(kubectl get pods --namespace default -l "app=post,release=post" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:5000

[Pipeline] sh
+ helm --kubeconfig=./admin.conf upgrade ui ui/charts/ui --install --set image.tag=latest
Release "ui" has been upgraded. Happy Helming!
LAST DEPLOYED: Wed Jan 23 13:25:16 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                 READY  STATUS   RESTARTS  AGE
ui-7d95c8ff49-9gztx  1/1    Running  0         21m
ui-7d95c8ff49-zwrgg  1/1    Running  0         21m

==> v1/Service
NAME  TYPE      CLUSTER-IP     EXTERNAL-IP  PORT(S)         AGE
ui    NodePort  10.102.81.249  <none>       9292:31334/TCP  21m

==> v1beta1/Deployment
NAME  DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
ui    2        2        2           2          21m


NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services ui)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
  export SERVICE_IP=$(kubectl get svc --namespace default ui -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
  echo http://$SERVICE_IP:9292
  export POD_NAME=$(kubectl get pods --namespace default -l "app=ui,release=ui" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:9292

[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS

```



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

