# ansible-config-mgt

In this project, we will be auntomating a number of activities on several servers uing Ansible configuration management. This implies that we can automate many of our activities and processes on our webservers and tool setups without going to the individual servers using Ansible client as a jump server(Bostian Host).

**The new architecture of the setup**
![Screenshot from 2023-03-29 17-39-45](https://user-images.githubusercontent.com/77943759/228608623-38dc6d1c-814c-4951-83b2-783979534af9.png)
                                              
Task
Install and configure Ansible client to act as a Jump Server/Bastion Host
Create a simple Ansible playbook to automate servers configuration


## **INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE**

Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.

![jenkins-ansible](https://user-images.githubusercontent.com/77943759/228610659-db4a5ad7-0c23-42f0-8b45-2248f1d881e5.png)


In GitHub account create a new repository and name it ansible-config-mgt.

Install Ansible

```
sudo apt update

sudo apt install ansible
```
![ansibleinstall](https://user-images.githubusercontent.com/77943759/228625714-354afc3f-bcfb-4c29-a5e3-3915fe66c10f.png)



Check your Ansible version by running 

`ansible --version`

![ansibleversion](https://user-images.githubusercontent.com/77943759/228625837-ea66cb5e-2041-4f3c-bd84-3b089f2c02f4.png)


**Configure Jenkins build job to save your repository content every time you change it** 

1. Create Elastic ip and attach it to Jenkins-Ansible server. This is because we dont want the public Ip changing every time we power the Instance off and on. Learn how to create elastic ip [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html). Learn how to associate your elastic instance to your webserver [here](https://medium.com/progress-on-ios-development/connecting-an-ec2-instance-with-a-godaddy-domain-e74ff190c233).


2.Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.

![ansiblejob](https://user-images.githubusercontent.com/77943759/228625943-e5310d57-9ee8-4c3b-8999-c96699d5993b.png)


3. Configure Webhook in GitHub and set webhook to trigger ansible build

![webhook](https://user-images.githubusercontent.com/77943759/228629316-4c30870c-9d3d-4f3c-a9c4-8c75757cdc50.png)

4. Configure a Post-build job to save all (**) files

![artifacts](https://user-images.githubusercontent.com/77943759/228681107-d79f9475-5ef5-4b2e-92ca-8dd2d3225fb7.png)


5. Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

`sudo cat /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

![catreadme](https://user-images.githubusercontent.com/77943759/228681572-5b59fc90-a05b-47df-a3c3-51745c58935e.png)

This is the current setup of the tooling solution at this point

![Screenshot from 2023-03-29 23-28-52](https://user-images.githubusercontent.com/77943759/228682147-703c6287-86b0-400a-916b-bab56620e090.png)


## **Prepare your development environment using Visual Studio Code**

Download and install Visual studio Code which we will use to write our code for the servers [here](https://code.visualstudio.com/download)

Clone down the  ansible-config-mgt repository from Github to your Jenkins-Ansible instance. Open the visual studio code terminal and run:

`git clone <ansible-config-mgt repo link>`

## **BEGIN ANSIBLE DEVELOPMENT**

In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

![branch](https://user-images.githubusercontent.com/77943759/228684146-bd2e18e5-48e5-42c7-ac15-506d83671df6.png)

Checkout the newly created feature branch to your local machine

`git checkout <name-of-branch>`

![gitcheckout](https://user-images.githubusercontent.com/77943759/228684616-5ea08f2f-da4b-4886-8ace-e4aecd642034.png)


Create a directory and name it playbooks – it will be used to store all your playbook files.

Create a directory and name it inventory – it will be used to keep your hosts organised.

Within the playbooks folder, create your first playbook, and name it common.yml

Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.

![fouryml](https://user-images.githubusercontent.com/77943759/228684537-562808b8-4216-4a83-9cbf-4bb2f6d7cc08.png)

## **Set up an Ansible Inventory**

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate

Let us organise our host in the inventory 

Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent: Watch [this video](https://www.youtube.com/watch?v=RRRQLgAfcJw) to know how to learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```
![eval](https://user-images.githubusercontent.com/77943759/228691748-cc878105-fbd1-48b1-a826-0136cca8d3f4.png)


Confirm the key has been added with the command below, you should see the name of your key

`ssh-add -l`

Now, ssh into your Jenkins-Ansible server using ssh-agent

`ssh -A ubuntu@<public-ip>`

![connectviassh](https://user-images.githubusercontent.com/77943759/228691955-f3c6f401-0453-45b5-8cdc-6b09eee88247.png)


Update your inventory/dev.yml file with this snippet of code:

Update the approprite private ip addresses 

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ubuntu' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```
![edityml](https://user-images.githubusercontent.com/77943759/228692081-d59078e7-6312-430f-9ad1-3bd397f9924c.png)


## **CREATE A COMMON PLAYBOOK**

Give Ansible instructions on what you need it to  perform on all servers listed in inventory/dev.

In common.yml playbook we will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

`cd playbooks`

`sudo vi common.yml`

paste the following code

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


This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

## **Update GIT with the latest code**

Commit your code into GitHub

```
git status

git add <selected files>

git commit -m "commit message"
```
![gitstatusadd](https://user-images.githubusercontent.com/77943759/228691073-363a638c-ba8e-4a64-971b-994fb706179a.png)

![pushed](https://user-images.githubusercontent.com/77943759/228691127-ba36a1e3-a196-426b-830c-5dd15e50a805.png)

Create a Pull request (PR). Learn how [here](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)

Merge requests

![mergerequest](https://user-images.githubusercontent.com/77943759/228690832-a317a513-daa8-43d3-98bc-56db062c2232.png)


Return to the terminal and checkout the current branch

Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server

## **RUN FIRST ANSIBLE TEST**

`cd ansible-config-mgt`

`ansible-playbook -i inventory/dev.yml playbooks/common.yml`

![playbooksummary](https://user-images.githubusercontent.com/77943759/228690719-b27cabd8-dc0e-4d70-b989-0648c78147f1.png)


Go to each of the servers and check if wireshark has been installed by running:

`which wireshark`
or 
`wireshark --version`

![Screenshot from 2023-03-30 00-32-15](https://user-images.githubusercontent.com/77943759/228690640-2772ddee-9a53-46b8-a37a-958804b5a424.png)


![Screenshot from 2023-03-30 00-28-29](https://user-images.githubusercontent.com/77943759/228690229-26d4ec7d-ed1c-434f-b165-582c1b211497.png)

This is the new architecture of our web solution

