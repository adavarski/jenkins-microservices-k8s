# microservises app: bluepost (ui, post, mongodb)

```
Bluepost: Reddit-like application written in Ruby (Sinatra) created for learning purposes

The functionality is simple. The application allows you to share interesting links with the other, as well as vote and comment on them.

Application requirements:

Ruby v2.3
Bundler
MongoDB v3.2

```

We will build, test, release, deploy with GitLab CI/CD to k8s (minikube):

https://gitlab.com/bluepost
```
$ git clone https://github.com/adavarski/GitLab-microservices-k8s
$ cd GitLab-group-microservices-minikube-deploy
$ rm -rf .git
```

This is an example of a microservices application with **Gitlab CI** pipelenes (build, test, release, deploy)

In Gitlab UI, create a new group for bluepost application

Gitlab CI groups allow to group related projects. In our example, bluepost group will contain the microservices that the bluepost application consists of.

Create a new project in Gitlab CI web UI for each component of bluepost application. Each component of the bluepost application is contained in its own repository and has its own CI/CD pipeline defined in a .gitlab-ci-yml file stored in the root of each of the component’s directory.

Gitlab CI has its own container registry, but in this example we use Docker Hub,  you’ll need to make sure the CONTAINER_IMAGE variable is set according to your Docker Hub account name and repository for each image has been created. For each project define variables: Settings -> CI/CD. Define CI_REGISTRY_USER and CI_REGISTRY_PASSWORD variables to allow logging to Docker Hub. After define projects and setup CI/CD pipeline env variables execute:

Example: 
```
$ cd ui
$ git init
$ git add .
$ git commit -m "Init commit"
$ git remote add origin https://gitlab.com/bluepost/ui.git
$ git push origin master

$ cd ../post
$ git init
$ git add .
$ git commit -m "Init commit"
$ git remote add origin https://gitlab.com/bluepost/post.git
$ git push origin master

$ cd ../mongodb
$ git init
$ git add .
$ git commit -m "Init commit"
$ git remote add origin https://gitlab.com/bluepost/mongodb.git
$ git push origin master
```
if we dont use mongodb chart:

$ helm  install --set usePassword=false --name mongodb  stable/mongodb if we not use helm chart ... chart values -> usePassword=false

``` Setup ENV variables for pipeline: CI_REGISTRY_PASSWORD, CI_REGISTRY_USER ```
```
If we want only to build, test, deploy without minikube 

stages:
  - build
  - test
  - release

variables:

  CONTAINER_IMAGE: davarski/ui
  CONTAINER_IMAGE_BUILT: ${CONTAINER_IMAGE}:${CI_COMMIT_REF_SLUG}_${CI_COMMIT_SHA}
  CONTAINER_IMAGE_LATEST: ${CONTAINER_IMAGE}:latest
  CI_REGISTRY: index.docker.io  # container registry URL

# build container image
build:
  tags:
  - docker-minikube
  stage: build
  image: docker:latest
  services:
  - docker:dind
  script:
    - echo "Building Dockerfile-based application..."
    - docker build -t ${CONTAINER_IMAGE_BUILT} .
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD}
    - echo "Pushing to the Container Registry..."
    - docker push ${CONTAINER_IMAGE_BUILT}

# run tests against built image
test:
  stage: test
  script:
    - exit 0

# tag container image that passed the tests successfully
# and push it to the registry
release:
  tags:
  - docker-minikube
  stage: release
  image: docker:latest
  services:
  - docker:dind
  script:
    - echo "Pulling docker image from Container Registry"
    - docker pull ${CONTAINER_IMAGE_BUILT}
    - echo "Logging to Container Registry at ${CI_REGISTRY}"
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD}
    - echo "Pushing to Container Registry..."
    - docker tag ${CONTAINER_IMAGE_BUILT} ${CONTAINER_IMAGE}:$(cat VERSION)
    - docker push ${CONTAINER_IMAGE}:$(cat VERSION)
    - docker tag ${CONTAINER_IMAGE}:$(cat VERSION) ${CONTAINER_IMAGE_LATEST}
    - docker push ${CONTAINER_IMAGE_LATEST}
    - echo ""
  only:
    - master
  ```  

