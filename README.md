sudo su

apt update

#install terraform

wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform


#(check the installation)
terraform --version 

#create a directory 

mkdir project
cd project


#create i am user in aws

#goto aws create iam user
#Admin access
 
#next 
 
#create user
#Click on the user
 
#click on security credentials
#Create accesskey and secret acces key

vi connection.tf

#paste the below code

provider "aws" {
  region     = "eu-north-1"
  access_key =# access key
  secret_key = #secret key
}


vi ec2.tf



data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical
}

# Jenkins Instance
resource "aws_instance" "jenkins" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.medium"

  tags = {
    Name = "Jenkins-Server"
  }
}



apt update -y 
apt-get install -y software-properties-common 
apt-add-repository ppa:ansible/ansible 
apt-get update 
apt-get install -y ansible

vi /etc/ansible/hosts 

 
[ansibledemo] 

Privateip of node1 

Privateip of node2 



adduser devops 

in all the node including master

vi /etc/ssh/sshd_config


vi /etc/ssh/sshd_config.d/60-cloudimg-settings.conf

service ssh restart


visudo 

devops  ALL=(ALL:ALL) NOPASSWD: ALL


su - devops in master
ssh-keygen
cd .ssh 

 ssh-copy-id devops@private ip of node

cd ..

vi inventory.ini  
[all]
#(node private ip)

vi playbook.yml



---
- name: Install Jenkins and Tomcat
  hosts: all
  become: true
 
  vars:
    tomcat_version: "9.0.118"
    tomcat_url: "https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.118/bin/apache-tomcat-9.0.118.zip"
    tomcat_dir: "/opt/apache-tomcat-9.0.118"
 
  tasks:
 
    # -------------------------
    # SYSTEM UPDATE
    # -------------------------
    - name: Update apt cache
      apt:
        update_cache: yes
 
    # -------------------------
    # JAVA + MAVEN
    # -------------------------
    - name: Install Java
      apt:
        name: openjdk-21-jre
        state: present
 
    - name: Install Maven
      apt:
        name: maven
        state: present
 
    # -------------------------
    # JENKINS INSTALLATION
    # -------------------------
    - name: Install required packages
      apt:
        name:
          - fontconfig
          - wget
          - gnupg
        state: present
 
    - name: Create keyrings directory
      file:
        path: /etc/apt/keyrings
        state: directory
        mode: '0755'
 
    - name: Download Jenkins key
      get_url:
        url: https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
        dest: /etc/apt/keyrings/jenkins-keyring.asc
 
    - name: Add Jenkins repo
      copy:
        dest: /etc/apt/sources.list.d/jenkins.list
        content: |
          deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/
 
    - name: Update apt again
      apt:
        update_cache: yes
 
    - name: Install Jenkins
      apt:
        name: jenkins
        state: present
 
    - name: Start Jenkins
      service:
        name: jenkins
        state: started
        enabled: yes
 
    - name: Get Jenkins initial password
      shell: cat /var/lib/jenkins/secrets/initialAdminPassword
      register: jenkins_pass
 
    - name: Show Jenkins password
      debug:
        msg: "Jenkins Initial Password: {{ jenkins_pass.stdout }}"
 
    # -------------------------
    # TOMCAT INSTALLATION
    # -------------------------
    - name: Install unzip
      apt:
        name: unzip
        state: present
 
    - name: Download Tomcat
      get_url:
        url: "{{ tomcat_url }}"
        dest: /opt/apache-tomcat.zip
 
    - name: Unzip Tomcat
      unarchive:
        src: /opt/apache-tomcat.zip
        dest: /opt/
        remote_src: yes
 
    - name: Rename Tomcat folder
      shell: mv /opt/apache-tomcat-{{ tomcat_version }} {{ tomcat_dir }}
      args:
        creates: "{{ tomcat_dir }}"
 
    # -------------------------
    # CHANGE PORT 8080 -> 9090
    # -------------------------
    - name: Change Tomcat port from 8080 to 9090
      replace:
        path: "{{ tomcat_dir }}/conf/server.xml"
        regexp: 'Connector port="8080"'
        replace: 'Connector port="9090"'
 
    # -------------------------
    # START TOMCAT
    # -------------------------
    - name: Give execute permission
      file:
        path: "{{ tomcat_dir }}/bin"
        mode: '0755'
        recurse: yes
 
    - name: Start Tomcat
      shell: "{{ tomcat_dir }}/bin/startup.sh"
 
ansible-playbook -i inventory.ini playbook.yml




the task with master completed

now goto node...

fork the repo from GitHub
copy the repo link from ur GitHub
clone the repo to ur local system
	git clone repo_link

go to repo directory
	cd -----
check the branch
	git checkout -b branch_name
create files:
	touch file1 file2 ....
update to GitHub
	git add .
	git commit -m "added test files"


	git push -u origin branch_name
	user name :
	password :

for password:
		Log into your account on GitHub
		Click your profile picture->settings
		Scroll down on the left -> click Developer settings.
		Click Personal access tokens ->  select Tokens (classic).
		Click Generate new token -> select Generate new token (classic).
		Give it a Note (e.g., "AWS EC2 Server").
		Select the repo scope checkbox (this grants full repository control).
		Scroll to the bottom -> click Generate token.
		Copy the token immediately. You will not see it again.

pull request:
	open the repo
	click pull requests
	new pull req
	Set branches correctly 
		base: master  compare:banch_name
	Create pull request
	Merge
		After clicking it:

		Scroll down
		Click:
		Merge pull request
		confirm merge

go to repo 
go to settings webhook
payload URL: http://publicIP:8080/github-webhook/
content type: change to application/json



Jenkins  

create the new item with the pipeline 

go to configure
in triggers  click on the GitHub hook trigger for GITScm polling and Poll SCM and keep * * * * *
and in pipeline definition 
pipeline script from SCM
and in SCM select GIT
in repo (github repo link)

and in branches to build

keep your branch name

Script Path
jenkinsfile name 

apply and save


go to machine 

visudo
jenkins ALL=(ALL) NOPASSWD: ALL

create jenkinsfile
pipeline {
    agent any
 
    options {
        timeout(time: 10, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
 
    stages {
 
        stage("Clone") {
            steps { 
                git 'github repo link'
                echo 'clone done'
            }
        }
 
        stage("Build") {
            steps {
                sh 'mvn clean package'
            }
        }
 
        stage("Approval") {
            steps {
                input "Approve to deploy"
            }
        }
 
        stage("Deploy to Tomcat") {
            steps {
                sh 'sudo cp target/addressbook.war /opt/apache-tomcat-9.0.118/webapps/'
            }
        }
    }
 
    post {
        success {
            archiveArtifacts artifacts: 'target/*.war'
        }
    }
}

 
esc :wq


git add .
git commit -m "added Jenkinsfile"
git push -u origin branch_name 



if auto build not worked try this
git commit --allow-empty -m "trigger build"
git push origin test
