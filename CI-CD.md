**Create an end to end CI/CD pipeline in AWS platform using Jenkins as the orchestration tool, Github as the SCM, Maven as the Build tool, Deploy in a docker instance and 
create a Docker image, Store the docker image in ECR, Achieve Kubernetes deployment using the ECR image. Build a sample java web app using maven.**

Create an ec2 instance ,t2.medium ( 2 core medium is used because jenkins is an orchestration tool ), created keypair , subnet-1a , security group - 8080 custom tcp  
Instance created.

We need to create 3 instances ( developer server , jenkins server , tomcat server ) # developer and tomcat server can have t2.micro 

**Open developer server - connect the instance on terminal**
1. sudo su -
2. hostnamectl set-hostname developer  
3. bash 
4. ssh-keygen (to connect github with dev server ) 
5. yum install git -y (to install git on the server )
6. mkdir /data ( create a directory named data to store the git repo )
7. cd /data
8. git init (initialize git )
9. cd
10. cd .ssh/
11. cat id_rsa.pub (to check the key id )
12. # go to github account add ssh-keygen in github
13.come back to terminal 
14. cd /data 
15.git pull the ssh of the project that you have in your repo 
16. git branch 
17. ls
18. git branch -M main
19. git branch 
20. ### Done witht the developer server 


##########################################################################################################################################################

1.now open the jenkins server , connect the ec2 instance 
2.sudo su -
3. hostnamectl set-hostname jenkins server 
4. bash 
5.yum install git -y
5. dnf install java-17-amazon-corretto -y
6.java -version
7.wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
8.rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
9.dnf install jenkins -y
10.systemctl enable jenkins
11.systemctl start jenkins
12. Use the public ip of the instance  > :8080 to the ip , your jenkins server should open 
13. come on terminal and shoot this command > cat /var/lib/jenkins/secrets/initialAdminPassword
**you will get a password**
14. paste the password 
15. username , password - admin - admin
16. Configure 
17. generate token 
18. copy the token no. 
19. add webhook in the repo where the project is located - url /github-webhook/, application /json 
20. Go to jenkins 
21.  create new item (jenk-job)- freestyle project , git - paste link , build it 

#########################################################################################################################################################################

Now build maven on the jenkins server itself 
22. yum install maven -y
23. mvn -v
24. Go to manage jenkins > tools 
25. add jdk > copy path of java form terminal 
26. add maven installations > untick install automatically > add mvn path from terminal .
27 . save 
28.   Plugins 
29.  Available plugins - Maven 
30 . Installed plugins >git branch source plugin off > restart once no jobs are running .
31. go to  new item > choose maven project > add git repo > save > build now > after build go to webapp-war and add ss

###########################################################################################################################################################################

Tomcat server :
1. sudo su -
2. dnf install java-17-amazon-corretto -y
3. java --version 
4.Go to apache tomcat link and copy tar.gz file choose version 9
5.cd /opt 
6.wget (copy targz link)
7.ls (find targz file)
8.tar -xvzf apache.....   (copy the link from ls)
9.ls 
10.mv apache-tomcat-9.0:76 tomcat (move the file to tomcat folder)
11.ls
12. cd 
13.cd tomcat
14.cd bin
15./startup.sh
16. copy the public IP of server an open it on new tab with port 8080
To resolve 403 access denied go to context.xml 
17.cd..
18. find /-name context.xml
19.(Here we need to edit last two files)
open first file of the last two files with vim
(Since the valve line is only allowing local host we will comment out those lines)
comment <!-- 0:1 -->
save the file
20.edit second file on similar lines
21.find / -name tomcat users.xml 
22.cd .opt 
23.cd apache .....
24. vim conf/tomcat-users.xml
add the roles script
tomcat-users.xml
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-status"/>
  <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
  <user username="developer" password="developer" roles="manager-script"/>
  <user username="tomcat" password="s3cret" roles="manager-gui"/>
