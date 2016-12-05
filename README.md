$ ansible-playbook devenv-launch.yml
=========================

## Ansible playbook for provisioning OpenShift All-In-One Using AWS
(for testing/development)

### Differences from master branch:
* This branch launches with `devenv-centos7_*` AMI
* This branch syncs local origin to /data/src/github.com/openshift/origin in ec2
* Does not install charlie rpm, since it's already there.
 
This is an example Ansible playbook that:

1. Launches an EC2 instance, by default with 'centos7' AMI, or you can configure to run with 'rhel7' 
2. Starts an OpenShift All-In-One cluster in the EC2
3. Deploys a sample cakephp-mysql app in project 'myproject'
4. Can be configured to run any post-cluster plays you need, just add any plays to 'devenv-launch.yml'
5. Can be run with '-e LOCAL_BUILD=True' see `Notes-Local Build` below for more info.
   - syncs your local origin code to the ec2, builds if necessary and restarts services with locally built binaries
6. When running with '-e LOCAL_BUILD=True', can also pass '-e SYNC=True', to ensure local branch     
   is synced with that run.  Otherwise, local branch will only be synced during original run of 
   '-e LOCAL_BUILD=True'

###Initial Preparation Local Machine:
*  Clone this repository:      
   `openshift-ansible` is a submodule in this repository, hence the --recursive 
```
$ git clone --recursive git@github.com:sallyom/origin-allinone-dev
```
*  `$ sudo dnf install ansible`
*  `$ sudo dnf copr enable abutcher/ansible`  <--you'll want Ansible 2.2
*  `$ sudo dnf update ansible`
*  `$ sudo pip install boto six`
*  You need this file:
```
$cat ~/.aws/credentials
    [default]
    aws_access_key_id = XXXX
    aws_secret_access_key = XXXXX
```

Then run this:
### $ ansible-playbook devenv-launch.yml

All tasks are defined in roles/<task>/tasks/main.yml.

Your ec2 instance will be named: '$USER-allinone-devenv' in the AWS console.

## Now What?

* You can run oc commands locally OR you can ssh into your ec2 and run oc commands there
  (you are logged in as 'system:admin' in your ec2 instance).       
  You can login as 'admin/password' on your local machine.      
  `$ oc login -u admin -p password <public_dns_of _ec2>:8443`

* To access the web console:
```
https://<ec2----.compute-1.amazonaws.com>:8443
 
login using 'anypassword' to gain admin access to all projects.

login with 'admin/password'
```

* Running the playbook a second time will skip tasks that have been done and are unchanged, so 
  a new ec2 instance will not be started every time the playbook is run.
* To terminate and start again, 
    * In AWS console, terminate the instance.
    * Now you are ready to run `$ ansible-playbook devenvup.yml` again!

This playbook defaults to launching a 'centos7' AMI.  If you prefer 'rhel7' and have credentials and
a pool id for subscription-manager, then fill in vars/launch_vars.yml, vars/user_vars.yml  and use 
`$ ansible-playbook -e OS=rhel7 devenv-launch.yml`

### Notes-Local Build:

* `$ ansible-playbook -e LOCAL_BUILD=True devenv-launch.yml` will restart origin services      
   with locally build origin binaries and will set the PATH appropriately for openshift|oc|oadm

* playbook will sync local origin repository, expected at `../origin` from 'origin-allinone-devenv'      
  or passed as `-e ORIGIN_PATH=/your/local/path/to/origin`.  Origin repo will be at /data/src/github.com/openshift/origin in ec2.     
  If your local machine is linux it is recommended to build the binaries locally **before** running     
  the playbook.  This is much more efficient.  I repeat, run `make` from your `local origin repo`     
  before running `ansible-playbook -e LOCAL_BUILD=True devenv-launch.yml`. 

* If /data/src/github.com/openshift/origin/marker.file is not in the ec2, then the local origin repo will be synced,      
  golang will be installed and playbook will run `make` from `/data/src/github.com/openshift/origin`.  This takes     
  a few minutes and lots of memory.

* On subsequent runs of the playbook, if you run with '-e LOCAL_BUILD=True -e SYNC=True', then     
  the local origin branch will be re-synced.  Otherwise, it will only be synced the first run.
* Currently, this branch will install origin v1.4 rpm.  Therefor,      
  to use LOCAL_BUILD=True, you should checkout the version 1.4.x from your local origin repository.    
  (master branch works)               
