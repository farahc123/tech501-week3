# Using Jenkins to build a CICD pipeline 

- [Using Jenkins to build a CICD pipeline](#using-jenkins-to-build-a-cicd-pipeline)
  - [Creating our first project in Jenkins](#creating-our-first-project-in-jenkins)
  - [Create a general diagram of CICD](#create-a-general-diagram-of-cicd)
  - [Thurs morning notes](#thurs-morning-notes)
  - [Jenkins CICD](#jenkins-cicd)
  - [3 jobs](#3-jobs)
  - [Setting up Jenkins and GitHub repo connection](#setting-up-jenkins-and-github-repo-connection)
    - [Setting up the SSH public key (i.e. padlock) on my GitHub repo for Jenkins](#setting-up-the-ssh-public-key-ie-padlock-on-my-github-repo-for-jenkins)
  - [Setting up Job 1 and the SSH private key on Jenkins](#setting-up-job-1-and-the-ssh-private-key-on-jenkins)
  - [Setting up webhook on Jenkins](#setting-up-webhook-on-jenkins)
    - [Setting up notification on GitHub for Jenkins webhook](#setting-up-notification-on-github-for-jenkins-webhook)
  - [Setting up Job 2 on Jenkins](#setting-up-job-2-on-jenkins)
  - [Setting up Job 3 on Jenkins](#setting-up-job-3-on-jenkins)
  - [Blockers for creating Job 3:](#blockers-for-creating-job-3)
  - [To do:](#to-do)
  - [Code store; probably not needed but keeping from here:](#code-store-probably-not-needed-but-keeping-from-here)


## Creating our first project in Jenkins

- Note that in Jenkins, creating a project/job is the equivalent of creating a pipeline
  1. **New item** from left sidebar
  2. **Name**: *farah-first-project*
  3. Choose **Freestyle project**
  4. **General** tab:
     - **Description**: *Testing Jenkins*
     - choose **Discard old builds**, keep on **Log Rotation**, put **5** in **Max # of builds to keep**
     - Near bottom of this tab, add **Execute shell** as a **Build step**:
       - for first pipeline, we entered `uname -a` for  testig purposes (this prints details about version of OS) 
  5. **Save**
  6. **Build now** from left sidebar -- this spins up/launches a VM (worker node) to run the job (visible on **Build Executor Status** menu towards bottom left of screen) -- note that these worker nodes are removed once they reach the threshold of **Idle** that the admin has set (not us yet)
![alt text](image-1.png)

- we know a job is successful because there will be a **green tick next to the build number
- shows detail of the command we ran:
![alt text](image-2.png)
![alt text](image-3.png)
- if need to edit a pipeline, choose Congifure from lefthand menu of the job

- multiple stage pipelines: we can link projects/jobs together to create a multi-stage pipeline
1. on first project job, go to configure
2. add post-build action at end of general tab > build other projects > choose farah-get-date > keep on **Trigger only if build is stable** to run only if first build step(s) are successful
3. Save
4. Build now on first project

![alt text](image-4.png)
- both jobs have two builds now, showing that the first job did run with a post-build action of running the second job ![alt text](image-5.png)

Detail on second job, showing its upstream project and console output
![alt text](image-6.png)
![alt text](image-7.png)

## Create a general diagram of CICD

![alt text](image.png)

## Thurs morning notes

- git is a distributed version control system
- `add .` -- stage which is saying what we want to go into next commit
- commit is saving a snapshot of those staged changes
- push is transferring the commit history to the branch of the remote repo
- devs need to do a git pull to merge changes from the next branch up (feature or main) before they can push their code, so their code is implemented into the latest version of the code that users are using
- if you run the CICD pipeline on the master node of your Jenkins server, it could impact the file system and mess everything up if something goes wrong; so you want to use worker/agent nodes that will be spun up to execute the jobs
- 3 jobs for agent nodes
- job 1:
  - unit test the code, which has been pushed on the developer branch
- job 2, if job 1 is successful:
  - merge code to main branch
- job 3, if job 2 successful (**CD**):
  - deploy code to where users can use it (in our case, to an EC2 instance/VM)

- for jobs 1 and 2, Jenkins needs credentials to read code in dev branch of github repo, as well as write access to merge the dev branch to the main branch of github repo 
  - this is SSH credentials to GitHub repo
- it will also need the private SSH key to our AWS/Azure SSH keypair to deploy the code to VMs for job 3

![alt text](image-16.png)


to fix my blocker:
- ensure the IP address used to visit the app page isn't HTTPS (as AWS sends me to this by default)

![alt text](images-jenkins/jenkins-cicd-pipeline-for-our-app.drawio.png)

## Jenkins CICD

- Refresher on Git:
  - Git is a distributed version control system
  - `add .` — stage which is saying what we want to go into next commit
  - commit is saving a snapshot of those staged changes
  - push is transferring the commit history to the branch of the remote repo
  - devs need to do a git pull to merge changes from the next branch up (feature or main) before they can push their code, so their code is implemented into the latest version of the code that users are using
- if you run the CICD pipeline on the master node of your Jenkins server, it could impact the file system and mess everything up if something goes wrong; so you want to use worker/agent nodes that will be spun up to execute the jobs

## 3 jobs
- there are 3 jobs for agent nodes in our pipeline
- **Job 1** (**CI job**):
  - unit test the code, which has been pushed on the developer branch
- **Job 2** (**CI job**), only if Job 1 is successful:
  - merge code to main branch
- **Job 3** (**CD job**), only if Job 2 is successful:
  - deploy code to where users can use it (in our case, to an EC2 instance/VM)

- for jobs 1 and 2, Jenkins needs credentials to read code in dev branch of github repo, as well as write access to merge the dev branch to the main branch of github repo 
  - this is SSH credentials to GitHub repo
- it will also need the private SSH key to our AWS/Azure SSH keypair to deploy the code to VMs for job 3

![alt text](images-jenkins/jenkins-cicd-pipeline-for-our-app.drawio.png)

## Setting up Jenkins and GitHub repo connection

  ### Setting up the SSH public key (i.e. padlock) on my GitHub repo for Jenkins 
1. In terminal, `ssh-keygen -t rsa -b 4096 -C "fcheded@gmail.com"` **name**: *farah-sparta-app-jenkins-to-github*
2. On **GitHub**, navigate to new repo (*tech501-sparta-app-cicd*):
   1. **Settings** > **Deploy keys** > **Add new key**
   2. **Name**: farah-sparta-app-jenkins-to-github (i.e. same as on local)
   3. In terminal, `cat farah-sparta-app-jenkins-to-github.pub` and paste contents of this **public** key into GitHub's **Key** field
   4. Enable **Allow write access** (this is for Job 2 to work)

## Setting up Job 1 and the SSH private key on Jenkins

1. On Jenkins, choose **New job** from the left sidebar
2. **Name**: *tech501-farah-job1-ci-test*
3. Choose **Freestyle project**
4. Enable **Discard old builds**
   1. Set *Max # of builds* to **5**
5. **Description**: *CI with GitHub webhook*
6. Enable **Github project**:
  - **URL**: https://github.com/farahc123/tech501-sparta-app-CICD/ — be sure to **remove *.git*** from end because Jenkins doesn't like it and **add / at end**
7. **Source code management tab**:
     1. Enable **GitHub**
        1. **Repository URL** (this needs to be in SSH format): git@github.com:farahc123/tech501-sparta-app-CICD.git
        2. **Add credentials** via Jenkins credentials provider:
          - **Kind**: ssh username with private key
          - **ID**: farah-sparta-app-jenkins-to-github
          - **Description**: to read and write to the repo
          - **Username**: farah-sparta-app-jenkins-to-github
          - **Private key**: enable **Enter directly** > **Add** > paste in **entire private key contents**, including the -----BEGIN OPENSSH PRIVATE KEY--- and footer part
          - **no passphrase** (because we never set it when generating this key-pair with `ssh-keygen`)
     2. Add the above and select it from **Credentials** dropdown
      - ![alt text](image-18.png)
     3. **Branch Specifier**: \**/main* (later we changed this to ***/dev***)
   ![alt text](image-19.png)
8. **Build Environment tab**:
     1. Enable **Provide Node & npm bin/ folder to PATH:**
        - Choose ***NodeJS version 20***
9. **Build Steps tab**:
    1.  Choose **Execute shell**
          - enter `cd nodejs20-sparta-test-app/app`
              `npm install`
              `npm test`
9. **Save**
10. **Build now** (may need to wait for it to run, and may fail if other people are running jobs on port 3000)
  - Example of successful build page:
    ![alt text](image-20.png)
  - Example of console output for successful build:
    ![alt text](image-22.png) 

## Setting up webhook on Jenkins

1. Enter the job (*tech501-farah-job1-ci-test*), choose **Configure** from left sidebar
2. In **Build Triggers tab**, enable **GitHub hook trigger for GITScm polling**
3. **Save**

### Setting up notification on GitHub for Jenkins webhook

1. Within the repo on GitHub, **Settings** > **Webhooks** > **Add webhook**
2. **Payload URL**: http://52.31.15.176:8080/github-webhook/ (made up of the URL of our Jenkins server + *github-webhook/*)
3. **Disable SSL verification**
4. Keep **Just the push event** enabled
5. **Add webhook**
 
## Setting up Job 2 on Jenkins

1. On Jenkins, choose **New job** from the left sidebar
2. **Name**: *tech501-farah-job2-ci-merge*
3. Choose **Freestyle project**
4. **Description**: *Merging changes from the dev branch of my GitHub repo to the main branch.*
5. Enable **Discard old builds**
   1. Set *Max # of builds* to **5**
6. Enable **Github project**:
  - **URL**: https://github.com/farahc123/tech501-sparta-app-CICD/ -- be sure to **remove *.git*** from end because Jenkins doesn't like it and **add / at end**
7. **Source code management tab**:
     1. Enable **GitHub**
        1. **Repository URL (this needs to be in SSH format)**: git@github.com:farahc123/tech501-sparta-app-CICD.git
        2. From **Credentials** dropdown, choose: farah-sparta-app-jenkins-to-github
      - ![alt text](image-18.png)
     2. **Branch Specifier**: \**/main* and **/dev*
8. **Build triggers tab**:
   1. Enable **Build after other projects are built**:
      - Under **Projects to watch**, choose *tech501-farah-job1-ci-test*
      - Choose **Trigger only if build stable**
9.  **Build Environment tab**:
     1. Enable **Provide Node & npm bin/ folder to PATH:**
        - Choose ***NodeJS version 20***
        - Enable **SSH agent** plugin and choose my credentials:
      ![alt text](image-24.png)
10. **Build Steps tab**:
      1.  Choose **Execute shell** and enter:
>`#!/bin/bash`
>
>`git checkout dev # switch to "dev" branch if not already`
>
>`git merge main # this optional command is being used to resolve any conflicts if i pushed any changes to my "main" but "dev" doesn't have that commit`
>
>`git checkout main # switch to "main"`
>
>`git merge --no-ff dev # merge the "dev" branch into "main"`
>
>`git push origin main # push the changes to the remote main branch`

**then will change this method to use a Jenkins plugin that allows me to merge rather than choosing *Execute Shell***

1.    **Save**
2.   Go back into Job 1's configuration, and **add Job 2 as a Post-Build Step**  
3.    Back on Job 2, **Build now** (may need to wait for it to run, and may fail if other people are running jobs on port 3000)
  - Example of successful build page:
    ![alt text](image-25.png)
  - Example of successful build console output:
   ![alt text](image-26.png)
  - Example of changes to dev branch automatically pushed to GitHub main branch:
  ![alt text](image-27.png)

- Note that I initially had a permissions error running Job 2, but this was because I hadn't enabled SSH Agent in the configuration    

**to do**

1. update above to reflect changes made to Job 2 configuration
2. test job 2 a couple more times (edit readme file on dev branch for this)

## Setting up Job 3 on Jenkins

1. job 3:
   - edit NSG rules on EC2 to allow Jenkins IP to SSH in (i.e.  make SSH source any)
   - the change we will be making is to the app front page (need to do this via dev so it passes, job 2 merges it to main, and then job 3 runs it on an instance)
     - go to sparta app CICD and dev branch
     - inside app folder, go into views folder, nano into index.ejs and make a change (e.g. after h2 "app is running correctly" add "this line shows a change has been made by the CICD pipeline at <time>")
   - need to rsync or scp code from jenkins previous jobs (don't git clone)
   - need to start EC2 instance and SSH into it
   - post link to changed front page in chat
   - wait for thumbs up, go back and make another change, push them, add new link for proof 2


2. On AWS, start App EC2 instance
3. On Jenkins, choose **New job** from the left sidebar
   1. **Name**: *tech501-farah-job3-cd-deploy*
   2. Choose **Freestyle project**
   3. **Description**: *Putting our GitHub changes into action on our app*
   4. Enable **Discard old builds**
   5. Set *Max # of builds* to **5**
   6. Enable **Github project**:
  - **URL**: https://github.com/farahc123/tech501-sparta-app-CICD/ -- be sure to **remove *.git*** from end because Jenkins doesn't like it and **add / at end**
   7. **Source code management tab**:
     1. Enable **GitHub**
        1. **Repository URL (this needs to be in SSH format)**: git@github.com:farahc123/tech501-sparta-app-CICD.git
        2. From **Credentials** dropdown, choose: farah-sparta-app-jenkins-to-github
      - ![alt text](image-18.png)
       3. **Branch Specifier**: **/main*
   8. **Build triggers tab**:
      1. Enable **Build after other projects are built**:
         - Under **Projects to watch**, choose *tech501-farah-job2-cd-merge*
         - Choose **Trigger only if build stable**
   9.  **Build Environment tab**:
         1.  Enable **Provide Node & npm bin/ folder to PATH:**
        2. Choose ***NodeJS version 20***
          3.  Enable **SSH agent** plugin and add my AWS credentials:
      ![alt text](image-31.png)
   10. **Build Steps tab**:
       1.  Choose **Execute shell** and run:
  >`scp -o StrictHostKeyChecking=no -r /var/jenkins/workspace/farah-job2-ci-merge/nodejs20-sparta-test-app/app/ ubuntu@ec2-34-255-121-219.eu-west-1.compute.amazonaws.com:/home/ubuntu`
  - SCPs the folder resulting from my job 2 (i.e. in its workspace) to the EC2 instance
  - note the use of the `-o` option to allow turning off of Strict Host Key Checking (i.e. bypass the `yes` input required to SSH into a machine for the first time)
> `ssh ubuntu@ec2-34-255-121-219.eu-west-1.compute.amazonaws.com <<'EOF'`
>     
>     cd /home/ubuntu/nodejs20-sparta-test-app/app
> 
>     npm install
>
>     npm start
>   - doesn't need SSH key specifying because I'm using the SSH agent configured above
>   - note the `<<'EOF'`, which allows code to be indented below and performed in the machine SSHed into 

## Blockers for creating Job 3:

- Note: the following are the commands I initially used, but they both failed when it comes to Host key verification on the EC2 app instance:
  - rsync command for job 3:
`rsync -avz -e "ssh" nodejs20-sparta-test-app/app/ ubuntu@ip-172-31-49-60:/tech501-sparta-app/app`
    - failure message:
![alt text](image-29.png)

  - scp command for job 3:

`scp -r nodejs20-sparta-test-app/app/ ubuntu@ip-172-31-49-60:/tech501-sparta-app/app`
  - failure message:
![alt text](image-30.png)

  - I removed my SCP command to try SSHing into my EC2 instance to ensure this command is okay and it is:
![alt text](image-33.png)


- I then used this SCP and SSH command and it worked:
- `scp -o StrictHostKeyChecking=no -r /var/jenkins/workspace/farah-job2-ci-merge/nodejs20-sparta-test-app/app/ ubuntu@ec2-34-255-121-219.eu-west-1.compute.amazonaws.com:/home/ubuntu`

`ssh ubuntu@ec2-34-255-121-219.eu-west-1.compute.amazonaws.com`
![alt text](image-34.png)

- My Job 2 wasn't working correctly so I manually started Job 3 (after Job 1 and 2) and the job didn't complete as below:
  ![alt text](image-35.png)
  ![alt text](image-36.png)

- But the app page successfully updated nonetheless:
  ![alt text](image-37.png)

- I fixed Job 2, but now Job 3 isn't completing due to the port being in use
![alt text](image-39.png)

- However the app page has now updated (note timestamp of latest index.ejs edit)
![alt text](image-38.png)


## To do:

edit documentation for Job 2 since I have changed these steps now 

---

## Code store; probably not needed but keeping from here:

sudo scp -o StrictHostKeyChecking=no -r -i ~/.ssh/tech501-farah-aws-key.pem nodejs20-sparta-test-app/app/ ubuntu@ip-172-31-49-60:/tech501-sparta-app/app

```ssh -i "tech501-farah-aws-key.pem" -o StrictHostKeyChecking=no ubuntu@ec2-34-255-121-219.eu-west-1.compute.amazonaws.com

cd /
cd tech501-sparta-app/app
npm stop
npm start```

note that this SSH command works on its own:

`ssh -o StrictHostKeyChecking=no ubuntu@ec2-34-255-121-219.eu-west-1.compute.amazonaws.com`

but not when I scp



sudo scp -o StrictHostKeyChecking=no -r nodejs20-sparta-test-app/app/ ubuntu@ip-172-31-49-60:/tech501-sparta-app/app

ssh -o StrictHostKeyChecking=no ubuntu@ec2-34-255-121-219.eu-west-1.compute.amazonaws.com

cd /
cd tech501-sparta-app/app
npm stop
npm start