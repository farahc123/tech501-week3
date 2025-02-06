# Using Jenkins to build a CICD pipeline 

- [Using Jenkins to build a CICD pipeline](#using-jenkins-to-build-a-cicd-pipeline)
  - [Jenkins CICD](#jenkins-cicd)
  - [3 jobs](#3-jobs)
  - [Setting up Jenkins and GitHub repo connection](#setting-up-jenkins-and-github-repo-connection)
    - [Setting up the SSH public key (i.e. padlock) on my GitHub repo for Jenkins](#setting-up-the-ssh-public-key-ie-padlock-on-my-github-repo-for-jenkins)
  - [Setting up Job 1 and the SSH private key on Jenkins](#setting-up-job-1-and-the-ssh-private-key-on-jenkins)
  - [Setting up webhook on Jenkins](#setting-up-webhook-on-jenkins)
    - [Setting up notification on GitHub for Jenkins webhook](#setting-up-notification-on-github-for-jenkins-webhook)
  - [Setting up Job 2 on Jenkins](#setting-up-job-2-on-jenkins)
  - [Setting up Job 3 on Jenkins](#setting-up-job-3-on-jenkins)



## Jenkins CICD

- git is a distributed version control system
- `add .` -- stage which is saying what we want to go into next commit
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
  - **URL**: https://github.com/farahc123/tech501-sparta-app-CICD/ -- be sure to **remove *.git*** from end because Jenkins doesn't like it and **add / at end**
7. **Source code management tab**:
     1. Enable **GitHub**
        1. **Repository URL** (this needs to be in SSH format): git@github.com:farahc123/tech501-sparta-app-CICD.git
        2. **Add credentials** via Jenkins credentials provider:
          - **Kind**: ssh username with private key
          - **ID**: farah-sparta-app-jenkins-to-github
          - **Description**: to read and write to the repo
          - **Username**: farah-sparta-app-jenkins-to-github
          - **Private key**: enable **Enter directl**y > **Add** > paste in **entire private key contents**, including the -----BEGIN OPENSSH PRIVATE KEY--- and footer part
          - **no passphrase** (because we never set it when generating this key-pair with `ssh-keygen`)
     2. Add the above and select it from **Credentials** dropdown
      - ![alt text](image-18.png)
     3. **Branch Specifier**: \**/main* (later we changed this to ***/dev***)
   ![alt text](image-19.png)
8. **Build Environment tab**:
     1. Enable **Provide Node & npm bin/ folder to PATH:**
        - Choose ***NodeJS version 20***
     2. **Build Steps tab**:
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

- BLOCKER FACED (double-checked GitHub has write access for this SSH key and it does)
![alt text](image-21.png)

**to do after blocker is resolved**:

1. test job 2 a couple of times (edit readme file for this)

## Setting up Job 3 on Jenkins

2. job 3:
   - edit NSG rules on EC2 to allow Jenkins IP to SSH in (i.e.  make source any)
   - need to rsync or scp code from jenkins previous jobs (don't git clone)
   - need to start EC2 instance and SSH into it
   - 