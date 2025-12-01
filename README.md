## **Introduction**

This project's goal is to automate the manual operations performed in Projects 7 to 10 using Ansible Configuration Management. By the help of DevOps tools, we will streamline the process of setting up virtual servers, installing and configuring required software, and deploying web applications.

## **Ansible Client as a Jump Server**

A Jump Server is also known as a Bastion Host, it serves as an intermediary for accessing internal networks securely. In our architecture, the webservers reside in a secured (private) network, inaccessible directly from the Internet. DevOps engineers must use the Jump Server to SSH into the webservers, this practice enhances security and reduces the attack surface.

![ansible](/images/ansible39.png)

## **Ansible Installation and Configuration**

-  EC2 Instance Preparation

Update the Name tag on the Jenkins EC2 Instance to Jenkins-Ansible. This server will be used to run Ansible playbooks.

- Github REpository Creation

Create a new repository named *ansible-config-mgt* in your GitHub account.

- Ansible Installation

Install Ansible on the jenkins-ansible server:

```bash
sudo apt update
sudo apt install ansible -y
```
![ansible!](/images/ansible.jpg)

Verify the installation:

```bash
ansible --version
```
![ansible2](/images/ansible2.jpg)

- Jenkins Job Configuration

Configure a Jenkins job save your repository content automatically:

1. Set up a webhook in Github:

- Go to your ansible-config-mgt repository
- Navigate to Settings > Webhooks > Add webhook

![ansible3](/images/ansible3.png)

2. Create a new Freestyle project in Jenkins named ansible:

![ansible4](/images/ansible4.jpg)

3. Configure Source Code Management:

- Choose Git
- Provide the ansible-config-mgt repository URL
- Add credentials for Jenkins to access the repository

![ansible5](/images/ansible5.jpg)

4. Configure Build Triggers:

- Select "GitHub hook trigger for GITScm polling"

![ansible6](/images/ansible6.jpg)

5. Configure Post-build Actions:

- Archive all files: **
![ansible7](/images/ansible7.jpg)

5. Testing the Setup

Make a change to the README.md file in the main branch and ensure that the build starts automatically.

![ansible8](/images/ansible8.jpg)

![ansible9](/images/ansible9.jpg)

* Allocate an Elastic IP to your Jenkins-Ansible server to maintain a consistent IP address for your GitHub webhook configuration.

![ansible10](/images/ansible10.jpg)

![ansible11](/images/ansible11.jpg)


## **Development Environment Preparation**

1. Install Visual Studio Code

Download and install [Visual Studio Code](https://code.visualstudio.com/download) as your integrated Development Environment(IDE)

2. Configure VS Code for GitHub

Connect VS Code to your newly created GitHub repository:

![ansible12](/images/ansible12.jpg)

## **Ansible Development**

1. Create Feature Branch

Create a new branch for Development:

```bash
git checkout -b feature/prj-11-ansible-config
```
![ansible13](/images/ansible13.jpg)

2.  Create Playbook and Inventory Files

Create the initial playbook and inventory files:

```bash
touch playbooks/common.yml
touch inventory/dev.ini inventory/staging.ini inventory/uat.ini inventory/prod.ini
```
![ansible14](/images/ansible14.jpg)

![ansible15](/images/ansible15.jpg)

![ansible16](/images/ansible16.jpg)

## Ansible Inventory Setup

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. We will set up the inventory to organize our servers for different environments.

## *8SSH Agent Configuration**

Ansible uses SSH to connect to target servers. Set up SSH agent to manage your SSH keys:

```bash
eval `ssh-agent -s`
ssh-add <path-to-private-key>
ssh-add -l
```

```bash
Get-Service ssh-agent | Set-Service -StartupType Automatic

Start-Service ssh-agent

Get-Service ssh-agent

ssh-add "Path/to/key.pem"
```
![ansible17](/images/ansible17.jpg)

![ansible18](/images/ansible18.jpg)

![ansible19](/images/ansible19.jpg)

![ansible20](/images/ansible20.jpg)

Connect to your Jenkins-Ansible server using SSH agent:

```bash
ssh -A ennoiaconcept@<public-ip>
```
![ansible21](/images/ansible21.jpg)

## **Update Inventory File**

Update your inventory/dev.ini file with the following structure:

```bash
all:
  children:
    nfs:
      hosts:
        <NFS-Server-Private-IP-Address>:
          ansible_ssh_user: ec2-user
    webservers:
      hosts:
        <Web-Server1-Private-IP-Address>:
          ansible_ssh_user: ec2-user
        
    db:
      hosts:
        <Database-Private-IP-Address>:
          ansible_ssh_user: ubuntu
```

![ansible22](/images/ansible22.jpg)

## **Creating a Common Playbook**

Create a common playbook to perform tasks on all servers listed in the inventory.

Update your playbooks/common.yml file:

```bash
---
- name: Update web and NFS servers
  hosts: webservers, nfs
  remote_user: ubuntu
  become: true
  become_user: root
  tasks:
    - name: Ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: Update LB and DB servers
  hosts: lb, db
  remote_user: ubuntu
  become: true
  become_user: root
  tasks:
    - name: Update apt repo
      apt:
        update_cache: yes

    - name: Ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

```

![ansible24](/images/ansible24.jpg)

This playbook installs or updates wireshark on both RHEL 9 and Ubuntu servers, using the appropriate package manager for each OS. It then creates a directory named Kosenuel_directory and a file named Kose_file.txt inside the directory. After that, it runs a script on the webservers to store some dynamic greeting message in a file named server_greeting.txt in the /tmp/ directory.


## **Version Control with Git**

## **Commit and Push Changes**

Use Git commands to add, commit, and push your changes:

```bash 
git status
git add <selected files>
git commit -m "Initial commit: add inventory and common playbook"
git push origin feature/prj-11-ansible-config
```
![ansible25](/images/ansible25.jpg)

![ansible26](/images/ansible26.jpg)

## **Update Local Repository**

After merging, update your local main branch:

```bash
git checkout master
git pull origin master
```
![ansible27](/images/ansible27.jpg)

Jenkins will automatically build and archive the changes:

![ansible28](/images/ansible28.jpg)

## **Executing Ansible Playbook**

Setup Remote Development in VS Code

1. Install the Remote Development and Remote - SSH extensions in VS Code.

![ansible29](/images/ansible29.jpg)

## **Run Ansible Playbook**

Execute the ansible playbook:

```bash
ansible playbook -i inventory/dev.ini playbooks/common.yml
```
![ansible30](/images/ansible30.jpg)

## **Verify Installation**

Check if wireshark is installed on each server:

```bash
which wireshark
wireshark --version
```
![ansible31](/images/ansible31.jpg)

[ansible32](/images/ansible32.jpg)

[ansible33](/images/ansible33.jpg)

## **Conclusion**

We have successfully automated our routine tasks by implementing Ansible configurations. The updated architecture now looks like this:

![ansible35](/images/ansible35.png)

Copy-ansible-artifact












