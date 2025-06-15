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

1. Go to you Aws console, create a IAM role with adminstartion access and attach the role to ec2-instance

