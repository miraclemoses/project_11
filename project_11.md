# AWESOME DOCUMENTATION OF PROJECT_11 ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10


### In Projects 7 to 10 you had to perform a lot of manual operations to seet up virtual servers, install and configure required software, deploy your web application.

### This Project will make you appreciate DevOps tools even more by making most of the routine tasks automated with Ansible Configuration Management, at the same time you will become confident at writing code using declarative language such as YAML.

## Ansible Client as a Jump Server (Bastion Host)
### A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.

__On the diagram below the Virtual Private Network (VPC) is divided into two subnets – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.__

![baston](/bastion.png)

__Before we start create 4 Red-hat instances and 2 ubuntu instances_
tag names for red-hat instances
* db
* webserver1
* webserver2
* NFS
tag names for ubuntu
* lb
* Jenkins-Ansible
# Task
* __Install and configure Ansible client to act as a Jump Server/Bastion Host__

* __Create a simple Ansible playbook to automate servers configuration__

## STEP 1: INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE WITH THE FOLLOWING


1. Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. 
2. We will use this server to run playbooks.
In your GitHub account create a new repository and name it ansible-config-mgt.
3. Instal Ansible
```
sudo apt update

sudo apt install ansible
```

__check out if ansible is installed using:__
` ansible --version`

__output__
![ansible](/ansible%20ver.PNG)

4. Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.

* Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
* Configure Webhook in GitHub and set webhook to trigger ansible build.
* Configure a Post-build job to save all (**) files, like you did it in Project 9.

5. Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

* Run this command to see your files

`ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

## Step 2 – Prepare your development environment using Visual Studio Code

* First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSC), you can get it here.

* After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

* Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance

__Run this comand to clone repo__
`git clone <ansible-config-mgt repo link>`

## STEP3: BEGIN ANSIBLE DEVELOPMENT
* In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

__hint: Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)__

*  Checkout the newly created feature branch to your local machine and start building your code and directory structure

* Create a directory and name it playbooks – it will be used to store all your playbook files.
* Create a directory and name it inventory – it will be used to keep your hosts organised.
* Within the playbooks folder, create your first playbook, and name it common.yml
* Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

## Step 4 – Set up an Ansible Inventory
An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

__Save below inventory structure in the inventory/dev file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.__

* Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent: do this by running the following command

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
ssh-add -l
```
_then after that ssh into the instance_
by running :

`ssh -A username@ip_addr`

_after logging in, add ssh key for persistance_

`ssh-add -l`

__Update your inventory/dev.yml file with this snippet of code:__

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

__ssh into other instances__

```ssh ec2-user@ip_addr```

result
![ec2](/ec2.PNG)
## CREATE A COMMON PLAYBOOK

It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.

In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

* Update your playbooks/common.yml file with following code:

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

## Step 6 – Update GIT with the latest code

* use git commands to add, commit and push your branch to GitHub.
```
git status

git add <selected files>

git commit -m "commit message"
Create a Pull request (PR)
```

If the reviewer is happy with your new feature development, merge the code to the master branch.

Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.

Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.

## RUN FIRST ANSIBLE TEST
Now, it is time to execute ansible-playbook command and verify if your playbook actually works:

```
cd ansible-config-mgt

ansible-playbook -i inventory/dev.yml playbooks/common.yml
```
_output_

![playbook](/playbook.PNG)

* You can go to each of the servers and check if wireshark has been installed by running which wireshark or wireshark --version

_output__
![wr](/wireshark.PNG)

## Your updated with Ansible architecture now looks like this:
![arch](/ansible_architecture.png)

## Congratulations
You have just automated your routine tasks by implementing your first Ansible project! There is more exciting projects ahead, so lets keep it moving!

![grt](/greatjob11.png)
