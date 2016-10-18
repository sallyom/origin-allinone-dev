$ ansible-playbook devenv-launch.yml
=========================

## Ansible playbook for provisioning OpenShift All-In-One Using AWS
(for testing/development)

This is an example Ansible playbook that:

1. Launches an EC2 instance, by default with 'centos7' AMI, or you can configure to run with 'rhel7' 
2. Starts an OpenShift All-In-One cluster in the EC2
3. Deploys a sample cakephp-mysql app in project 'myproject'
4. Can be configured to run any post-cluster plays you need, just add any plays to 'devenv-launch.yml'
5. Can be run with '-e LOCAL_BUILD=True'
   - syncs your local origin code to the ec2, builds if necessary and restarts services with locally built binaries

###Initial Preparation Local Machine:
*  Clone this repository:      
   `openshift-ansible` is a submodule in this repository, hence the --recursive 
```
$ git clone --recursive git@github.com:sallyom/example-Ansible-OpenShift-all-in-one.git
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
* I've provided a publicly available `centos7` AMI in `vars/launch_vars.yml`, but you can change to any centos7 or rhel7 AMI.
* You need to fill in `vars/launch_vars.yml`:
  * This setup assumes you have an AWS account and a VPC that enables public DNS hostnames.
    See the **VPC Dashboard, Start VPC Wizard** link in the AWS console for more info.
  * More info: https://aws.amazon.com/premiumsupport/knowledge-center/ec2-linux-ssh-troubleshooting/

Then run this:
### $ ansible-playbook devenv-launch.yml

All tasks are defined in roles/<task>/tasks/main.yml.

Your ec2 instance will be named: '$USER-allinone-devenv' in the AWS console.

## Now What?:

* You can run oc commands locally OR you can ssh into your ec2 and run oc commands there
  (you are logged in as 'system:admin' in your ec2 instance).  You can login as 'admin/password' on your local machine.
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