```
If we want to build, test, release and DEPLOY to minikube local k8s
For ui pipeline (the same for post, mongodb)

Go to https://gitlab.com/bluepost/ui/settings/ci_cd -> Runners (Expand)


Set up a specific Runner manually
Install GitLab Runner
Specify the following URL during the Runner setup: https://gitlab.com/ 
Use the following registration token during setup: yC87WF_USt9KZqF9c-Ps 
Reset runners registration token
Start the Runner!
```
```

minikube setup:

$ kubectl create -f gitlab-runner-deployment.yaml

$ kubectl get pod|grep runner|grep Running
gitlab-runner-5d49c87d4f-d6rwj                1/1     Running       0          2h

$kubectl exec -it gitlab-runner-5d49c87d4f-d6rwj /bin/bash

#gitlab-runner register --non-interactive --url "https://gitlab.com/" --registration-token "yC87WF_USt9KZqF9c-Ps" --executor "docker" --docker-image alpine:3 --description "docker-runner" --tag-list "docker-minikube" --run-untagged --locked="false"
Registering runner... succeeded                     runner=yC87WF_U
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```
```
https://gitlab.com/bluepost/ui/settings/ci_cd


Runners activated for this project
 757cded9 Pause Remove Runner
#562830
docker-runner

docker-minikube

Setup tag: docker-minikube for example for minikube cluster
```

```
Setup tags docker-minikube to use this runner with tag:minikube-runner


```

