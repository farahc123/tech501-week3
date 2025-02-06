# CI/CD

- [CI/CD](#cicd)
  - [Note on Week 3 tasks](#note-on-week-3-tasks)
  - [Software development lifecycle (SDLC)](#software-development-lifecycle-sdlc)
    - [Non-DevOps SDLC:](#non-devops-sdlc)
    - [DevOps SDLC:](#devops-sdlc)
  - [CICD pipelines](#cicd-pipelines)
    - [Why build a pipeline?](#why-build-a-pipeline)
  - [CI and its benefits](#ci-and-its-benefits)
  - [CD and its benefits](#cd-and-its-benefits)
  - [The difference between CD and CDE](#the-difference-between-cd-and-cde)
  - [Jenkins](#jenkins)
    - [Why use Jenkins?](#why-use-jenkins)
    - [Disadvantages of Jenkins](#disadvantages-of-jenkins)
    - [Stages of Jenkins](#stages-of-jenkins)
    - [Alternatives to Jenkins](#alternatives-to-jenkins)
  - [Creating our first project in Jenkins](#creating-our-first-project-in-jenkins)
  - [Create a general diagram of CICD](#create-a-general-diagram-of-cicd)
  - [Thurs morning notes](#thurs-morning-notes)

## Note on Week 3 tasks

 - We will be using **AWS** and **Jenkins** for these tasks
 - Jenkins Server 1 address: http://34.254.6.118:8080
 - **Jenkins Server 2 address:**  http://52.31.15.176:8080 **(my assigned server)**

## Software development lifecycle (SDLC)

- **High-level SDLC workflow**:
  1. **Plan**
  2. **Design** (i.e. code)
  3. **Develop**
  4. **Deploy**

### Non-DevOps SDLC:

![alt text](images-CICD/image.png)

### DevOps SDLC:

![alt text](images-CICD/image2.png)

- Remember that the goal of DevOps is to shorten the development lifecycle while maintaining quality and security
- Because the goal is to get things into production quicker, **automation** is used in all stages beyond planning and designing
- **Iteration** is also a key principle, so changes to the **main branch**'s code are **small and frequent**

## CICD pipelines

- **a CICD pipeline**: an automated process of steps that must be performed in order to get a new version of software into production (i.e. deployed)
- a **build**: every instance of the pipeline running
- CICD happens **after development** (see diagram above)
- the **trigger** for a CICD pipeline is usually (and preferably) a **push of code**; for our CICD pipeline, it's specifically a **Git push** on our developer branch (this will merge into our main branch)
   -  Jenkins needs to listen for a notification that there has been a change to code, which is via a **webhook** (one service telling another service that something has happened -- need to set up the sender and receiver ends of a webhook for it to work)
   -  during testing, it's okay to trigger these **manually** instead e.g. through **Build now** button

- as a junior DevOps engineer, we will be getting familiar with infrastructure already in place rather than creating things for a new app
  - in a couple of years, we might be given more responsibility to create architecture/infrastructure/CICD pipeline for a new feature
- CICD goes hand in hand with DevOps practice of **small iterative changes**

-  CI:
     1. build and test
     2. if it passes the test, then it's safe to merge/integrate from developers' branch with main branch
-  CD: 
     1. deploy code (where to depends on the arhcitecture, e.g. a server, VM)
     2. 
- when deploying to a VM, it's called virtualised deployment; if on kubernetes, it's containerised deployment
- the CD element has to be set up correctly such that, if multiple pushes have been made, only one ends up being deployed
  - push from Feature Branches --> Dev --> Main
  - In more complex scenarios you would also either have a more complex pipeline or multiple pipelines

### Why build a pipeline?

- to automate the (build + test), integration & deployment parts of the software development lifecycle
  - allows business to quickly get changes to the end user/customer 
  - makes life easier for devs
  - reduce risk that big code changes will mess things up (because CICD goes hand in hand with DevOps practice of **frequent small** code changes)
  - ultimately this all creates business value: save time, save money; lower risk; users are actually using the latest code quicker

## CI and its benefits

- **Continuous integration**: a software development practice that automates the frequent (e.g. multiple times a day) integration of (naturally small) code changes by multiple developers into one central repository as well as automates its testing
- **Steps**:
  1. Integration involves frequent, small commits to version control systems like Git
  2. a CI service would then immediately build and run unit tests on the code changes to identify errors straight away and give developers feedback
- CI addresses the issues of big code change merges being time-consuming, difficult, and buggy, ultimately meaning users had to wait longer to get the software 
- CI involves the automation of the integration process as well as cultural shift to small, frequent integrations
- **Benefits**:
  - finds and addresses bugs quicker/earlier
  - this ultimately improves software quality
  - and reduces the time it takes to get software out
  - less of a big shift for users because changes happen incrementally

## CD and its benefits

- **Continuous delivery**: a software development practice in which code changes integrated via CI are deployed to a tasting environment for automated testing
- in CD, the tests go beyond the tests run in CI stage -- examples here:
  - UI testing
  - load testing
  - API reliability testing
- **Benefits**:
  - developers can uncover issues quicker
  - delivery is therefore quicker
  - 

## The difference between CD and CDE

- **Continuous delivery**: 

- rather than deployed, might mean the code needs to be delivered to a testing environment
- often, it's about having it all tested and ready to be deployed when approved/given the green light; not necessarily that it will be deployed to production (i.e. end users) straight away
- in contrast,deployment usually means deployed straight to production and end users  
- whether an org uses CDeployment or CDelivery depends on its culture -- i.e. if CDeployment, its users often expect to get updates frequently, and expect to get bugs that they will then routinely quickly feedback for quick resolution

## Jenkins

- **Jenkins**: open-source automation server commonly used by DevOps teams for CICD pipelines
- it manages and controls...

### Why use Jenkins?

- free to use,
- powerful plugin architecture with lots of plugins so plugins can be added as and when needed
- great to understand how CICD works, so you can use any other CICD tool after

### Disadvantages of Jenkins

- plugins are third-party, which means quality can vary
- plugins can be abandonware, i.e. they are no longer updated and support not provided which can be 

### Stages of Jenkins

### Alternatives to Jenkins

- GitLab (paid)
- Spinnaker (free)
- CircleCI (paid)

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