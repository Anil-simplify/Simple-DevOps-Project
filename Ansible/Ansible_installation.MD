# Ansible Installation

Ansible is an open-source automation platform. It is very, very simple to set up and yet powerful. Ansible can help you with configuration management, application deployment, task automation.

### Pre-requisites

1. An AWS EC2 instance (on Control node)

### Installation steps:
#### on Amazon EC2 instance

1. Install python and python-pip
   ```sh
   yum install python
   yum install python-pip
   ```
1. Install ansible using pip check for version
    ```sh
    pip install ansible
   ansible --version
   ```
   
1. Create a user called ansadmin (on Control node and Managed host)  
   ```sh
   useradd ansadmin
   passwd ansadmin
   ```
1. Below command grant sudo access to ansadmin user. But we strongly recommended using "visudo" command if you are aware vi or nano editor.  (on Control node and Managed host)
   ```sh
   echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
   ```
   
1. Log in as a ansadmin user on master and generate ssh key (on Control node)
   ```sh 
   sudo su - ansadmin
   ssh-keygen
   ```
1. Copy keys onto all ansible managed hosts (on Control node)
   ```sh 
   ssh-copy-id ansadmin@<target-server>
   ```
   A.  Copy also on the locahost
   ```sh 
   ssh-copy-id localhost
   ```
   B.  Create file hosts inside /etc/ansible  #this will contain hosts
   ```sh
   # add following lines to the file
   <docker-host ip addr>
   localhost
   ```
   C. Validate the set up by pinging both the hosts
   ansible all -m ping
 
1. Integrating Jenkins with Ansible
   A. Manage Jenkin -> Configuration > Publish over SSH > Add
   B. Name : ansible-server host: Ansible IP Addr username: ansadmin Check box: Use Password authentication  Enter password : ansadmin password Test : Success Apply: Save
   C. Build a new job : Deploy_on_container_using_Ansible copy from deploy on container
      a. Just change Post Build actions: Send build artifacts over SSH
      b. Name: ansible server
      c. Create docker directory in ansible server and give complete ownership to ansadmin
         ```sh
         sudo mkdir docker
         sudo chown -R ansadmin:ansadmin /opt/docker
         # to validate ownership change check below
         ls -l /opt
         ```
      d. set the remote director in jenkins to "//opt//docker"  # double slash is necessary in this path setting
      e. remove all exec commands for time being
      f. trigger build in jenkins # this should transfer webapp.war in the docker folder in ansible
      g. go to docker folder in ansible do ls and verify
      
   D. Create Ansible playbook
      a. Ansible playbook name simple-devops-project
      ```sh
      # yml file content will be like this
      ---
      - hosts: all
        become: true should be removed.  It won't work if its set to true.

        tasks:

        - name: stop current running container
          command: docker stop simple-devops-container
          ignore_errors: yes

        - name: remove stopped container
          command: docker rm simple-devops-container
          ignore_errors: yes

        - name: remove docker image simple-devops-image
          command: docker rmi simple-devops-image
          ignore_errors: yes

        - name: build docker image using war file
          command: docker build -t simple-devops-image .
          args:
            chdir: /opt/docker

        - name: create container using simple-devops-image
          command: docker run -d --name simple-devops-container -p 8080:8080 simple-devops-image

      ```
      ```sh
      # below is check how ansible playbook would run but it will not run its just how would it run
      ansible-playbook -i hosts simple-devops-image.yml --check
      # below is to actually run the playbook
      ansible-playbook -i hosts simple-devops-project.yml
      # check the container created by below
      docker ps -a
      ```
   E. Pushing docker image through ansible while deploying on target docker-host
      a. Before pushing the image into dockerhub you need to do docker login
      b. create new ansible playbook create-simple-devops-image.yml
      ```sh
      ---
      - hosts: all

        tasks:

        - name: create docker image using war file
          command: docker build -t simple-devops-image:latest .
          args:
            chdir: /opt/docker

        - name: create tag to image
          command: docker tag simple-devops-image anildocks/simple-devops-image

        - name: push image to docker hub
          command: docker push anildocks/simple-devops-image

        - name: remove image from ansible server after pushing to docker hub
          command: docker rmi simple-devops-image:latest anildocks/simple-devops-image
          ignore_errors: yes
       ```
         
   F. Ansible playbook for Pulling docker image from docker hub and deploying on container
      a. Rename the yml file
         ```sh
         mv simple-devops-project.yml create-simple-devops-project.yml
         ``
      b. Final Ansible playbook yml will be below
         ---
         - hosts: all

           tasks:

           - name: stop current running container
             command: docker stop simple-devops-container
             ignore_errors: yes

           - name: remove stopped container
             command: docker rm simple-devops-container
             ignore_errors: yes

           - name: remove docker image simple-devops-image
             command: docker rmi anildocks/simple-devops-image:latest
             ignore_errors: yes

         # - name: build docker image using war file
         #   command: docker build -t simple-devops-image .
         #   args:
         #     chdir: /opt/docker

           - name: pull docker image from docker hub
             command: docker pull anildocks/simple-devops-image:latest

           - name: create container using simple-devops-image
             command: docker run -d --name simple-devops-container -p 8080:8080 anildocks/simple-devops-image:latest
   
   G. Integrating in Jenkins
      a. In the Send artifacts over SSH -> Transfer -> exec command type this in
         "ansible-playbook -i /opt/docker/hosts /opt/docker/create-simple-devops-image.yml --limit  localhost;
          ansible-playbook -i /opt/docker/hosts /opt/docker/create-simple-devops-project.yml --limit 172.31.37.87;"
      b. Build the job
      c. Now its done Hurray

      
       

1. Ansible server used to create images and store on docker registry. Hence install docker, start docker services and add ansadmin to the docker group. 
   ```sh
   yum install docker
   
   # start docker services 
   service docker start
   service docker start 
   
   # add user to docker group 
   usermod -aG docker ansadmin

   ```
1. Create a directory /etc/ansible and create an inventory file called "hosts" add control node and managed hosts IP addresses to it. 
 
### Validation test

   
1. Run ansible command as ansadmin user it should be successful (Master)
   ```sh 
   ansible all -m ping
   ```
