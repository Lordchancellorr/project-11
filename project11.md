## Documentation of Project 11
**ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10**
- 
- Please  note, i used two different terminals for this project due to an error that surfaced with one of the terminals(Windows Powershell) so i had to switch to Gitbash for the completion of the project (Nothing was impaired/disrupted). You are advised to start with gitbash so you won's experirence the same glitch. Thanks.
- In previous projects, i have  performed a lot of manual operations to set up virtual servers, install and configure required software, deploy your web application. However, this prject will take us a step higher by automating most of the processes we performed before manually and seamlessly using [Ansible Configuration Management](https://www.redhat.com/en/topics/automation/what-is-configuration-management#:~:text=Configuration%20management%20is%20a%20process,in%20a%20desired%2C%20consistent%20state.&text=Managing%20IT%20system%20configurations%20involves,building%20and%20maintaining%20those%20systems.) and writing codes effortlessly using [YAML](https://en.wikipedia.org/wiki/YAML). Quickly, we would develop Ansible scripts to simulate the use of a [Jump box/Bastion host](https://en.wikipedia.org/wiki/Bastion_host) to access our Web Servers. 

- *INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE*
- - - 
- Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible which will eventually be used to run the ansible-playbook command
- In your GitHub account create a new repository and name it ansible-config-mgt.
- Install Ansible with `sudo apt install ansible` and confirm the version with `ansible --version.` check the image below for the expected output! ![ansible output](./images/Installation%20if%20ansible%20and%20status%20confirmation.png)
- Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in [Project 9](https://github.com/Lordchancellorr/project-9) (click to read docummentation for beter understanding)
- Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository. Configure Webhook in GitHub and set webhook to trigger ansible build. Configure a Post-build job to save all (**) files. 
Test your setup by making some change in **README.MD** file in *main* branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder - `ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` Check the images below

![Successful pull](./images/Successful%20pull.png)  ![Console Output](./images/Console%20Output.png) ![Confirmation of the builds](./images/Changes%20made%20on%20Github.png)
 
We will stop here and take a moment to set up our environmrnt. As a developer, at some ppoint in time, you bmight have to write codes and you will need  proper tools that will make your coding and debugging comfortable. Simply put, you need an Integrated development nt also known as IDE or a Source Code Editor. There is however a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code [VSC](https://code.visualstudio.com/download). After you might have downloaded and installed it, connect your new repository to it by cloning it using `git clone+your repository link`

-----------------------------------

- **ANSIBLE DEVELOPMENT**

  - In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
  - Checkout the newly created feature branch to your local machine and start building your code and directory structure
  - Create a directory and name it playbooks – it will be used to store all your playbook files.
  - Create a directory and name it inventory – it will be used to keep your hosts organised. Within the playbooks folder, create your first playbook, and name it common.yml
  - Within the inventory folder, create an inventory file (.yml) for each environment (**Development, Staging Testing and Production) dev, staging, uat, and prod** respectively).

- SETTING UO THE ANSIBLE INVENTORY
- What is ansible inventory? An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory. Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. You will need to setup SSH agent and connect VS Code to your Jenkins-Ansible instance. if you don't know how, watch these videos for guide- [SSH for windows](https://youtu.be/OplGrY74qog) and for [Linux](https://youtu.be/OplGrY74qog). Now run eval `ssh-agent -s` and `ssh-add <path-to-private-key>`. You can confirm if it's being added with `ssh-add -l`
- Now, ssh into your Jenkins-Ansible server using ssh-agent using `ssh -A ubuntu@(public IP)` (Please ensure you confirm your username from AWS console to know it it is ubuntu or redhat)
- Update your *inventory/dev.yml* file with this snippet of code: (This does not have to neccessarily be your format)
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

````
It's time to update the playbooks/common.yml file with following code: (Note that your format might be different from mine depending on the user you bused in creating your instances)
```
---
- name: update web and  nfs 
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB and DB server
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
Now it is time to commit and oush the files to the github account previously created. After that, Create a Pull request (PR)
Wear a hat of another developer for a second, and act as a reviewer.
If the reviewer is happy with your new feature development, merge the code to the main branch.
Head back on your terminal, checkout from the feature branch into the main, and pull down the latest changes.
- Once your code changes appear in main branch – Jenkins will do its job and save all the files (build artifacts) to `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on Jenkins-Ansible server. Finally, In your `archive folder` run `ansible-playbook -i inventory/dev.yml playbooks/common.yml`. If it was successful, the image below should be your output. (Ensure you exit and login to these servers using ssh) ![ansible-playbook](./images/ansible-playbook%20command.png)
You can see that the tasks were successfully created. See the images below for confirmation of the tasks[wireshack version] on all the servers
![NFS](./images/wireshark%20version%20on%20nfs.png) ![Database](./images/wireshark%20version%20on%20Database.png) ![Webserver 1](./images/wireshark%20version%20on%20web1.png) ![Webserver 2](./images/wireshark%20version%20on%20web2.png)
## Optional Tasks

- Update your ansible playbook with some new Ansible tasks and go through the full checkout `-> change codes` `-> commit ->` `PR` `-> merge` `-> build` `-> ansible-playbook` cycle again to see how easily you can manage a servers fleet of any size with just one command!
- I added updated my `common.yml` file with the following code
```

- name: create directory, file and set timezone on all servers
  hosts: webservers, nfs, db, lb
  become: yes
  tasks:

      - name: create a directory
        file:
          path: /home/sample
          state: directory
  - name: create a file
    file:
        path: /home/sample/ansible.txt
        state: touch

    name: set: timezone
     timezone:
        name: Africa/Lagos
```
The following images were the results! 
![New Tasks](./images/Added%20tasks%20successfully%20created.PNG)
Check the mind-blowing effect it had on two of the servers below! Focus on the time the screenshot were taken and teh time displayed in the output on the terminal!

![NFS](./images/NFS%20tasks%20confirmation.PNG)

![Database](./images/Database%20tasks%20confirmation.PNG)

- We have now come to the end of Project 11. Thanks for your time! Have a nice day