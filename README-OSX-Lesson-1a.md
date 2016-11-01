# CDK Workshop - Install on OS X - Lesson 1a
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