wq!
25.cd /bin
26. Go to jenkins > manage jenkins > plugins > available plugins > deploy to container > install without restart
27.manage jenkins > security > system > global > add creds > developer (username) > password
28.manage jenkins > security > system > global > add creds > developer (username)deployer  > password-deployeer
ID - (tomcat cred)
descr - ("")
click on create 
dashboard >> new job >> build and deploy >> maven project >> ok
git
repository (HTTPS)
main
build
goals and option >> clean install
post build action >> deploy war to container 
**/*.war (no space)
container (Tomcat 9x version)
credentials (developer)
URL of tomcat servers
Apply >> save 
(Before clicking build now close the apache tab to avoid any conflict)
paste the public ip:8080
manager app >> web app
Now go to git hub >> web app> webapp> index-jsp > edit > commit changes > build now

##############################################################################################

    After tomcat :
1. Go to jenkins , plugins 
2. publishover ssh > in plugins
3. Create a docker instance ,which is a linux instance , t2.medium , same sec group , same key name .
4. yum update -y
5. sudo yum install -y docker
6. sudo service docker start
7.docker ps
8.ssh-keygen
9. do  aws configure on docker terminal 
10.come on jenkins terminal server
11. do ssh-keygen
12. cd /etc/ssh
13.vim sshd_config , permtlogin yes , prohibited password yes do the same on docker terminal 
14.cd
15.systemctl restart sshd
16.set docker passwd first 
17.ssh-copy-id root@docker ip
ssh-copy-id root@jenkins ip 
18.ssh-copy-root@jenkins on docker terminal .
18.allow security group permission - all icmp4
ping .....
19. jenkinsterminal check no.keys > cat _authkeys... ( 3 keys , already existing , jenkins public key and docker public key)
remove known hosts if they exsit . (docker consists 2 keys ) 
rm- rf  to remove hosts.
18. get private key of jenkins - cd .ssh/ > cat id_rsa , copy this and paste in publish key (>manage jenk> sys > publish over ssh)
18. Go to jenkins dashboard > Manage jenkis > System > ssh server >
19. add ssh > name:jenkins > paste ip a s as hosttname(jenkins) > username root > remote dir /root
20.add ssh> name :docker > paste ip a s of docker 2nd inet> username root >remote dir /root
21. Go to jenkins > tomcat build > postbuild actions > add /root on context path >
22. search ecr on aws > create ecr > 
23. go to tomcat build , check console output and copy the path var/lib/jenkins/workspace/"build-name" add this path to 
go to jenkins > tomcat build >configure > post build actions > send to build artifacts over ssh  > exec commands > rsync .... tomcat build name rsync -avh /var/lib/jenkins/workspace/tomcat-job/* root@docker ip :/opt
cd /opt 
#paste the 4 commands that we get after creating an ecr . 
24.save and apply and build

##################################################################################################################################

Kubernetes:
1. create iam role with ecrfull access,ekspolicy and iam full access
2.yum update -y
3.yum install unzip -y
4.curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
5. unzip awscliv2.zip
6.sudo ./aws/install
7. aws configure
8.Install EKS Tool
  curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
9.sudo mv /tmp/eksctl /usr/local/bin
10.eksctl version
11. Install Kubectl
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
12.sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl 
13. kubectl version --client
14. Create EKS Cluster
    eksctl create cluster --name my-cluster --region region-code --version 1.29 --vpc-public-subnets subnet-ExampleID1,subnet-ExampleID2 --without-nodegroup
15. create node group
     eksctl create nodegroup \
  --cluster my-cluster \
  --region us-east-2 \
  --name my-node-group \
  --node-ami-family Ubuntu2004 \
  --node-type t2.small \
  --subnet-ids subnet-086ced1a84c94a342,subnet-01695faa5e0e61d97 \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 4 \
  --ssh-access \
  --ssh-public-key /root/.ssh/id_rsa.pub
16.open sshd config file and make
   PasswordAuthentication yes
   PermitRootLogin yes
17.systemctl restart sshd
18.ssh-copy-id root@ip of jenkins on kubectl 
do ssh-copy-id root@ip of kubectl on jenkins
19.create deployment file
    in deployment.yaml file
    apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-project-deployment
  labels:
    app: regapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: regapp
  template:
    metadata:
      labels:
        app: regapp
    spec:
      containers:
      - name: regapp
        image: copy latest image from ecr here
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
20.create service file
   in service.yaml file
   apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app: webapp
spec:
  selector:
    app: webapp
  ports:
    - port: 8080
      targetPort: 8080
  type: LoadBalancer
21.kubectl apply -f deployment.yaml
22.kubectl apply -f service.yaml
23.kubectl expose deployment/deploymentname --type="LoadBalancer" --port 8080
24.kubectl get svc
expose your application using external ip address
     
    
    
