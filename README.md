First Stage.

1. login to your github account

2. create a pvt repository

3. click on top right of profile, go to settings and , go to developer settings,
click on Peasonal access(PAT), select tokens(classic), click on genarate new token , select generate new classic,
keep note , select expiry based on your choice and give repo access only and scroll down and click on generate token.

4. copy it some where
* Not:- Never ever share your credentais or tokens with anyone.


5. go to your pvt repo copy the git url of repo.


6. open your vs or your local terminal and clone the repo, use token to clone the repo

```bash
aravi@Aravind MINGW64 ~/11_04
$ git clone https://github.com/xaravind/pvt-repo.git
Cloning into 'pvt-repo'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 0), reused 3 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (6/6), done.
```

7. copy the code to your local folder to git pvt repo folder.

8. Write a Dockerfile in your code, save and push it.

9. check git hub in your browser, the files should be updated like in below image.

As of now we created  a one github repo, created PAT key , cloned the repo in your local system , copied the code, create a Dockerfile and pushed files to github. 

Second Stage.

1. Login into your Aws account.

2. Go to EC2 , click on instances and click on launch instance.

3. give name , and create a key pair. remaining all other settings keep it as default. 
we will this VM to build images , create Eks cluster, and to communicate with cluster. 

4. we have taken Amazon linux AMi which is a RHEl based . click on launch instance.

5. copy the public ip and go to your putty , mobaxterm any one based on your choice and connect to the server.

6. swith to root and update the server.

```bash
sudo su -
yum update -y
```

here in this stage we created a aws ec2-instance, created a pem key, remaing setting we kept it default , launch instance and connected to it and updated the servver.

Third stage.

1. install git, docker package and start docker

```bash
yum install -y git docker
systemctl start docker
```

2. clone the pvt-repo, give your username and password as PAT key. 


3. switch to directory, built the docker image.

```bash
[root@ip-172-31-86-127 ~]# cd pvt-repo/
[root@ip-172-31-86-127 pvt-repo]# ls
README.md  js_code
[root@ip-172-31-86-127 pvt-repo]# cd js_code/
[root@ip-172-31-86-127 js_code]# ls
Dockerfile  dist  package-lock.json  package.json  readme.md  src  tsconfig.json  webpack.config.js
[root@ip-172-31-86-127 js_code]# docker build -t jscode:1.0 .
```

4. check the image and run the container
our docker image is just 56.2MB as we used alpine image shortest image as possible.

```bash
docker ps
docker run -dt --name jscode -p 80:80 jscode:1.0
```

5. go into container and twerk the app, i have update index.html code manually as it create a some difference in v1 to v2. we can debug the containers and can do modifictaions and create a new image of it for emergency situations.

```bash
[root@ip-172-31-86-127 js_code]# docker exec -it 224e0843b7 sh
/ # cd /usr/share/nginx/html
/usr/share/nginx/html # ls
50x.html    bundle.js   images      index.html
/usr/share/nginx/html # >index.html
/usr/share/nginx/html # vi index.html
```

6. if you want , you can access the application with <public-ip:80>, but you have to modify the securtiy groups and add inbound rules 80. we'll see it how it done next stages.

7. exit the container and create a new image out of running container.

```bash
[root@ip-172-31-86-127 js_code]# docker commit 224e0843b751 jscode:2.0
sha256:afa8f59407427d1339a6140c73cdcdb4ca1153437ab007eb21b4e02980740cb5
[root@ip-172-31-86-127 js_code]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
jscode       2.0       afa8f5940742   5 seconds ago    56.2MB
jscode       1.0       b2d1f7cddd1e   13 minutes ago   56.2MB
[root@ip-172-31-86-127 js_code]# docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS          PORTS                               NAMES
224e0843b751   jscode:1.0   "/docker-entrypoint.…"   11 minutes ago   Up 11 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   jscode
```
8. Login docker hub , create a pvt repo to store our docker images.

9. tag images  to push pvt repo

```bash
docker tag jscode:1.0 xaravind/jscode:v1
docker tag jscode:2.0 xaravind/jscode:v2
```

