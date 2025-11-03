## **1. Introduction**

This project focuses on improving our Ansible configuration management setup. We will refactor our Ansible code, create assignments, and utilize the imports functionality. Imports allow us to effectively reuse previously created playbooks in new playbooks, enabling better organization and reusability of our tasks.

## **2. Jenkins Job Enhancement**
To streamline our Jenkins setup, we will create a new job to store artifacts in a centralized location. This approach will improve organization and conserve space on the Jenkins server.

*2.1 Create Artifact Directory*

- SSH into your Jenkins-Ansible server and create a new directory.

```bash
sudo mkdir /home/ubuntu/ansible-config-artifact
```
- Set appropriate permissions so that Jenkins can write to the directory:

```bash
sudo chmod -R 0777 /home/ubunru/ansible-config-artifact
```
![ansible!](/images/ansible1.jpg)

*2.2 Install Copy Artifact Plugin*

1. Navigate to Jenkins web console > Manage Jenkins > Manage Plugins
2. Search for *Copy Artifact* in the Available tab and install without restarting Jenkins

![ansible2](/images/ansible2.jpg)

*2.3 Create and Configure save_artifacts Job*

1. Create a new Freestyle project named *save_artifacts*
2. In the Build Triggers section, configure it to be triggered upon completion of the existing ansible project.
3. In the Build step, choose "Copy artifacts from other project"
  - Source project: ansible
  - Target directory: /home/ubuntu/ansible-config-artifact

*2.4 Verify Setup*
1. Make a minor change to the README.md file in your ansible-config-mgt repository
2. Observe both Jenkins jobs running sequentially
3. Verify that files are copied to /home/ubuntu/ansible-config-artifact

If you encounter permission issues, add the Jenkins user to the ubuntu group:

```bash
sudo usermod -a -G Jenkins Ubuntu
```
![ansible3](/images/ansible3.jpg)
![ansible4](/images/ansible4.jpg)
![ansible5](/images/ansible5.jpg)

## **3. Refactoring Ansible Code**

We will refactor our Ansible code to improve organization and reusability.

*3.1 Create New Branch*

Create a new branch named refactor in your ansible-config-mgt repository using this command:

```bash
git checkout -b refactor
```
![ansible6](/images/ansible6.jpg)

*3.2 Reorganize Directory Structure*

1. Create a new folder naamed *static-assignments* in the repository root
2. Move the *common.yml*  file into the static-assignments folder
3. Create a new file *site.yml* in the playbooks folder.

![ansible7](/images/ansible7.jpg)

![ansible8](/images/ansible8.jpg)

Your folder structure should now look like this:

```bash
ansible-config-mgt/
├── static-assignments/
│   └── common.yml
├── inventory/
│   ├── dev
│   ├── stage
│   ├── uat
│   └── prod
└── playbooks/
    └── site.yml
```
*3.3 Updates site.yml*

Edit *site.yml* to import the common.yml playbook:

```bash
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
*3.4 Create common-del.yml*

1. Create common-del.yml in the static-assignments folder
2. Add tasks to remove wireshark from all servers:

```bash
---
- name: Delete Wireshark from all servers
  hosts: all
  remote_user: ubuntu
  become: true
  become_user: root

  tasks:
    - name: Ensure Wireshark is completely removed
      apt:
        name:
          - wireshark
          - wireshark-common
          - wireshark-qt
        state: absent
        purge: yes
        autoremove: yes
```
3. Update *site.yml* to use *common-del.yml*:

```bash
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
*3.5 Run Playbook*
Execute the playbook against the dev environment:

```bash
cd /home/ubuntu/ansible-config-mgt/
ansible-playbook -i inventory/dev.ini playbooks/site.yaml
```
Verify that wireshark has been removed from all servers by running:

```bash
wireshark --version
```
![ansible9](/images/ansible9.jpg)

![ansible10](/images/ansible10.jpg)

## **4. Configuring UAT Webservers.
We will now set up two new UAT webservers using a dedicated role.

*4.1 Launch UAT Instances*

Create two EC2 instances using RHEL 9 image. Name them Web1-UAT and Web2-UAT.

![ansible11](/images/ansible11.jpg)

*4.2 Create Webserver Role*

1. Create a roles directory in your ansible-config-mgt repository
2. Initialize the webserver role:

```bash
mkdir roles
cd roles
ansible-galaxy init webserver
```
![ansible12](/images/ansible12.jpg)

3. Remove unnecessary directories (tests, files, vars)

![ansible13](/images/ansible13.jpg)
![ansible14](/images/ansible14.jpg)

*4.3 Update Inventory*

Add the UAT webservers to your inventory file ansible-config-mgt/inventory/uat.ini:

```bash
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
```
** 4.4 Configure Ansible**
Update /etc/ansible/ansible.cfg or ~/.ansible.cfg or the ansible.cfg file in your playbooks directory to include the roles path:

```bash
roles_path = /home/ubuntu/ansible-config-mgt/roles
```
## ** Implementing the Webserver Role**

**5.1 Define Webserver Tasks**

Edit roles/webserver/tasks/main.yml:

```bash
---
- name: Install Apache
  become: true
  ansible.builtin.apt:
    name: "apache2"
    state: present

- name: Install Git
  become: true
  ansible.builtin.apt:
    name: "git"
    state: present

- name: Clone repository
  become: true
  ansible.builtin.git:
    repo: https://github.com/heritageolaleye/tooling.git
    dest: /var/www/html
    force: yes

- name: Copy HTML content
  become: true
  ansible.builtin.command: cp -r /var/www/html/html/ /var/www/

- name: Start Apache service
  become: true
  ansible.builtin.service:
    name: apache2 
    state: started

- name: Remove cloned directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```
**5.3 Update site.yml**

Update playbooks/site.yml to include the new assignment:

```bash
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
  import_playbook: ../static-assignments/uat-webservers.yml
```
## **Testing and Deployment**

**6.1 Commit changes**
Commit your changes by running this command:

```bash
git commit -m "Refactoring and static assignments"
```
Create a Pull Request by running this command:

```bash
git push --set-upstream origin refactor
```
![ansible16](/images/ansible16.jpg)

![ansible17](/images/ansible17.jpg)

![ansible18](/images/ansible18.jpg)

**6.2 Run Jenkins Jobs**

Ensure that the webhook triggers two consecutive Jenkins jobs and that they run successfully.
![ansible19](/images/ansible19.jpg)

![ansible20](/images/ansible20.jpg)

**6.3 Execute Playbook**

Run the playbook against the UAT inventory:

```bash
cd /home/ubuntu/ansible-config-artifact
ansible-playbook -i inventory/uat.ini playbooks/site.yml
```
![ansible21](/images/ansible21.jpg)

**6.4 Verify Deployment**

Access the UAT webservers through a web browser:

```bash
http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
http://<Web2-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
```
![ansible22](/images/ansible22.jpg)

![ansible23](/images/ansible23.jpg)

## **Conclusion**

With this, we come to the end of the Ansible refactoring and implementation of static assignments/project. This new structure we implemented improves code organization, reusability, and maintainability. The Ansible architecture now includes dedicated roles and a more modular approach to configuration management.

![ansible24](/images/ansible24.png)