```Deploy

we can define KUBE_CONFIG variable for deploy pipelines this way for minikube:
minikube ssh ->  get /etc/kubernetes/admin.conf and change localhost to 192.168.99.100 (minikube ip) 
cat admin.conf|base64 > admin.conf.base64
 
Setup env variable: 

KUBE_CONFIG:  cat admin.conf.base64

stages:
  - deploy

variables:

  CONTAINER_IMAGE: davarski/ui
  CONTAINER_IMAGE_BUILT: ${CONTAINER_IMAGE}:${CI_COMMIT_REF_SLUG}_${CI_COMMIT_SHA}
  CONTAINER_IMAGE_LATEST: ${CONTAINER_IMAGE}:latest
  CI_REGISTRY: index.docker.io  # container registry URL
  CA: /etc/deploy/ca.crt

   
deploy_staging:
  tags:
  - docker-minikube 
  stage: deploy
  image: davarski/k8s-helm:latest
  before_script:
    - mkdir -p /etc/deploy
    - echo ${KUBE_CONFIG}|base64 -d > ${CA}
    - export KUBECONFIG=/etc/deploy/ca.crt
    - helm init --client-only

  script:
    - helm  upgrade ui ./charts/ui --install  --set image.tag=latest 
  environment:
    name: staging
  only:
  - master 

or we can use CA and token for our deployment user, we well use default user for simplicity and setup as cluster-admin, in production we have to have deployment user with right k8s RBAC settings. 

$ kubectl create clusterrolebinding default-sa-admin --user system:serviceaccount:default:default  --clusterrole cluster-admin

$ ./get-sa-token.sh --namespace default --account default

$ cat ca.crt | base64 > ca.crt.code


Gitlab: Setup CI/CD ui env variables:

CI_ENV_K8S_CA = cat ca.crt.code

CI_ENV_K8S_MASTER = https://192.168.99.100:8443

CI_ENV_K8S_SA_TOKEN = cat sa.token

New gitlab-ci.yml for deploy 

stages:
  - build
  - test
  - release
  - deploy

variables:

  CONTAINER_IMAGE: davarski/ui
  CONTAINER_IMAGE_BUILT: ${CONTAINER_IMAGE}:${CI_COMMIT_REF_SLUG}_${CI_COMMIT_SHA}
  CONTAINER_IMAGE_LATEST: ${CONTAINER_IMAGE}:latest
  CI_REGISTRY: index.docker.io  # container registry URL
  CA: /etc/deploy/ca.crt

# build container image
build:
  tags:
  - docker-minikube
  stage: build
  image: docker:latest
  services:
  - docker:dind
  script:
    - echo "Building Dockerfile-based application..."
    - docker build -t ${CONTAINER_IMAGE_BUILT} .
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD}
    - echo "Pushing to the Container Registry..."
    - docker push ${CONTAINER_IMAGE_BUILT}

# run tests against built image
test:
  stage: test
  script:
    - exit 0

# tag container image that passed the tests successfully
# and push it to the registry
release:
  tags:
  - docker-minikube
  stage: release
  image: docker:latest
  services:
  - docker:dind
  script:
    - echo "Pulling docker image from Container Registry"
    - docker pull ${CONTAINER_IMAGE_BUILT}
    - echo "Logging to Container Registry at ${CI_REGISTRY}"
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD}
    - echo "Pushing to Container Registry..."
    - docker tag ${CONTAINER_IMAGE_BUILT} ${CONTAINER_IMAGE}:$(cat VERSION)
    - docker push ${CONTAINER_IMAGE}:$(cat VERSION)
    - docker tag ${CONTAINER_IMAGE}:$(cat VERSION) ${CONTAINER_IMAGE_LATEST}
    - docker push ${CONTAINER_IMAGE_LATEST}
    - echo ""
  only:
    - master
    
deploy_staging:
  tags:
  - docker-minikube 
  stage: deploy
  image: davarski/k8s-helm:latest
  before_script:
    - mkdir -p /etc/deploy
    - echo ${CI_ENV_K8S_CA}|base64 -d > ${CA}
    - kubectl config set-cluster minikube --server=${CI_ENV_K8S_MASTER} --certificate-authority=/etc/deploy/ca.crt --embed-certs=true
    - kubectl config set-credentials default --token=${CI_ENV_K8S_SA_TOKEN}
    - kubectl config set-context myctxt --cluster=minikube --user=default
    - kubectl config use-context myctxt
    - helm init --client-only

  script:
    - helm  upgrade ui ./charts/ui --install  --set image.tag=latest 
  environment:
    name: staging
  only:
  - master 


```
```post pipeline

Set up a specific Runner manually
Install GitLab Runner
Specify the following URL during the Runner setup: https://gitlab.com/ 
Use the following registration token during setup: 7R8sUMcNtoku3eKVHQ-G 
Reset runners registration token
Start the Runner!

gitlab-runner register --non-interactive --url "https://gitlab.com/" --registration-token "7R8sUMcNtoku3eKVHQ-G" --executor "docker" --docker-image alpine:3 --description "docker-runner" --tag-list "docker-minikube2" --run-untagged --locked="false"

Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 


Runners activated for this project
 d6c05440 Pause Remove Runner
#563402
docker-runner

docker-minikube2

Setup tag: docker-minikube2

stages:
  - build
  - test
  - release
  - deploy
  
variables:

  CONTAINER_IMAGE: davarski/post
  CONTAINER_IMAGE_BUILT: ${CONTAINER_IMAGE}:${CI_COMMIT_REF_SLUG}_${CI_COMMIT_SHA}
  CONTAINER_IMAGE_LATEST: ${CONTAINER_IMAGE}:latest
  CI_REGISTRY: index.docker.io  # container registry URL
  CA: /etc/deploy/ca.crt
  

# build container image
build:
  tags:
  - docker-minikube2
  stage: build
  image: docker:latest
  services:
  - docker:dind
  script:
    - echo "Building Dockerfile-based application..."
    - docker build -t ${CONTAINER_IMAGE_BUILT} .
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD}
    - echo "Pushing to the Container Registry..."
    - docker push ${CONTAINER_IMAGE_BUILT}

# run tests against built image
test:
  stage: test
  script:
    - exit 0

# tag container image that passed the tests successfully
# and push it to the registry
release:
  tags:
  - docker-minikube2
  stage: release
  image: docker:latest
  services:
  - docker:dind
  script:
    - echo "Pulling docker image from Container Registry"
    - docker pull ${CONTAINER_IMAGE_BUILT}
    - echo "Logging to Container Registry at ${CI_REGISTRY}"
    - docker login -u ${CI_REGISTRY_USER} -p ${CI_REGISTRY_PASSWORD}
    - echo "Pushing to Container Registry..."
    - docker tag ${CONTAINER_IMAGE_BUILT} ${CONTAINER_IMAGE}:$(cat VERSION)
    - docker push ${CONTAINER_IMAGE}:$(cat VERSION)
    - docker tag ${CONTAINER_IMAGE}:$(cat VERSION) ${CONTAINER_IMAGE_LATEST}
    - docker push ${CONTAINER_IMAGE_LATEST}
    - echo ""
  only:
    - master
    
deploy_staging:
  tags:
  - docker-minikube2 
  stage: deploy
  image: davarski/k8s-helm:latest
  before_script:
    - mkdir -p /etc/deploy
    - echo ${CI_ENV_K8S_CA}|base64 -d > ${CA}
    - kubectl config set-cluster minikube --server=${CI_ENV_K8S_MASTER} --certificate-authority=/etc/deploy/ca.crt --embed-certs=true
    - kubectl config set-credentials default --token=${CI_ENV_K8S_SA_TOKEN}
    - kubectl config set-context myctxt --cluster=minikube --user=default
    - kubectl config use-context myctxt
    - helm init --client-only

  script:
    - helm  upgrade post ./charts/post --install  --set image.tag=latest 
  environment:
    name: staging
  only:
  - master 
  
 ``` 
 
 ```
 
Set up a specific Runner manually
Install GitLab Runner
Specify the following URL during the Runner setup: https://gitlab.com/ 
Use the following registration token during setup: uyC4Dx8R5D4qzyYP_9wM 
Reset runners registration token
Start the Runner!


davar@home ~/LABS/GitLab-group-microservices-minikube-deploy/ui $ kubectl exec -it gitlab-runner-5d49c87d4f-d6rwj /bin/bashroot@gitlab-runner-5d49c87d4f-d6rwj:/#gitlab-runner register --non-interactive --url "https://gitlab.com/" --registration-token "uyC4Dx8R5D4qzyYP_9wM" --executor "docker" --docker-image alpine:3 --description "docker-runner" --tag-list "docker-minikube3" --run-untagged --locked="false"


 b8d4c7ed Pause Remove Runner
#563710
docker-runner

docker-minikube3

change tag 

Runners activated for this project
 b8d4c7ed Pause Remove Runner
#563710
docker-runner

docker-minikube3 
 $ cat mongodb/.gitlab-ci.yml
stages:
  - deploy
  
variables:

  CA: /etc/deploy/ca.crt
  
deploy_staging:
  tags:
  - docker-minikube3
  stage: deploy
  image: davarski/k8s-helm:latest
  before_script:
    - mkdir -p /etc/deploy
    - echo ${CI_ENV_K8S_CA}|base64 -d > ${CA}
    - kubectl config set-cluster minikube --server=${CI_ENV_K8S_MASTER} --certificate-authority=/etc/deploy/ca.crt --embed-certs=true
    - kubectl config set-credentials default --token=${CI_ENV_K8S_SA_TOKEN}
    - kubectl config set-context myctxt --cluster=minikube --user=default
    - kubectl config use-context myctxt
    - helm init --client-only

  script:
    - helm  upgrade mongodb ./charts/mongodb --install  --set image.tag=latest 
  environment:
    name: staging
  only:
  - master   

 
 ```
 ```
davar@home ~/LABS/GitLab-group-microservices-minikube-deploy/post $ helm list
NAME   	REVISION	UPDATED                 	STATUS  	CHART         	APP VERSION	NAMESPACE
jenkins	1       	Fri Nov 23 11:50:09 2018	DEPLOYED	jenkins-0.20.1	2.121.3    	default  
post   	1       	Mon Nov 26 21:34:39 2018	DEPLOYED	post-0.1.0    	           	default  
ui     	1       	Mon Nov 26 20:54:12 2018	DEPLOYED	ui-0.1.0      	           	default
mongodb	1       	Mon Nov 26 23:34:19 2018	DEPLOYED	mongodb-4.6.2 	4.0.3      	default  

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

### Deploy with Jenkins and Helm

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
### Deploy with GitLab and Helm minikube : TODO

```
Install GitLab or use gitlab chart to install 