10. login docker with your  credentials

```bash
[root@ip-172-31-86-127 js_code]# docker login
Log in with your Docker ID or email address to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com/ to create one.
You can log in with your password or a Personal Access Token (PAT). Using a limited-scope PAT grants better security and is required for organizations using SSO. Learn more at https://docs.docker.com/go/access-tokens/

Username: xaravind
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

```
it will create /root/.docker/config.json.  and it will have our credtentails , we can use this file to pull this image in k8s.

11. now push the images

```bash
[root@ip-172-31-86-127 js_code]# docker push xaravind/jscode:v1
The push refers to repository [docker.io/xaravind/jscode]
076da8722e67: Pushed
08000c18d16d: Mounted from library/nginx
v1: digest: sha256:64ad49cab2398a14cd27301aaaeddba5774147ca9cba10185c2bcc942bc6ef69 size: 2200
[root@ip-172-31-86-127 js_code]# docker push xaravind/jscode:v2
The push refers to repository [docker.io/xaravind/jscode]
ae431e792082: Pushed
08000c18d16d: Layer already exists
v2: digest: sha256:a8da07d6fbdce74c9e679e512d258a1d23b9ffd04931ba5f6ceb218417d4826e size: 2408
```

12. check in docker hub in the browser, you will see your images.

In this stage we installed required packages, clone the code, created a docker image from docker file and run the container and connected to container and did some modificataions and created a image out of it. and we pushed images to docker hub private repo.

Fourth stage.

1. Go to you Aws console, search for IAM

2. click on role, create role

3. select trusted entity type Aws Service and use case ec2, scroll down and click on next.

4. select administator access as it will have access to all ec2-services that required to create a EKS cluster. scroll down and click on next.

5. give role name and scroll down and click on create role.

6. go to ec2 - instances and select our instance , click on actions , go to securiy, modify IAM role. 

7. select newly created IAM role and click on update role.
Now with our EC2-Instance we can control all aws service with aws cli.

8. Aws cli is already as  we selected aws linux, in other os you have to install aws cli.

9. check aws access.

```bash
[ec2-user@ip-172-31-86-127 ~]$ aws s3 mb s3://test-k8s.js
make_bucket: test-k8s.js
[ec2-user@ip-172-31-86-127 ~]$ aws s3 ls
2025-06-15 06:52:01 test-k8s.js
```

10. create a eks.yml in local and push to github.

```bash
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: project
  region: us-east-1

managedNodeGroups:
- name: project
  instanceType: m5.large
  desiredCapacity: 3
  spot: true
```
we have taken spot instances as we'll have get 90 % discount. but it not recommended as Aws will remove the ec2 instances with very short notice. but for our test purpose it is fine.

11. pull the changes in ec2-instance

```bash
[root@ip-172-31-86-127 ~]# cd pvt-repo/
[root@ip-172-31-86-127 pvt-repo]# ls
README.md  js_code
[root@ip-172-31-86-127 pvt-repo]# git pull -a
[root@ip-172-31-86-127 pvt-repo]# ls
README.md  eks.yml  js_code
```

12. eks installation.

go to https://eksctl.io/installation/ and follow steps the steps.

```bash
[root@ip-172-31-86-127 pvt-repo]# ls
README.md  eks.yml  js_code
[root@ip-172-31-86-127 pvt-repo]# # for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin
eksctl_Linux_amd64.tar.gz: OK
rm: remove regular file 'eksctl_Linux_amd64.tar.gz'? y

[root@ip-172-31-86-127 pvt-repo]# eksctl version
0.210.0

```

13. create a eks cluster with config file, it will take time create all required aws-service to up the cluster,like 
nodes, vpcs, sg,asg..et.., 

