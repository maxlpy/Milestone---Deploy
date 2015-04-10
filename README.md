DevOps Milestone - Deployment
===================
Team :

 - Nikhil Katre (nkatre@ncsu.edu)
 - Pengyu Li (pli5@ncsu.edu)
 
Submission: **Milestone#Deploy** <br>
Link To Sample Repository Used: [WebGoat](https://github.com/nkatre/WebGoat) <br>
Submission Files:
>  - README.md

Public IP Addresses
-------------
All servers are hosted in AWS using EC2 service

 1. Master = 52.4.40.18
 2. Canary 1 (Blue) = 52.1.163.109
 3. Canary 2 (Green) = 52.5.15.126

Diagram
![ProjectPlan](https://github.com/nkatre/Milestone---Deploy/blob/master/outputImages/diagram.png)


Installations and AWS Settings
-------------
Install the following **ON ALL THREE INSTANCES (master, canary1 and canary2)** to achieve this Milestone
 1.  Download and install [salt](http://docs.saltstack.com/en/latest/topics/installation/)
 2.  Download and Install [git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
 3. ON ALL THREE INSTANCES (master, canary1 and canary2) allow **all traffic** option selected from security group in AWS

Make canary1 and canary2 remote repositories to master
-------------
The following steps should be followed to make canary1 and canary2 as remote repository of master

-- Steps for Master
 1. `ssh` to master using `ssh -i Nikhil.pem ubuntu@52.4.40.18` where `Nikhil.pem` is the AWS key file attached with this submission
 2. Create a folder `Proj@Master` using `mkdir` 
 2. Inside this folder initialize a bare repository using `git init --bare`
 3. Clone [WebGoat](https://github.com/nkatre/WebGoat) repository inside this folder
 4. Transfer the key file Nikhil.pem to master from local using the command :

    scp -i Nikhil.pem Nikhil.pem ubuntu@52.4.40.18
 5. Generate ssh key at master and bind AWS key (Nikhil.pem) with this ssh key as follows

    ssh-keygen 

    cat ~/.ssh/id_rsa.pub | ssh -i Nikhil.pem ubuntu@52.1.163.109 "cat >> .ssh/authorized_keys"

    cat ~/.ssh/id_rsa.pub | ssh -i Nikhil.pem ubuntu@52.5.15.126 "cat >> .ssh/authorized_keys"
6. Bind this key with ssh command at master for authentication

     ssh-add Nikhil.pem

7. Setup SSH at MASTER

```bash
$ ssh-add Nikhil.pem
```

8. Add git remote

```bash
$ git remote add blue ssh://ubuntu@52.1.163.109/blue.git
$ git remote add green ssh://ubuntu@52.5.15.126/green.git
```
# Setup remote git repo on canary1

## Create a bare repository

```bash
$ mkdir blue.git
$ cd blue.git
$ git init --bare
```

## Set GIT_WORK_TREE

```bash
$ mkdir /home/ubuntu/blue-www/
$ cat > hooks/post-receive
#!/bin/sh
GIT_WORK_TREE=/home/ubuntu/blue-www/ git checkout -f


Make post-receive executable:
$ chmod +x hooks/post-receive
```


# Setup remote git repo on canary2

## Create a bare repository

```bash
$ mkdir green.git
$ cd green.git
$ git init --bare
```

## Set GIT_WORK_TREE

```bash
$ mkdir /home/ubuntu/green-www/
$ cat > hooks/post-receive
#!/bin/sh
GIT_WORK_TREE=/home/ubuntu/green-www/ git checkout -f


Make post-receive executable:
$ chmod +x hooks/post-receive
```


Evaluation
-------------

**Milestone#Deploy** is evaluated based on the following
 **Evaluation Parameters:**

 - Automatic deployment environment configuration: 20%
   
 - Deployment of binaries created by build step: 20%
   
 - Remote deployment: 20%
   
 - Canary releasing: 20%
   
 - Canary analysis: 20%

----------

I. Triggered Builds
-------------------

We have used **Poll SCM** feature of Jenkins to achieve build trigger.<br>
The following steps were followed to achieve build trigger.<br>

 - Install Github plugin in Jenkins
 - Goto `Configure` option in Jenkins project.
 - In `Build Triggers`, select the options as shown in the image below

>-  Build when a change is pushed to github
>- Poll SCM

 - In the Schedule, type `*/5 * * * *` which indicates that the git
   repository will be polled for every 5 minutes.<br> If a change is
   identified in the repository then the project will be build in
   Jenkins

![Build Trigger Configuration](https://github.com/nkatre/DevOpsProject/blob/master/Images/buildTrigger.png "Build Trigger Configuration")

 - I create a sample file called **sampleFile** and add this to the project [WebGoat](https://github.com/nkatre/WebGoat)
 - In the next git poll, Jenkins identifies the changes made to the repository and hence builds the project automatically on identifying changes.

**Output of build triggered by changes to Project**
![BuildTriggeredByPoll](https://github.com/nkatre/DevOpsProject/blob/master/Images/pollOutput.png)
![BuildTriggeredByPoll](https://github.com/nkatre/DevOpsProject/blob/master/Images/pollOutput1.png)

----------


II. Dependency Management
-------------

#### <i class="icon-upload"></i> Build dependency and restore to a clean state

 1. We achieve this by configuring maven in Jenkins<br>
 2. In the Project Configuration, we invoke maven and set goal to `clean install`<br>
 3. To demonstrate this, a dependency is added in `pom.xml` file of the
    project<br>
 4. When the project is build then this dependency is also included in
    the project which can be seen in console output in Jenkins.

**Steps for maven configuration**

 5. Install [Maven Project Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Maven+Project+Plugin) in Jenkins
 6. Goto `Manage Jenkins` > `Configure System`.
 7. In `Maven Configuration`, set the following
![Maven Configuration](https://github.com/nkatre/DevOpsProject/blob/master/Images/mj_mavenConfig.png)
 8. Scroll down to `Maven` option and select to install maven automatically.
 9. In `Maven Project Configuration` select `Default` as `Local Maven Repository` as shown in the figure below
 ![Maven Config](https://github.com/nkatre/DevOpsProject/blob/master/Images/mj_mavenConfig.png)
 10. `Save` this configuration

**Steps for clean install**

 11. Goto Dashboard of Jenkins, select Project and click on `Configure`.
 12. Select `Build` > `Add a Build Step` > `Invoke Top Level Maven Targets`
 13. In this, select `Maven Version` as **maven** and in `Goals` we have to write **clean install** as mentioned in the following image.
 ![Maven Clean Install](https://github.com/nkatre/DevOpsProject/blob/master/Images/mavencleanInstall.png)

**Steps to add a dependency and build the project**

 14. We add a dependency ***commons-fileupload*** in `pom.xml` file of the project as follows
      ![Initial State](https://github.com/nkatre/DevOpsProject/blob/master/Images/dependency1.png)
      ![Added Dependancy](https://github.com/nkatre/DevOpsProject/blob/master/Images/dependency2.png)
 15. Now, go to the `Dashboard` of Jenkins and `Build` the Project

**Output of build after adding `"commons-fileupload"` dependency**
![dependency output](https://github.com/nkatre/DevOpsProject/blob/master/Images/commons-fileupload-dependency.png)
![dependency output](https://github.com/nkatre/DevOpsProject/blob/master/Images/common-fileupload.png)

----------


III. Build Script Execution
--------------------

 1. To demonstrate a build script execution, we have added a shell script
   file named `sampleScript.sh` in the
   [WebGoat](https://github.com/nkatre/WebGoat) project directory.
 2. The contents of `sampleScript.sh` file is an echo command which outputs "**Hello WebGoat**" to the console screen when the project is built

**Steps to demonstrate build script execution**

 1. Goto Dashboard of Jenkins, select Project and click on `Configure`.
 2. Select `Build` > `Add a Build Step` > `Execute Shell`
 3. In the `command`, write "***sh sampleScript.sh***" which will execute the shell script sampleScript.sh
 ![build script](https://github.com/nkatre/DevOpsProject/blob/master/Images/buildScript.png)
 4. Goto [WebGoat](https://github.com/nkatre/WebGoat) project directory and add a file `sampleScript.sh` which echoes "**Hello WebGoat**" to the console screen on execution
	 ![sampleScript.sh](https://github.com/nkatre/DevOpsProject/blob/master/Images/Screenshot%20from%202015-02-09%2020:26:04.png)
 5.  Now goto Dashboard of Jenkins and build the project

**Output of build script execution**
![scriptExecution](https://github.com/nkatre/DevOpsProject/blob/master/Images/helloWebGoat.png)

----------


IV. Multiple Nodes
--------------------

 To demonstrate multiple nodes, we have created a slave node along with the master node
 
 The following steps are followed to set up a Slave node 
 1. Install [NodeLabel Parameter Plugin](https://wiki.jenkins-ci.org/display/JENKINS/NodeLabel+Parameter+Plugin) plugin in Jenkins
 2. Goto `Manage Jenkins` > `Manage nodes`> `New Node`
 3. Set the following configuration to the Slave Node as shown in the figure.
 ![Slave Configuration](https://github.com/nkatre/DevOpsProject/blob/master/Images/slaveConfiguration.png)
 4.  Now, goto Dashboard of Jenkins, select Project and click on `Configure`.
 5. Select on the option `This build is parameterized`
 6. Select both `master` and `slave` as default nodes
 7. Select option `Allow multi node selection for concurrent builds`  
 8. Select `Execute concurrent builds if necessary`
 9. All the above changes made to the Project Configuration is also shown in the below diagram
![ProjectConfigurationForSlaveNode](https://github.com/nkatre/DevOpsProject/blob/master/Images/slave1.png)
![ProjectConfigurationForSlaveNode](https://github.com/nkatre/DevOpsProject/blob/master/Images/slave2.png)
 10.  Now goto Dashboard of Jenkins and click on `Build Executor Status` option mentioned in the navigation box in the left
 11. Click on `Slave` and Run the slave node to make it active
	 ![masterslave](https://github.com/nkatre/DevOpsProject/blob/master/Images/masterslave.png)
 12. Finally goto Dashboard of Jenkins and build the project
 
 **Output of the multinode build**
 
 13.  After clicking build, we get the following screen. Select both the nodes and click build.
	 ![multinodebuild](https://github.com/nkatre/DevOpsProject/blob/master/Images/buildmultinode.png)
 14. We will notice that in the build history, we get two builds in progress where one is master and the other is slave.
	 ![twobuilds](https://github.com/nkatre/DevOpsProject/blob/master/Images/twoexecutions.png)
 15. In the console output, we can verify the two builds.
	This is the master build which ran successfully.
	![masterBuild](https://github.com/nkatre/DevOpsProject/blob/master/Images/masterBuild.png)
	This is the slave build which ran successfully.
	![slaveBuild](https://github.com/nkatre/DevOpsProject/blob/master/Images/slaveBuild.png)

 ----------


V. Status
--------------------
We can retrieve the status of any build if we know two parameters:<br>

 1. `IP Address` or `Computer Name`
 2. `Port Number` on which Jenkins is running

For Example:  The name of my computer is set as`nkatre-Inspiron-3521` and the port number on which Jenkins is running is `8080`<br>

Thus, the status of any build can be accessed by any machine in the network via the following URL:<br>
[http://nkatre-inspiron-3521:8080/job/WebGoat/19/](http://nkatre-inspiron-3521:8080/job/WebGoat/19/ "http://nkatre-inspiron-3521:8080/job/WebGoat/19/")<br>
The above URL will show the status of build #19<br>

**The following steps are followed to check status via http**

 1. Goto `Manage Jenkins` > `Configure System`
 2. In `Jenkins Location`, set the `Jenkins URL` as [http://computer-name:8080/](http://computer-name:8080/ "http://computer-name:8080/") <br>
	 For Example: In my case it is [http://nkatre-Inspiron-3521:8080/](http://nkatre-Inspiron-3521:8080/ "http://nkatre-Inspiron-3521:8080/")
 3. The below figure shows the settings
   ![statusSettings](https://github.com/nkatre/DevOpsProject/blob/master/Images/status1.png) 
 4. Now to check the status of any previous builds, enter the following URL in the web browser [http://nkatre-inspiron-3521:8080/job/WebGoat/19/](http://nkatre-inspiron-3521:8080/job/WebGoat/19/ "http://nkatre-inspiron-3521:8080/job/WebGoat/19/")
This will show the status of build #19
![StatusOfBuild#19ViaHTTP](https://github.com/nkatre/DevOpsProject/blob/master/Images/build%2319.png)
