# CDK Workshop
The Container Development Kit and Deploying Apps to OpenShift

## Setting Up the Software Components - OS X
Open a terminal and unzip the vagrant plugins
```bash
$ cd
$ unzip ~/Downloads/cdk*.zip
$ cd ~/cdk/plugins
$ ls -l *.gem
```
Next install the plugins
```bash
$ vagrant plugin install ./vagrant-registration-1.2.1.gem  \
   ./vagrant-service-manager-1.0.1.gem  \
   ./vagrant-sshfs-1.1.0.gem
$ vagrant plugin list
```
Add the Vagrant box
```bash
$ cd
$ vagrant box add --name cdkv2 ~/Downloads/rhel-cdk-kubernetes-7.2*.x86_64.vagrant-virtualbox.box
$ vagrant box list
```
## Start the CDK - OS X
```bash
$ cd ~/cdk/components/rhel/rhel-ose/
$ vagrant up
```
**Note:** You will be asked if you wish to register the CDK.  Type in your Red Hat Developer Network username and password.
## Setting Up the Software Components - Windows
Start Cygwin

Set Up the Environment
```bash
$ cat “export VAGRANT_DETECTED_OS=cygwin” >> ~/.bashrc
$ cat “export VAGRANT_HOME=/opt” >> ~/.bashrc
$ source ~/.bashrc
```
Unzip the and install plugins
```bash
$ cd /opt
$ unzip /cygdrive/c/Users/Kenneth\ D\ Evensen/Downloads/cdk*.zip
$ cd cdk/plugins
$ vagrant plugin install ./vagrant-registration-1.2.1.gem  \
   ./vagrant-service-manager-1.0.1.gem  \
   ./vagrant-sshfs-1.1.0.gem
```
Add the Vagrant box
```bash
vagrant box add --name cdkv2 \
  rhel-cdk-kubernetes-7.2*.x86_64.vagrant-virtualbox.box
```
## Start the CDK - Windows
```bash
$ cd /opt/cdk/components/rhel/rhel-ose/
$ vagrant up
```
**Note:** You will be asked if you wish to register the CDK.  Type in your Red Hat Developer Network username and password.
## Loggin in to the Vagrant Box aka. the CDK
```bash
$ cd cdk/components/rhel/rhel-ose
$ vagrant ssh
[vagrant@rhel-cdk ~]$
```
## Log into OpenShift via the CLI
```bash
[vagrant@rhel-cdk ~]$ oc login
Authentication required for https://127.0.0.1:8443 (openshift)
Username: admin
Password:
Login successful.

...

Using project "sample-project".
[vagrant@rhel-cdk ~]$
```
**USERNAME:** admin <br/>
**PASSWORD:** admin
## View the Security Context Constraints
```bash
[vagrant@rhel-cdk ~]$ oc get scc
```
## Adjust the Security Context Constraints
```bash
[vagrant@rhel-cdk ~]$ oc edit scc hostmount-anyuid
apiVersion: v1
defaultAddCapabilities: null
fsGroup:
  type: RunAsAny
groups:
- system:authenticated
kind: SecurityContextConstraints
….
```
## Create Directories to Support GitLab
```bash
[vagrant@rhel-cdk ~]$ mkdir -p gitlabce/{data,config,logs}
[vagrant@rhel-cdk ~]$ chmod o+w gitlabce/*
[vagrant@rhel-cdk ~]$ chcon -Rt svirt_sandbox_file_t gitlabce/
```
## Clone GitLab Template and Change into the Project Directory
```bash
[vagrant@rhel-cdk ~]$ git clone https://github.com/kevensen/openshift-development-infrastructure.git

[vagrant@rhel-cdk ~]$ cd openshift-development-infrastructure/gitlab

[vagrant@rhel-cdk gitlab]$
```
## Log into OpenShift via the CLI
```bash
[vagrant@rhel-cdk gitlab]$ oc login
Authentication required for https://127.0.0.1:8443 (openshift)
Username: openshift-dev
Password:
Login successful.

Using project "sample-project".
[vagrant@rhel-cdk gitlab]$
```
**USERNAME:** openshift-dev <br />
**PASSWORD:** Developer
## Create a New OpenShift Project
```bash
[vagrant@rhel-cdk gitlab]$ oc new-project gitlab
Now using project "gitlab" on server "https://127.0.0.1:8443".

You can add applications to this project with the 'new-app' command. For example, try:

    $ oc new-app centos/ruby-22-centos7~https://github.com/openshift/ruby-hello-world.git

to build a new hello-world application in Ruby.
[vagrant@rhel-cdk gitlab]$
```
## Add the Project Template to OpenShift
```bash
[vagrant@rhel-cdk gitlab]$ oc create -f gitlab-cdk-template.yaml
template "gitlab" created
[vagrant@rhel-cdk gitlab]$
```
## Follow the logs
```bash
[vagrant@rhel-cdk gitlab]$ oc get pods
NAME             READY     STATUS    RESTARTS   AGE
gitlab-1-9l11t   1/1       Running   0          5m
[vagrant@rhel-cdk gitlab]$ oc logs -f gitlab-1-9l11t
```
## Change GitLab's External URL (1/3)
```bash
[vagrant@rhel-cdk gitlab]$ cd ~/gitlabce/config/
[vagrant@rhel-cdk config]$
```
## Change GitLabs' External URL (2/3)
```bash
[vagrant@rhel-cdk gitlab]$ cd ~/gitlabce/config/
[vagrant@rhel-cdk config]$ ls -al
……
[vagrant@rhel-cdk config]$ sudo vi gitlab.rb
….
external_url 'http://gitlab-http-gitlab.rhel-cdk.10.1.2.2.xip.io'
….
```
## Change GitLabs' External URL (3/3)
```bash
[vagrant@rhel-cdk config]$ oc get pods
NAME             READY     STATUS    RESTARTS   AGE
gitlab-1-b4jto   1/1       Running   0          18m
[vagrant@rhel-cdk config]$ oc exec -it gitlab-1-b4jto /bin/bash

root@gitlab-1-b4jto:/# gitlab-ctl reconfigure
….
Chef Client finished, 10/273 resources updated in 13 seconds
gitlab Reconfigured!
root@gitlab-1-b4jto:/# exit
[vagrant@rhel-cdk config]$
```
## Edit the Deployment Config
```bash
[vagrant@rhel-cdk ~]$ oc project cakedemo
Now using project "cakedemo" on server "https://127.0.0.1:8443".
[vagrant@rhel-cdk ~]$ oc get bc
NAME              TYPE      FROM      LATEST
cakephp-example   Source    Git       1
[vagrant@rhel-cdk ~]$ oc edit bc cakephp-example
```
## Add a Generic Web Hook
```bash
triggers:
  - imageChange:
      lastTriggeredImageID: registry.access.redhat.com/rhscl/php-56-rhel7:latest
    type: ImageChange
  - type: ConfigChange
  - github:
      secret: 5jFwDsstlCDVimDOFOJfoC4Nmsj5rkMYHjLgD3VW
    type: GitHub
  - type: Generic
    generic:
      secret: secret101
```