```bash
[root@ip-172-31-86-127 pvt-repo]# eksctl create cluster --config-file=eks.yml
2025-06-15 07:05:09 [ℹ]  eksctl version 0.210.0
2025-06-15 07:05:09 [ℹ]  using region us-east-1
2025-06-15 07:05:09 [ℹ]  setting availability zones to [us-east-1b us-east-1a]
2025-06-15 07:05:09 [ℹ]  subnets for us-east-1b - public:192.168.0.0/19 private:192.168.64.0/19
2025-06-15 07:05:09 [ℹ]  subnets for us-east-1a - public:192.168.32.0/19 private:192.168.96.0/19
2025-06-15 07:05:09 [ℹ]  nodegroup "project" will use "" [AmazonLinux2023/1.32]
2025-06-15 07:05:09 [ℹ]  using Kubernetes version 1.32
2025-06-15 07:18:00 [✔]  EKS cluster "project" in "us-east-1" region is ready

```


14. install kubectl to interact  with cluster.

```bash
[root@ip-172-31-86-127 pvt-repo]# curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.0/2025-05-01/bin/linux/amd64/kubectl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 57.3M  100 57.3M    0     0  21.2M      0  0:00:02  0:00:02 --:--:-- 21.2M
[root@ip-172-31-86-127 pvt-repo]# chmod +x ./kubectl
[root@ip-172-31-86-127 pvt-repo]# mv ./kubectl /usr/local/bin
[root@ip-172-31-86-127 pvt-repo]# kubectl version --client
Client Version: v1.33.0-eks-802817d
Kustomize Version: v5.6.0
```

15. check the kubectl commands

```bash
[root@ip-172-31-86-127 pvt-repo]#  kubectl api-resources | wc -l
66
[root@ip-172-31-86-127 pvt-repo]# kubectl get nodes
NAME                             STATUS   ROLES    AGE    VERSION
ip-192-168-27-169.ec2.internal   Ready    <none>   9m7s   v1.32.3-eks-473151a
ip-192-168-55-115.ec2.internal   Ready    <none>   9m8s   v1.32.3-eks-473151a
ip-192-168-63-135.ec2.internal   Ready    <none>   9m7s   v1.32.3-eks-473151a
```

16. encrypt the file below command to use it in k8s-secret and to pull the pvt images in k8s.

```bash
[root@ip-172-31-86-127 pvt-repo]# cat ~/.docker/config.json | base64 -w 0
ewoJImF1dGhzIjxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxCn0=
```

17. create a mainfest file in local push the code and pull the code in ec2-intance

```bash
apiVersion: v1
kind: Secret
metadata:
  name: regcred
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOiB7CgkJCSJhdXRoIjogImVHRnlZWFpwYm1RNk1UazVOVjlLZFd4NSIKCQl9Cgl9Cn0=
---
[root@ip-172-31-86-127 pvt-repo]# cat manifest.yml
apiVersion: v1
kind: Secret
metadata:
  name: regcred
  namespace: project
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOiB7CgkJCSJhdXRoIjogImVHRnlZWFpwYm1RNk1UazVOVjlLZFd4NSIKCQl9Cgl9Cn0=
---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: project
  labels:
    app: frontend
    environment: project
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: web
  template:
    metadata:
      labels:
        app: frontend
        tier: web
    spec:
      imagePullSecrets:
        - name: regcred
      containers:
        - name: frontend
          image: xaravind/jscode:v2
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 20
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: project
spec:
  type: LoadBalancer
  selector:
    app: frontend
    tier: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```



18. create namespace and apply the manifest file.

```bash
[root@ip-172-31-86-127 pvt-repo]# kubectl create namespace project
namespace/project created
[root@ip-172-31-86-127 pvt-repo]# kubectl apply -f manifest.yml
secret/regcred unchanged
deployment.apps/frontend created
service/frontend created
```

19. check the pods and service, it will create a load balancer service and open a random nodeport 30000–32767

