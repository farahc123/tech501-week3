# Using Jenkins to build a CICD pipeline 

- [Using Jenkins to build a CICD pipeline](#using-jenkins-to-build-a-cicd-pipeline)
  - [Refresher on Git](#refresher-on-git)
  - [Jenkins notes](#jenkins-notes)
  - [Creating our first test project in Jenkins: *farah-first-project*](#creating-our-first-test-project-in-jenkins-farah-first-project)
  - [Multi-stage pipelines](#multi-stage-pipelines)
    - [Second test job: *farah-get-date*](#second-test-job-farah-get-date)
  - [Our Jenkins CICD pipeline containing 3 jobs](#our-jenkins-cicd-pipeline-containing-3-jobs)
  - [Setting up Jenkins and GitHub repo connection for CICD pipeline](#setting-up-jenkins-and-github-repo-connection-for-cicd-pipeline)
    - [Setting up the SSH public key (i.e. padlock) on my GitHub repo for Jenkins](#setting-up-the-ssh-public-key-ie-padlock-on-my-github-repo-for-jenkins)
  - [Setting up Job 1 and the SSH private key on Jenkins](#setting-up-job-1-and-the-ssh-private-key-on-jenkins)
    - [Setting up notification on GitHub for Jenkins webhook](#setting-up-notification-on-github-for-jenkins-webhook)
  - [Setting up Job 2 on Jenkins](#setting-up-job-2-on-jenkins)
  - [Setting up Job 3 on Jenkins](#setting-up-job-3-on-jenkins)
  - [Steps for Job 3](#steps-for-job-3)
    - [Explanation of this code](#explanation-of-this-code)
  - [Blockers for creating Job 3:](#blockers-for-creating-job-3)


## Refresher on Git
  - Git is a distributed version control system
  - `add` stages files, which is saying what we want to go into next commit
  - `commit` is saving a snapshot of those staged changes
  - `push` is transferring the commit history to the branch of the remote repo (i.e. GitHub for us)
  - note that developers need to do a `git pull` (i.e. `fetch` and `merge`) to merge changes from the next branch up (feature or main) before they can push their code, so that their code is implemented into the latest version of the code that users are already using (i.e. the code on the *main* branch)

## Jenkins notes

- we know a Jenkins job is successful because there will be a **green tick** next to the build number
- if we need to edit a pipeline, choose **Configure** from lefthand menu of the job
- **Console output** on the specific build page shows the detail of the commands we have run
- Jenkins has a master node and agent nodes on which it runs the jobs
- if you run the CICD pipeline on the master node of your Jenkins server, it could impact the file system and mess everything up if something goes wrong; so you want to use worker/agent nodes that will be spun up to execute the jobs

## Creating our first test project in Jenkins: *farah-first-project*

>  1. **New item** from left sidebar
>  2. **Name**: *farah-first-project*
>  3. Choose **Freestyle project**
>  4. **General** tab:
>     - **Description**: *Testing Jenkins*
>     - choose **Discard old builds**, keep on **Log Rotation**, put **5** in **Max # of builds to keep**
>     - Near bottom of this tab, add **Execute shell** as a **Build step**:
>       - for first pipeline, we entered `uname -a` for  testig purposes (this prints details about version of OS) 
>  5. **Save**
>  6. **Build now** from left sidebar — this spins up/launches a VM (worker node) to run the job (visible on **Build Executor Status** menu towards bottom left of screen) — note that these worker nodes are removed once they reach the threshold of **Idle** that the admin has set (not us yet)
![alt text](image-1.png)

- green tick next to the build number shows the job was successful
- **Console output** shows the detail of the command we ran:
![alt text](image-2.png)
![alt text](image-3.png)

## Multi-stage pipelines

- We can link projects/jobs together to create a multi-stage pipeline, in which jobs are chained together
1. On the first job, go to **Configure**
2. In the Post-Build Actions tab, add **Build other projects** from the dropdown list and choose the project to build on from it; also keep **Trigger only if build is stable** enabled so that the subsequent job runs only if the first job has run successfully
3. Save

### Second test job: *farah-get-date*

- To test chaining the jobs, I created a simple job called *farah-get-date* that simply runs `date` as an *Execute shell* build step
- I then configured it as a post-build step in the configuration for Job 1 (*farah-first-project*)
- both jobs have two builds now, showing that the first job did run with a post-build action of running the second job ![alt text](image-5.png)
- Job 2's its upstream project and console output, to illustrate that it has been chained from Job 1
![alt text](image-6.png)
![alt text](image-7.png)

## Our Jenkins CICD pipeline containing 3 jobs

![general diagram of our CICD pipeline](image.png)
![jobs diagram for our jenkins CICD pipeline](images-jenkins/jenkins-cicd-pipeline-for-our-app.drawio.png)
- there are 3 jobs for agent nodes in our CICD pipeline
- **Job 1** (**CI job**):
  - uses a webhook to listen for new pushes to the dev branch of my GitHub and unit tests this code
- **Job 2** (**CI job**), only if Job 1 is successful:
  - merges code tested in Job 1 to main branch
- **Job 3** (**CD job**), only if Job 2 is successful:
  - deploys Job 2 code to where users can use it (in our case, to an EC2 instance/VM)

- for jobs 1 and 2, Jenkins needs credentials to read code in dev branch of github repo, as well as write access to merge the dev branch to the main branch of github repo 
  - this is SSH credentials to GitHub repo
- it will also need the private SSH key to our AWS/Azure SSH keypair to deploy the code to VMs for job 3

## Setting up Jenkins and GitHub repo connection for CICD pipeline

- Steps:
  1. Set up SSH public key on GitHub repo
  2. Add SSH private key to Jenkins during Job 1 setup

  ### Setting up the SSH public key (i.e. padlock) on my GitHub repo for Jenkins 
1. In terminal, `ssh-keygen -t rsa -b 4096 -C "fcheded@gmail.com"`
   1. **name**: *farah-sparta-app-jenkins-to-github*
2. On **GitHub**, navigate to new repo (*tech501-sparta-app-cicd*):
   1. **Settings** > **Deploy keys** > **Add new key**
   2. **Name**: farah-sparta-app-jenkins-to-github (i.e. same as on local)
   3. In terminal, `cat farah-sparta-app-jenkins-to-github.pub` and paste contents of this **public** key into GitHub's **Key** field
   4. Enable **Allow write access** (this is for Job 2 to work)

## Setting up Job 1 and the SSH private key on Jenkins

1. On Jenkins, choose **New job** from the left sidebar
2. **Name**: *tech501-farah-job1-ci-test*
3. Choose **Freestyle project**
4. **Description**: *CI with GitHub webhook*
5. Enable **Discard old builds**
   1. Set *Max # of builds* to **5**
6. Enable **Github project**:
  - **URL**: https://github.com/farahc123/tech501-sparta-app-CICD/ — be sure to **remove *.git*** from end because Jenkins doesn't like it and **add / at end**
7. **Source code management tab**:
     1. Enable **Git**
        1. **Repository URL** (this needs to be in **SSH format**): git@github.com:farahc123/tech501-sparta-app-CICD.git
        2. **Add credentials** via Jenkins credentials provider:
          - **Kind**: SSH username with private key
          - **ID**: farah-sparta-app-jenkins-to-github
          - **Description**: to read and write to the repo
          - **Username**: farah-sparta-app-jenkins-to-github
          - **Private key**: enable **Enter directly** > **Add** > paste in **entire private key contents**, including the -----BEGIN OPENSSH PRIVATE KEY--- and footer part
          - **no passphrase** (because we never set it when generating this key-pair with `ssh-keygen`)
     2. Add the above and select it from **Credentials** dropdown
      - ![alt text](image-18.png)
     3. Under **Branches to build**, set **Branch Specifier**: *\*/dev*
   ![alt text](image-19.png)
8. **Build Triggers** tab:
   1. Enable **GitHub hook trigger for GITScm polling**
9.  **Build Environment tab**:
     1. Enable **Provide Node & npm bin/ folder to PATH:**
        - Choose ***NodeJS version 20***
      2. Enable SSH Agent and choose GitHub credentials added above 
10. **Build Steps tab**:
    1.  Choose **Execute shell**
          - enter `cd nodejs20-sparta-test-app/app`
              `npm install`
              `npm test`
11. **Save**
12. **Build now** (may need to wait for it to run, and may fail if other people are running jobs on port 3000)
  - Example of successful build page:
    ![alt text](image-20.png)
  - Example of console output for successful build:
    ![alt text](image-22.png) 

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
  - **URL**: https://github.com/farahc123/tech501-sparta-app-CICD/ — be sure to **remove *.git*** from end because Jenkins doesn't like it and **add / at end**
7. **Source code management tab**:
     1. Enable **Git**
        1. **Repository URL (this needs to be in SSH format)**: git@github.com:farahc123/tech501-sparta-app-CICD.git
        2. From **Credentials** dropdown, choose: farah-sparta-app-jenkins-to-github
      - ![alt text](image-18.png)
     2. **Branch Specifier**: *\*/dev*
     3. **Repository browser**: leave on **(Auto)**
     4. Add **Additional behaviours**:
        1. **Name of repository**: ***origin***
        2. **Branch to merge to**: ***main***
        3. **Merge strategy**: leave on **default**
        4. **Fast-forward mode**: leave on -**-ff**
        5. ![alt text](image-40.png)
8. **Build triggers tab**:
   1. Enable **Build after other projects are built**:
      - Under **Projects to watch**, choose *tech501-farah-job1-ci-test*
      - Choose **Trigger only if build stable** (we are essentially recreating the post-build step from Job 1 here; may be redundant but doing it as a back-up)
9.  **Build Environment tab**:
     1. Enable **Provide Node & npm bin/ folder to PATH:**
        - Choose ***NodeJS version 20***
        - Enable **SSH agent** plugin and choose my GitHub credentials:
      ![alt text](image-24.png)
10. In **Post-build actions**, choose **Git Publisher** from dropdown
   1.  Enable **Push Only If Build Succeeds**
   2.  Branches:
        - **Branch to push**: main
        - **Target remote name**: origin
    3. (Once Job 3 created) Add another action, choosing **Build other projects**: *farah-job3-cd-deploy* and enable **Trigger only if build is stable**
11.   **Save**
12.   Test manually with **Build now** (may need to wait for it to run, and may fail if other people are running jobs on port 3000)
  - Example of successful build page:
    ![alt text](image-25.png)
  - Example of successful build console output:
   ![alt text](image-26.png)
  - Example of changes to *dev* branch automatically pushed to GitHub *main* branch:
  ![alt text](image-27.png)

- Note that I initially had a permissions error running Job 2, but this was because I hadn't enabled **SSH Agent** in the configuration

## Setting up Job 3 on Jenkins

- Before testing Job 3, we need to:
  1. Start the app EC2 instance
  2. ensure NSG rules on app EC2 instance allow Jenkins IP to SSH in (i.e.  make SSH source any)
  3. navigate to `~/OneDrive - Sparta Global/Documents/Github/tech501-sparta-app-CICD/nodejs20-sparta-test-app/app/views tech501-sparta-app-CICD` and ensure I'm in the *dev* branch with `git checkout dev`
  4. from here, `nano` into `index.ejs` and make a change (e.g. after the h2 "app is running correctly" section, add "this line shows a change has been made by the CICD pipeline at \<time>")
 - **Job 3 needs to then**:
   1. either `rsync` or `scp` the app code from Jenkins's previous jobs (don't git clone) — note that `rsync` might be preferable because it only updates the changed files, making this step shorter
   2. SSH into the app EC2 instance
   3. navigate to the newly downloaded app folder
   4. export the DB_HOST connection string
   5. install `npm`
   6. install `pm2` (if not already installed on app VM)
   7. `pm2 restart app.js`
- 
- So Job 3 will ultimately be testing whether:
  1.  changes made to the app's front page (on the *dev* branch) successfully passes test (via Job 1)
  2.  merges to main branch (via Job 2)
  3.  and then runs on our specified instance (Job 3)

## Steps for Job 3

1. On AWS, start App EC2 instance
2. On Jenkins, choose **New job** from the left sidebar
   1. **Name**: *tech501-farah-job3-cd-deploy*
   2. Choose **Freestyle project**
   3. **Description**: *Putting our GitHub changes into action on our app*
   4. Enable **Discard old builds**
      1. Set *Max # of builds* to **5**
   5. Enable **Github project**:
  - **URL**: https://github.com/farahc123/tech501-sparta-app-CICD/ — be sure to **remove *.git*** from end because Jenkins doesn't like it and **add / at end**
   1. **Source code management tab**:
     1. Enable **Git**
        1. **Repository URL (this needs to be in SSH format)**: git@github.com:farahc123/tech501-sparta-app-CICD.git
        2. From **Credentials** dropdown, choose: farah-sparta-app-jenkins-to-github
      - ![alt text](image-18.png)
       1. **Branch Specifier**: **/main*
   2. **Build triggers tab**:
      1. Enable **Build after other projects are built**:
         - Under **Projects to watch**, choose *tech501-farah-job2-cd-merge*
         - Choose **Trigger only if build stable**
   3.  **Build Environment tab**:
         1.  Enable **Provide Node & npm bin/ folder to PATH:**
        1. Choose ***NodeJS version 20***
          1.  Enable **SSH agent** plugin and add my AWS credentials following same steps in Job 1:
      ![alt text](image-31.png)
   4.  **Build Steps tab**:
       1.  Choose **Execute shell** and run:
   `scp -o StrictHostKeyChecking=no -r /var/jenkins/workspace/farah-job2-ci-merge/nodejs20-sparta-test-app/app/ ubuntu@ec2-34-252-96-146.eu-west-1.compute.amazonaws.com:/home/ubuntu`

`ssh ubuntu@ec2-34-252-96-146.eu-west-1.compute.amazonaws.com << 'EOF'`
  
  #indent this block below the SSH line above
  `cd /home/ubuntu/app/`

  `export DB_HOST=mongodb://172.31.59.92:27017/posts`

  `npm install`

  `sudo npm install pm2 -g`

  `pm2 restart app.js`

 #unindent

 `EOF`

 ### Explanation of this code
 
  >`scp -o StrictHostKeyChecking=no -r /var/jenkins/workspace/farah-job2-ci-merge/nodejs20-sparta-test-app/app/ ubuntu@ec2-34-255-121-219.eu-west-1.compute.amazonaws.com:/home/ubuntu`
>  - SCPs the folder resulting from my job 2 (i.e. in its Jenkins workspace) to the EC2 instance
>  - note the use of the `-o` option to allow turning off of Strict Host Key Checking (i.e. bypass the `yes` input required to SSH into a machine for the first time)
>   - the SSH command doesn't need SSH key specifying because I'm using the SSH agent configured above
>   - note that the downside of using `pm restart app.js` is that it stops the app running if it is already running, which means users get disrupted (can be for a long time depending on the size of the app) — this is therefore okay for testing but not for production
>   - note the `<<'EOF'`, which allows code to be indented below and performed in the machine SSHed into 
- **Canary testing** or **blue-green deployment** can help make the experience seamless for users by reducing downtime of the app when it's restarted
- The rest of the code then indented below the `ssh` command to signify that it is to be performed in the shell of that machine, not Jenkins' shell

## Blockers for creating Job 3:

- Note: the following are the commands I initially used, but they both failed when it comes to Host key verification on the EC2 app instance:
  - `rsync` command for job 3:
`rsync -avz -e "ssh" nodejs20-sparta-test-app/app/ ubuntu@ip-172-31-49-60:/tech501-sparta-app/app`
    - failure message:
![alt text](image-29.png)

  - `scp` command for job 3:

`scp -r nodejs20-sparta-test-app/app/ ubuntu@ip-172-31-49-60:/tech501-sparta-app/app`
  - failure message:
![alt text](image-30.png)

  - I removed my `scp` command to try SSHing into my EC2 instance to ensure this command is okay and it is:
![alt text](image-33.png)

- I then used the code in Job 3 steps above and it all worked