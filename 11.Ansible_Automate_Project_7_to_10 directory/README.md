## ANSIBLE CONFIGURATION MANAGEMENT

Ansible Client as a Jump Server (Bastion Host)

A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.

On the diagram below the Virtual Private Network (VPC) is divided into two subnets – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.

![](image)


## Step 1 Installing Ansible On Jenkins Server

We install ansible on our jenkins server and rename it to ``Jenkins-Ansible``

```bash
sudo apt update

sudo apt install ansible
```

![](image)


Create a new repository called ``ansible-config-mgt`` on github and set up webhooks on it.

``https://<jenkins_url:port/github-webhooks>``

On the Jenkins server, create a job called ``ansible`` and configure automatic builds when a trigger is made on the ``ansible-config-mgt`` directory via GITScm polling.


![](image)



Test configuration by updating a README file on github.

![](image)


## Step 2 Prepare Development Using VScode

1. First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSC).

2. After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

3. Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance


```bash
git clone <ansible-config-mgt repo link>
```

![](image)


## Step 3 Begin Ansible Configuration

Clone ``ansible-config-mgt`` repo on local machine and create a new branch for development

![](image)

1. In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

**Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)**

2. Checkout the newly created feature branch to your local machine and start building your code and directory structure

3. Create a directory and name it playbooks – it will be used to store all your playbook files.

4. Create a directory and name it inventory – it will be used to keep your hosts organised.

5. Within the playbooks folder, create your first playbook, and name it common.yml

6. Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.


![](image)


## Step 4 Setting Up Ansible Inventory

we create inventories to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

We need to ssh into our target servers defined in the ``/inventory/dev.yaml``

using SSH-Agent to upload our ssh public key to the jenkins-ansible server

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from ``Jenkins-Ansible host`` – for this you can implement the concept of ssh-agent. Now you need to import your key into ``ssh-agent``:

```bash
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```
Confirm the key has been added with the command below, you should see the name of your key

```bash
ssh-add -l
```

Now, ssh into your ``Jenkins-Ansible`` server using ssh-agent

```bash
ssh -A ubuntu@public-ip
```

Also note, that your Load Balancer user is ``ubuntu`` and user for RHEL-based servers is ``ec2-user``.

Update your ``inventory/dev.yml`` file with this snippet of code:


```bash
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

## Step 5 Create A Common Playbook

It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in ``inventory/dev``.

In ``common.yml`` playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your ``playbooks/common.yml`` file with following code:

```bash
---
- name: update web, nfs and db servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb, db
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


## Step 6 Update GIT With The Lastest Code

Push all the changes made to the directories and files from local machine to Github.

Commit your code into GitHub:

1. Use git commands to add, commit and push your branch to GitHub.

```bash
git status

git add <selected files>

git commit -m "commit message"
```

Create a Pull request (PR)

Wear a hat of another developer for a second, and act as a reviewer.

If the reviewer is happy with your new feature development, merge the code to the master branch.

Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.

Once the code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to ``/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`` directory on ``Jenkins-Ansible`` server as shown below.

![](image)


## Step 7 Run First Ansible Test


Now, it is time to execute ansible-playbook command and verify if your playbook actually works:

Connect to your jenkins-ansible server via VScode (configure the .ssh/config file with your jenkins server information)

```bash
ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml
```


![](image)


At the end of this project we have implemented a solution that is shown below


![](image)
