# Ansible-Refactoring
## Ansible refactoring and static assignments (imports and roles).



Refactoring in Ansible involves restructuring and rewriting your Ansible code to make it more maintainable, and reusable.




We will refactor our Ansible code, create assignments and earn how to use imports functionality. 

Imports allows you to effectively re-use previously created playbooks in a new playbook and it allows you to organize your tasks and re-use them when needed.


For better understanding of Ansible artifacts re-use, [read this](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html)


We will continue working with the `ansible-config-mgt` in my github repository and make some improvements to the code.



__Code refactoring:__ Refactoring code is the process of restructuring or modifying the existing code to improve its readability, maintainability, and sometimes performance, without changing its external behavior. The goal is to make the code more efficient, understandable, and easier to maintain over time. 









## Refactor Ansible code by importing other playbooks into site.yml


__step 1:__ JENKINS JOB ENHANCEMENT

- Let us make some changes to our previous jenkins job.

- we will require `Copy Artifact` Plugin,it's a good choice for managing artifacts between Jenkins jobs.

- In our __"Jenkins-Ansible server"__, We create a new directory called __"ansible-config-artifact"__ where we will store all artifacts after each build.


`sudo mkdir ansible-config-artifacts`

Then we will Change permissions to this directory, so Jenkins can save files in the ansible-config-artifact.

`sudo chmod -R 777 ansible-config-artifact`

![](./images/1.png)





Go to Jenkins web console click on __Manage Jenkins__ then click on __Manage Plugins__.


![](./images/2.png)


On Available tab search for `copy artifacts` and install the plugin without restarting jenkins.

![](./images/3.png)


Then create a new free style projects and name it `save_artifacts`

![](./images/4.png)



This project will be triggered by completion of your existing ansible project. Configure it accordingly:

Go to configurations, make the following changes.

We can configure the number of builds to keep in order to save space on the server. In this project we want to keep the last 2 builds.



![](./images/5.png)



Then we update the source code management section.


![](./images/7.png)




The main idea of __save_artifacts__ project is to save artifacts into __/home/ubuntu/ansible-config-artifact__ directory. To achieve this, we will create a __Build step__ and choose __Copy artifacts__ from our previous my-job project. We will specify my-job as a source project and __/home/ubuntu/ansible-config-artifact__ as a target directory.



![](./images/8.png)


![](./images/9.png)


Then click on apply and save.











__Problem encountered__


```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/save_artifacts
FATAL: /home/ubuntu/ansible-config-artifact
java.nio.file.AccessDeniedException: /home/ubuntu/ansible-config-artifact
	at java.base/sun.nio.fs.UnixException.translateToIOException(UnixException.java:90)
	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:106)
	at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:111)
	at java.base/sun.nio.fs.UnixFileSystemProvider.createDirectory(UnixFileSystemProvider.java:397)
	at java.base/java.nio.file.Files.createDirectory(Files.java:700)
	at java.base/java.nio.file.Files.createAndCheckIsDirectory(Files.java:807)
	at java.base/java.nio.file.Files.createDirectories(Files.java:793)
	at hudson.FilePath.mkdirs(FilePath.java:3718)
	at hudson.FilePath$Mkdirs.invoke(FilePath.java:1382)
	at hudson.FilePath$Mkdirs.invoke(FilePath.java:1377)
	at hudson.FilePath.act(FilePath.java:1198)
	at hudson.FilePath.act(FilePath.java:1181)
	at hudson.FilePath.mkdirs(FilePath.java:1372)
	at hudson.plugins.copyartifact.CopyArtifact.copy(CopyArtifact.java:704)
	at hudson.plugins.copyartifact.CopyArtifact.perform(CopyArtifact.java:668)
	at hudson.plugins.copyartifact.CopyArtifact.perform(CopyArtifact.java:552)
	at jenkins.tasks.SimpleBuildStep.perform(SimpleBuildStep.java:123)
	at hudson.tasks.BuildStepCompatibilityLayer.perform(BuildStepCompatibilityLayer.java:80)
	at hudson.tasks.BuildStepMonitor$1.perform(BuildStepMonitor.java:20)
	at hudson.model.AbstractBuild$AbstractBuildExecution.perform(AbstractBuild.java:818)
	at hudson.model.Build$BuildExecution.build(Build.java:199)
	at hudson.model.Build$BuildExecution.doRun(Build.java:164)
	at hudson.model.AbstractBuild$AbstractBuildExecution.run(AbstractBuild.java:526)
	at hudson.model.Run.execute(Run.java:1895)
	at hudson.model.FreeStyleBuild.run(FreeStyleBuild.java:44)
	at hudson.model.ResourceController.execute(ResourceController.java:101)
	at hudson.model.Executor.run(Executor.java:442)
Finished: FAILURE
```





