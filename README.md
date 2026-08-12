Jenkins + Ansible + Google Cloud — Student Teaching Project

A beginner-friendly DevOps project that teaches students how GitHub, Jenkins, Ansible, SSH, Linux, Nginx, and Google Cloud fit together in a simple automation workflow.

Learning goal: By the end of this project, a student should understand how source code can trigger an automation pipeline that configures a cloud server and deploys a simple website.

1. What We Are Building

We are building a small CI/CD-style automation pipeline:

Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   v
Ansible
   |
   | SSH
   v
Google Cloud VM
   |
   v
Nginx
   |
   v
Website

What each tool does

Tool

What it does

GitHub

Stores our project and automation code

Jenkins

Runs the automation pipeline

Jenkinsfile

Tells Jenkins what steps to execute

Ansible

Configures the server and deploys the website

SSH

Provides remote access to the server

Google Cloud

Provides the virtual machine

Nginx

Serves the website

The key lesson is:

Jenkins orchestrates. Ansible configures. Google Cloud provides the infrastructure.

2. Learning Objectives

After completing this project, students should be able to:

Explain what CI/CD means

Explain the role of Jenkins in a pipeline

Explain what Ansible is used for

Create an Ansible inventory

Write a basic Ansible playbook

Use SSH keys for automation

Install and manage Nginx with Ansible

Deploy a web page automatically

Connect Jenkins to GitHub

Create and use a Jenkinsfile

Run a Jenkins build

Read Jenkins console output

Troubleshoot SSH authentication errors

Understand basic Linux service management

Understand why infrastructure sizing matters

3. Prerequisites

Students should have basic familiarity with:

Linux terminal

Git

GitHub

SSH

Basic YAML

Basic HTML

You will need:

A Google Cloud account

A Linux VM

A GitHub account

Jenkins

Ansible

Java

Git

Important: This project is designed as a learning/demo environment. Do not use the example credentials, IP addresses, or SSH keys in a production environment.

4. Project Architecture

The project uses one small Google Cloud VM for the learning environment.

The VM contains the automation tools and acts as the target web server.

A simplified architecture is:

                    +----------------+
                    |    GitHub      |
                    | Project Code   |
                    +-------+--------+
                            |
                            v
                    +----------------+
                    |    Jenkins     |
                    |   Pipeline     |
                    +-------+--------+
                            |
                            v
                    +----------------+
                    |    Ansible     |
                    |  Playbook      |
                    +-------+--------+
                            |
                          SSH
                            |
                            v
                    +----------------+
                    | Google Cloud   |
                    |      VM        |
                    +-------+--------+
                            |
                            v
                         Nginx
                            |
                            v
                       Web Page

5. Repository Structure

The project contains the core automation files:

ansible-gcp-project/
│
├── inventory
├── playbook.yml
├── Jenkinsfile
└── README.md

inventory

Defines the server Ansible will manage.

playbook.yml

Defines the configuration tasks Ansible should perform.

Jenkinsfile

Defines the pipeline Jenkins should execute.

README.md

Documents the project and teaches the concepts.

6. Ansible Inventory

Example:

[web]
YOUR_SERVER_IP ansible_user=root ansible_ssh_private_key_file=/root/.ssh/ansible_key

Replace YOUR_SERVER_IP with the public IP of your Google Cloud VM.

The important pieces are:

[web]

This creates an Ansible host group called web.

ansible_user=root

This tells Ansible which Linux user to use.

ansible_ssh_private_key_file=/root/.ssh/ansible_key

This tells Ansible which private SSH key to use.

7. Testing Ansible Connectivity

Before asking Ansible to change anything, test the connection:

ansible -i inventory web -m ping

A successful result looks similar to:

SUCCESS => {
    "changed": false,
    "ping": "pong"
}

What does this prove?

It proves:

Ansible can find the server.

Ansible can authenticate using SSH.

The SSH connection works.

Ansible can communicate with the target.

This is one of the first important checkpoints in the project.

8. The Ansible Playbook

Our playbook is intentionally simple.

---
- name: Configure web server
  hosts: web
  become: yes

  tasks:

    - name: Ensure Nginx is installed
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Deploy test website
      copy:
        content: |
          <html>
          <head>
              <title>Jenkins Ansible GCP</title>
          </head>
          <body>
              <h1>Hello from Jenkins + Ansible!</h1>
              <p>Running on Google Cloud.</p>
          </body>
          </html>
        dest: /var/www/html/index.html

9. Understanding the Playbook

hosts: web

hosts: web

Tells Ansible to run the tasks against the hosts in the web inventory group.

become: yes

become: yes

Allows Ansible to execute privileged operations.

Installing packages and writing to /var/www/html normally requires elevated privileges.

Installing Nginx

apt:
  name: nginx
  state: present

Ansible checks whether Nginx is installed.

If it is missing, Ansible installs it.

If it is already installed, Ansible does not needlessly reinstall it.

This is part of Ansible's idempotent approach.

Starting Nginx

service:
  name: nginx
  state: started
  enabled: yes

This ensures:

Nginx is running.

Nginx starts automatically after reboot.

Deploying the website

The copy module creates:

/var/www/html/index.html

Nginx then serves this file as the website.

10. Running Ansible Manually

Before Jenkins is introduced, students should understand what Ansible is doing.

Run:

ansible-playbook -i inventory playbook.yml

Ansible connects to the VM and performs the tasks.

The result should show tasks such as:

