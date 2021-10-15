# Installing Docker on Amazon Linux server

### Pre-requisites
1. Amazon Linux EC2 Instance

## Installation Steps

1. Install docker and start docker services
   ```sh 
   yum install docker -y
   docker --version 
   
   # start docker services
   service docker start
   service docker status
   ```
1. Create a user called dockeradmin
   ```sh
   useradd dockeradmin
   passwd dockeradmin
   ```
1. add a user to docker group to manage docker 
   ```
   # to check all the groups do following command
   cat /etc/group
   usermod -aG docker dockeradmin
   # to check if the user is part of docker admin group do below
   id dockeradmin
   ```
### Validation test
1. Create a tomcat docker container by pulling a docker image from the public docker registry
   ```sh
   docker run -d --name test-tomcat-server -p 8090:8080 tomcat:latest
   ```
2. Check if container is not running, the webapps folder might be empty
   ```sh
   docker exec -it tomcat-container /bin/bash
   cd webapps
   ls
   # webapps would be empty and needs to be filled
   cd ..
   cd webapps.dist
   ls
   # Contains files which should be moved to web app
   cp -R * ../webapps
   # now files are copied and tomcat server should be visible on browser
   cd ../webapps
   ls
   ```
### Integrating Docker-host with Jenkins

1. Install Publish over SSH plugin in Jenkins
2. Go to system configurations and then to Publish over SSH section
3. Add SSH server with name as docker-host and hostname as IP of docker-host server and username as dockeradmin
4. Now go to advanced and enable use password authentication or use a different key and give the password
5. Click Test Configuration and it will fail because password based access is not enabled on EC2 instances
6. To enable the same if Password Authentication is not set then do this
   ```sh
   vi /etc/ssh/sshd_config
   # find password by /password and change PasswordAuthentication to Yes
   # now restart sshd service for this to take effect
   service sshd reload
   ```
7. Or it can be case that dockeradmin user is not entitled properly in that case try to log in via dockeradmin through putty
   if your password is not accepted you are not successful sudoer
   Also confirm this by logging in as root and then try logging in as other user via sudo su <username>
   You will be able to login but now if you try to login as root via sudo su -  then you will be asked password and when you enter you will get the message "user is not in successful sudoers list".  To make user the successful sudoer change the file named /etc/sudoers
   ```sh
   vi /etc/sudoers
   # insert the following line below the root / admin
   username ALL=(ALL) ALL
   ```
   This will solve the problem.  Now this user can log in even via putty.  Jenkins Test configuration with show success.
   
### Deploy on docker
1. Build a new jenkins job
   A. Copy from Deploy_on_Tomcat_server jenkins job
   B. Remove SCM Poll for Github its not needed
   C. Remove post build action : Deploye War/ear to a container
   D. Add new post build action :  Send build artifacts over SSH
   E. In Transfers in Source Files = webapp/target/*.war  (and do Remove_prefix = webapp/target).  You need to get the path of your build in jenkins server.  So find the workspace in jenkins server through below SSH and then do ls to get navigate to the build
   ```sh
   find / -name "workspace"
   ```
   Now go to the build directory and withing that build war file will lie somewhere in webapp/target.  You need to give this path after build path as your source file.
   F. In Remove_prefix  you can remove the tree and only keep War file
   G. File will be copied in home directory of the dockeradmin user.  Check that to confirm the build.
   
### Deploy on Container
1. Create a docker file provided in documents "Anil docker file for deploy on container job"
2. ```sh
   vi Dockerfile
   ```
3. Save the file
4. Build image from Docker
   ```sh
   docker build -t devops-project .
   ```
5. Run container based on this image
   ```sh
   docker run -d --name devops-container -p 8080:8080 devops-project
   ```
6. Build a new jenkins job  : Deploy_on_Container
   A. Copy from Deploy_on_Docker job
   B. In SSH server -> Transfer (keep everything same but add exec command) -> exec command
   "cd /home/dockeradmin; docker build -t devops-image .; docker run -d --name devops-container -p 8080:8080 devops-image;"  (without quotes)
   C. Before building remove previously created files
   ```sh
   # remove webapp.war
   rm -rf webapp.war
   
   # remove containers
   docker ps -a  
   # to see the containers if any
   # do below for stopping the container first
   docker stop <container id>
   # do below for removing the container
   docker rm <container id>
   
   # remove images
   # to see images
   docker images
   # to remove images 
   docker rmi <image name or image id>
   ```
   D. Now build the job
   E. Verify on docker-host if new image has been created and container is running



## Docker Installation on CentOS server
##### Referance URL : https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository
### Pre-requisites

Please follow below steps to install docker CE on CentoOS server instance. For RedHat only Docker EE available 

1. Install the required packages.

   ```sh 
   sudo yum install -y yum-utils \
   device-mapper-persistent-data \
   lvm2
   ```
  
1. Use the following command to set up the stable repository.
 
   ```sh 
   sudo yum-config-manager \
   --add-repo \
   https://download.docker.com/linux/centos/docker-ce.repo
   ```

### INSTALLING DOCKER CE

1. Install the latest version of Docker CE.
   ```sh 
   sudo yum install docker-ce
   ```

   Note: If prompted to accept the GPG key, verify that the fingerprint matches 
060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35, and if so, accept it.

1. Start Docker.
   ```sh 
   sudo systemctl start docker
   ```

1. Verify that docker is installed correctly by running the hello-world image.
   ```sh
   sudo docker run hello-world
   ```