```bash

[root@ip-172-31-86-127 pvt-repo]# kubectl get pods -n project
NAME                        READY   STATUS             RESTARTS      AGE
frontend-644df5c8b4-97gbj   0/1     Running            1 (32s ago)   92s
frontend-6467b94fb8-hghzx   0/1     Running            0             11s
frontend-6467b94fb8-t95fs   1/1     Running            0             25s
frontend-6d78b8b5b6-xw8hl   0/1     CrashLoopBackOff   5 (21s ago)   6m22s
[root@ip-172-31-86-127 pvt-repo]# kubectl describe pod frontend-6467b94fb8-hghzx -n project
Name:             frontend-6467b94fb8-hghzx

Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  28s   default-scheduler  Successfully assigned project/frontend-6467b94fb8-hghzx to ip-192-168-63-135.ec2.internal
  Normal  Pulled     27s   kubelet            Container image "xaravind/jscode:v1" already present on machine
  Normal  Created    27s   kubelet            Created container: frontend
  Normal  Started    27s   kubelet            Started container frontend
[root@ip-172-31-86-127 pvt-repo]# kubectl get svc frontend -n project
NAME       TYPE           CLUSTER-IP     EXTERNAL-IP                                                              PORT(S)        AGE
frontend   LoadBalancer   10.100.15.41   ab39bfabc67f9405f8e6c5b308273a51-706994256.us-east-1.elb.amazonaws.com   80:31689/TCP   7m3s
```

20. we need to add security group  30000–32767 to the node to access the application, other wise the target groups not become healthy.

21. select any node intance , go to security, click on sg and edit inbound rules add port.

22. access the application

```bash
curl http://<EXTERNAL-IP>/
curl http://ab39bfabc67f9405f8e6c5b308273a51-706994256.us-east-1.elb.amazonaws.com/
```

23. update mainfest file with v2 image - xaravind/jscode:v2


24. repply the changes

```bash
[root@ip-172-31-86-127 pvt-repo]# vi manifest.yml
[root@ip-172-31-86-127 pvt-repo]# kubectl get pods -o wide -n project
NAME                        READY   STATUS    RESTARTS   AGE     IP               NODE                             NOMINATED NODE   READINESS GATES
frontend-6467b94fb8-hghzx   1/1     Running   0          7m12s   192.168.62.210   ip-192-168-63-135.ec2.internal   <none>           <none>
frontend-6467b94fb8-hs94j   1/1     Running   0          6m59s   192.168.43.93    ip-192-168-55-115.ec2.internal   <none>           <none>
frontend-6467b94fb8-t95fs   1/1     Running   0          7m26s   192.168.29.173   ip-192-168-27-169.ec2.internal   <none>           <none>
[root@ip-172-31-86-127 pvt-repo]# kubectl apply -f manifest.yml
secret/regcred unchanged
deployment.apps/frontend configured
service/frontend unchanged
[root@ip-172-31-86-127 pvt-repo]# kubectl get pods -o wide -n project
NAME                        READY   STATUS    RESTARTS   AGE     IP               NODE                             NOMINATED NODE   READINESS GATES
frontend-6467b94fb8-hghzx   1/1     Running   0          7m55s   192.168.62.210   ip-192-168-63-135.ec2.internal   <none>           <none>
frontend-6467b94fb8-hs94j   1/1     Running   0          7m42s   192.168.43.93    ip-192-168-55-115.ec2.internal   <none>           <none>
frontend-6467b94fb8-t95fs   1/1     Running   0          8m9s    192.168.29.173   ip-192-168-27-169.ec2.internal   <none>           <none>
frontend-f8f8b89df-zn2gn    0/1     Running   0          4s      192.168.2.38     ip-192-168-27-169.ec2.internal   <none>           <none>
[root@ip-172-31-86-127 pvt-repo]# kubectl get pods -o wide -n project
NAME                        READY   STATUS    RESTARTS   AGE     IP               NODE                             NOMINATED NODE   READINESS GATES
frontend-6467b94fb8-hghzx   1/1     Running   0          8m3s    192.168.62.210   ip-192-168-63-135.ec2.internal   <none>           <none>
frontend-6467b94fb8-hs94j   1/1     Running   0          7m50s   192.168.43.93    ip-192-168-55-115.ec2.internal   <none>           <none>
frontend-6467b94fb8-t95fs   1/1     Running   0          8m17s   192.168.29.173   ip-192-168-27-169.ec2.internal   <none>           <none>
frontend-f8f8b89df-zn2gn    0/1     Running   0          12s     192.168.2.38     ip-192-168-27-169.ec2.internal   <none>           <none>
[root@ip-172-31-86-127 pvt-repo]# kubectl get pods -o wide -n project
NAME                        READY   STATUS    RESTARTS   AGE     IP               NODE                             NOMINATED NODE   READINESS GATES
frontend-6467b94fb8-hghzx   1/1     Running   0          8m10s   192.168.62.210   ip-192-168-63-135.ec2.internal   <none>           <none>
frontend-6467b94fb8-hs94j   1/1     Running   0          7m57s   192.168.43.93    ip-192-168-55-115.ec2.internal   <none>           <none>
frontend-f8f8b89df-ggncq    0/1     Running   0          5s      192.168.47.194   ip-192-168-63-135.ec2.internal   <none>           <none>
frontend-f8f8b89df-zn2gn    1/1     Running   0          19s     192.168.2.38     ip-192-168-27-169.ec2.internal   <none>           <none>
[root@ip-172-31-86-127 pvt-repo]# kubectl get pods -o wide -n project
NAME                       READY   STATUS    RESTARTS   AGE   IP               NODE                             NOMINATED NODE   READINESS GATES
frontend-f8f8b89df-ggncq   1/1     Running   0          61s   192.168.47.194   ip-192-168-63-135.ec2.internal   <none>           <none>
frontend-f8f8b89df-xmjqn   1/1     Running   0          49s   192.168.39.242   ip-192-168-55-115.ec2.internal   <none>           <none>
frontend-f8f8b89df-zn2gn   1/1     Running   0          75s   192.168.2.38     ip-192-168-27-169.ec2.internal   <none>           <none>

```