Install the GitLab runner chart:

$ helm install \
    --namespace gitlab \
    --name gitlab-runner \
    -f gitlab-runner.yml \
    gitlab/gitlab-runner
Register the Docker runner:

$ gitlab-runner register -n \
    --url http://gitlab.dq.b.com/ \
    --registration-token ${RUNNER_TOKEN} \
    --executor docker \
    --description "Docker Runner" \
    --docker-image "docker:latest" \
    --docker-volumes /var/run/docker.sock:/var/run/docker.sock

$ helm init # initialize Helm
$ helm repo add gitlab https://charts.gitlab.io
$ helm install --name gitlab \
--namespace default \
--set baseIP=192.168.100.99 \
--set baseDomain=ci.devops.fun \
gitlab/gitlab-omnibus

Import project from github:

Setup deploy pipeline: add .gitlab-ci.yml to root of repo:

stages:
  - deploy

# deploy to staging environment (minikube)
deploy_staging:
  stage: deploy
  image: ibmcom/k8s-helm:v2.6.0
  before_script:
    - export KUBECONFIG=/etc/kubernetes/admin.conf 
    - helm init --client-only
  script:
    - helm upgrade  upgrade post ./post/charts/post --install --set image.tag=latest
    - helm upgrade  upgrade ui ./ui/charts/ui --install  --set image.tag=latest

 environment:
    name: staging
  only:
  - master
```


 ###  test with with gitlab-runner on host
 
 ```
 
 sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
 sudo chmod +x /usr/local/bin/gitlab-runner


$ cat .gitlab-ci.yml
stages:
  - deploy

# deploy to staging environment
deploy_staging:
  stage: deploy
  image: davarski/k8s-helm:latest
  script:
    - export KUBECONFIG=/home/davar/.kube/config
    - helm init --client-only
    - helm  upgrade ui /ui/charts/ui --install  --set image.tag=latest 
  environment:
    name: staging
  only:
  - master

davar@home ~/LABS/GitLab-group-microservices-minikube-deploy/ui $ gitlab-runner exec docker deploy_staging --docker-volumes /home/davar/.kube:/home/davar/.kube --docker-volumes /home/davar/.minikube:/home/davar/.minikube --docker-volumes ${PWD}:/ui
Runtime platform                                    arch=amd64 os=linux pid=13116 revision=3afdaba6 version=11.5.0
.....

Job succeeded
  
```  
  