TASK [Ensure Nginx is installed]
TASK [Ensure Nginx is running]
TASK [Deploy test website]

This establishes the basic automation before introducing Jenkins.

11. Why Jenkins?

Without Jenkins, a developer would have to manually run:

ansible-playbook -i inventory playbook.yml

Jenkins gives us an automation server that can execute the process for us.

The simplified workflow becomes:

Code change
     |
     v
GitHub
     |
     v
Jenkins
     |
     v
Ansible
     |
     v
Server

This is the beginning of a CI/CD pipeline.

12. Jenkinsfile

The repository contains a Jenkinsfile.

The Jenkinsfile is the pipeline definition stored alongside the project code.

A basic pipeline can look like:

pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Ansible Deployment') {
            steps {
                sh 'ansible-playbook -i inventory playbook.yml'
            }
        }
    }
}

The exact Jenkinsfile used in the project may vary depending on how Jenkins and credentials are configured.

13. What Happens During a Jenkins Build?

When the Jenkins job runs:

1. Jenkins starts the build
        |
2. Jenkins checks out the Git repository
        |
3. Jenkins reads the Jenkinsfile
        |
4. Jenkins executes the pipeline
        |
5. Ansible runs
        |
6. Ansible connects to the target server
        |
7. Nginx is installed/configured
        |
8. Website is deployed

The important distinction is:

Jenkins is not replacing Ansible.

Jenkins is triggering and orchestrating Ansible.

14. What Does "Build" Mean?

A Jenkins build is an execution of the pipeline.

For example:

Build #1
Build #2
Build #3
Build #4

Each build represents another execution of the automation.

A successful build means Jenkins completed the defined pipeline without an error.

In this project we successfully reached:

Last successful build: #4

15. Real-World Troubleshooting

A major part of DevOps is not just writing the configuration.

It is diagnosing when things don't work.

During this project we encountered:

Permission denied (publickey)

This happened because the SSH authentication configuration between the automation environment and the target server was not initially correct.

We investigated:

SSH keys

authorized_keys

SSH configuration

Ansible inventory

SSH users

private key permissions

direct SSH connectivity

This is a realistic DevOps troubleshooting scenario.

16. GitHub SSH vs Ansible SSH

Two separate SSH relationships existed.

GitHub

Used for:

Server → GitHub

with:

/root/.ssh/github_key

Ansible

Used for:

Ansible → Target VM

with:

/root/.ssh/ansible_key

Keeping these keys separate is a good security and administration practice.

17. Jenkins Resource Lesson

The learning environment used a very small VM:

CPU: 2
RAM: ~1 GB
Swap: initially 0

Jenkins is a Java application and requires considerably more resources than a simple Linux web server.

Jenkins repeatedly took too long to initialize and eventually hit the systemd startup timeout.

We diagnosed the problem using:

free -h
nproc
systemctl status jenkins
journalctl -u jenkins

We then added a 2 GB swap file.

The important lesson is:

Infrastructure size matters.

A small VM can be useful for learning, but a more serious Jenkins environment should use a larger machine.

18. What We Achieved

By the end of this project, we had demonstrated:

Google Cloud VM provisioning

Linux server administration

SSH authentication

Git and GitHub

Ansible installation

Ansible inventory

Ansible playbook development

Nginx installation

Nginx service management

Automated website deployment

Jenkins installation

Jenkins job creation

Git/Jenkins integration

Jenkinsfile usage

Jenkins builds

CI/CD fundamentals

Troubleshooting SSH

Troubleshooting Jenkins

Basic server resource monitoring

19. What This Project Does NOT Cover Yet

This is intentionally a foundation project.

It does not yet implement:

Terraform

Docker

Kubernetes

HTTPS/SSL

Load balancing

Production secrets management

Automated unit/integration testing

Blue/green deployment

Canary deployment

Monitoring and alerting

High availability

Jenkins agents

Production-grade infrastructure

These can become future projects.

20. Student Exercises

Students can extend this project by completing the following challenges.

Beginner

Change the website title.

Change the HTML heading.

Add an image.

Add a CSS file.

Add another Ansible task.

Install curl using Ansible.

Intermediate

Create a separate Nginx configuration.

Create an Ansible variable for the website title.

Use an Ansible template.

Add a deployment verification task.

Add a Jenkins test stage.

Trigger Jenkins automatically after a GitHub push.

Advanced

Add Terraform.

Use Terraform to create the GCP VM.

Use Ansible after Terraform completes.

Add Docker.

Add HTTPS.

Add a load balancer.

Add monitoring.

Create separate development and production environments.

21. Next Project

The natural next step is:

Terraform + Jenkins + Ansible + Google Cloud

The architecture would become:

GitHub
   |
   v
Jenkins
   |
   +------------------+
   |                  |
   v                  v
Terraform           Ansible
   |                  |
   v                  v
Google Cloud       Server Config
Infrastructure         |
   |                   |
   +---------+---------+
             |
             v
        Application

Terraform will manage infrastructure.

Ansible will manage server configuration.

Jenkins will orchestrate the workflow.

This will be a more complete DevOps project.

22. Key Lesson

The most important concept from this project is not memorizing commands.

It is understanding the responsibilities of each component:

GitHub
"What code are we using?"

Jenkins
"When and how should the automation run?"

Ansible
"How should the server be configured?"

Google Cloud
"Where does the infrastructure run?"

Nginx
"How is the website served?"

Once those responsibilities are clear, the individual commands become much easier to understand.