__Resolution__

i had to give full permission to /home/ubuntu 

`sudo chmod -R 777 /home/ubuntu`

![](./images/12.png)

And the build was successful. The __save_artifacts__ build in the Jenkins.

![](./images/11.png)


And then i changed the permission back to 755.

`sudo chmod -R 755 /home/ubuntu`

![](./images/13.png)

Test this setup, we will be making some changes in the __README.MD__ file inside the ansible-config-mgt repository.



If the Jenkins jobs was completed, we will see our files inside __/home/ubuntu/ansible-config-artifact__ directory and it will be updated with every commit to the main branch.


![](./images/14.png)





__REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML__



Before starting to refactor the code, We pull down the latest code from main branch and creat a new branch refactor.


![](./images/15.png)


Refactoring is one of the techniques that can be used for constant iterative improvement for better efficiency in DevOps.






In [Project-11](https://github.com/ogechukwu1/Ansible-Automate-Project), all the tasks are written in a single playbook common.yml. These are pretty simple set of instructions for only 2 types of OS. In a case where we have many more tasks and we need to apply this playbook to other servers with different requirements, We will have to read through the whole playbook to check if all tasks written there are applicable and to check if there is anything that needs to be added for certain server/OS families. This will become a tedious exercise and the playbook will become messy which will make it difficult for your colleagues to use your playbook.

Breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

- Within the playbook folder, create a new file and name it __site.yml__. This file will be an entry point into the entire infrastructure configuration. In Other words, `site.yml` will become a parent to all other playbooks that will be developed. Including `common.yml` that you created previously. playbooks will be included here as a reference.



![](./images/16.png)



- Create a new directory within the __ansible-config-mgt__ directory and name it __static-assignment__. The static-assignment directory is where all other children playbooks will be stored.




![](./images/17.png)



Move __common.yml__ file into the newly created __static-assignment__ directory.


![](./images/18.png)


Inside __playbooks/site.yml__ file, import __common.yml playbook__.



```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```

![](./images/19.png)



Run ansible-playbook command against the dev environment


`ansible-playbook playbooks/site.yml -i inventory/dev.ini`



since you need to apply some tasks to your `dev` servers and `wireshark` is already installed . We will go ahead and create another playbook under `static-assignments` and name it `common-del.yml`. In this playbook, configure deletion of wireshark utility. 



![](./images/20.png)



In the static-assignment/common-del.yml we paste the code below




```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes

```


![](./images/21.png)



Update __site.yml__ with `- import_playbook: ..static-assignments/common-del.yml` instead of __common.yml__ and run it against __dev__ servers:




```
---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml
```


![](./images/22.png)




To push the codes to github and merge to the main branch.

We run the command



`git status`

`git add .`

`git commit -m "updated"`

`git push origin refactor`


![](./images/23.png)



We then go to the github and create a pull request


![](./images/24.png)

![](./images/25.png)

![](./images/27.png)




Problem encountered.

I gave full permission to /home/ubuntu but the build wasn't successful so i removed full permission and gave it 755 and it was successful. I also gave jenkins the ownership of the destination directory.

`sudo chmod -R 755 /home/ubuntu`

`sudo chown -R jenkins:jenkins /home/ubuntu/ansible-config-artifact`



![](./images/26.png)

Resolution

![](./images/28.png)





















































































































































































































