you can observe it clearly it will create a new pod with updated version and removing older pods one by one woth out having any downtime , it is called rolling update.


```bash
[root@ip-172-31-86-127 pvt-repo]# kubectl describe svc frontend -n project
Name:                     frontend
Namespace:                project
Labels:                   <none>
Annotations:              <none>
Selector:                 app=frontend,tier=web
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.100.15.41
IPs:                      10.100.15.41
LoadBalancer Ingress:     ab39bfabc67f9405f8e6c5b308273a51-706994256.us-east-1.elb.amazonaws.com
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31689/TCP
Endpoints:                192.168.2.38:80,192.168.47.194:80,192.168.39.242:80
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:
  Type    Reason                Age                From                Message
  ----    ------                ----               ----                -------
  Normal  EnsuringLoadBalancer  12m (x2 over 17m)  service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer   12m (x2 over 17m)  service-controller  Ensured load balancer
```

25 . once your done, clean off the project

```bash
 eksctl delete cluster --config-file=eks.yml
 [root@ip-172-31-86-127 pvt-repo]# eksctl delete cluster --config-file=eks.yml
2025-06-15 08:25:07 [ℹ]  deleting EKS cluster "project"
2025-06-15 08:25:07 [ℹ]  will drain 0 unmanaged nodegroup(s) in cluster "project"
2025-06-15 08:25:07 [ℹ]  starting parallel draining, max in-flight of 1
2025-06-15 08:25:07 [ℹ]  deleted 0 Fargate profile(s)
2025-06-15 08:25:08 [✔]  kubeconfig has been updated
2025-06-15 08:25:08 [ℹ]  cleaning up AWS load balancers created by Kubernetes objects of Kind Service or Ingress
2025-06-15 08:25:33 [ℹ]
2 sequential tasks: { delete nodegroup "project", delete cluster control plane "project" [async]
}
2025-06-15 08:25:33 [ℹ]  will delete stack "eksctl-project-nodegroup-project"
2025-06-15 08:25:33 [ℹ]  waiting for stack "eksctl-project-nodegroup-project" to get deleted
2025-06-15 08:25:33 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:26:03 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:26:48 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:27:40 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:28:30 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:29:31 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:30:55 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:32:25 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:34:07 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:35:46 [ℹ]  waiting for CloudFormation stack "eksctl-project-nodegroup-project"
2025-06-15 08:35:46 [ℹ]  will delete stack "eksctl-project-cluster"
2025-06-15 08:35:46 [✔]  all cluster resources were deleted

```

