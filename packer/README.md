# aws-ami-builder-packer

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

This project is built to create an AMI that is pre-configured to host a PHP application in AWS EC2 instance.


### Built With

The project is built using,
* AWS resources
* Packer
* Ansible


<!-- GETTING STARTED -->
## Getting Started

### Prerequisites

* Create a free tier AWS account.
* Create an IAM user with programmable access and make a note of the access and secret keys.

### Installation

1. Clone the repo
   ```
   git clone https://github.com/nikhil15041993/AWS.git
   ```
   move to packer directory.
2. [Install Packer](https://www.packer.io/docs/install)
3. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)



<!-- USAGE EXAMPLES -->
## Usage

### Creating AMI

The AMI is created using packer.

1. Set the environment vaiables: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
2. Update the default values of base_ami (amazon linux), subnet_id (with internet access) and security_group_id (allow port 22 from local ip) in the  variables.pkr.hcl to match the ones in your AWS account.
3. cd into the packer folder in the cloned repository.
4. Run the following commands in order
    - packer init .
    - packer fmt .
    - packer validate .
    - packer build .

This will provision the AMI in you AWS account.

Now, you can use the AMI to launch an EC2 instance which will have the web application pre configured and ready to use.



## Failed to connect to the host via SSH on Ubuntu 22.04 ???

In my case I was trying to build an AWS EC2 image via packer and the ansible provisioner, and I had this error:
```
amazon-ebs.aws: Failed to connect to the host via ssh: Unable to negotiate with 127.0.0.1 port
amazon-ebs.aws: 40015: no matching host key type found. Their offer: ssh-rsa
```

The proposed solution is to add this snippet to either your /etc/ssh/ssh_config or ~/.ssh/config:
```
PubkeyAcceptedKeyTypes +ssh-rsa
```
or just for some specific hosts:
```
Host host.example.com
    PubkeyAcceptedKeyTypes +ssh-rsa
```
In the case of ansible connecting to a host, or packer launching ansible connecting to a host, this needs an additional step or two.

For ansible:
```
ansible --ssh-extra-args="-o PubkeyAcceptedKeyTypes=+ssh-rsa"
```
For packer with ansible provisioning:
```
build {
  sources = ["sources.amazon-ebs.aws"]
  provisioner "ansible" {
    ansible_env_vars = [
      ...
      "ANSIBLE_SSH_ARGS='-o PubkeyAcceptedKeyTypes=+ssh-rsa -o HostkeyAlgorithms=+ssh-rsa'"
    ]
    playbook_file       = "..."
    galaxy_file         = "..."
    ...
    extra_arguments     = "${concat(local.default_ansible_extra_args, var.ansible_extra_args)}"
  }
}

```



##  packer output ami id to terraform variables automatically

First, create a file called ami.tf.template:
```
# "ami.tf" was automatically generated from the template "ami.tf.template".
variable "ami" {
  default     = "${AMI_GENERATED_BY_PACKER}"
  description = "The latest AMI."
}
```
This template will be used to create the ami.tf file, which makes the AMI from packer available to your existing Terraform setup.

Second, create a shell wrapper script for running packer. You can use the following ideas:

```
# run packer (prints to stdout, but stores the output in a variable)
packer_out=$(packer build packer.json | tee /dev/tty)

# packer prints the id of the generated AMI in its last line
ami=$(echo "$packer_out" | tail -c 30 | perl -n -e'/: (ami-.+)$/ && print $1')

# create the 'ami.tf' file from the template:
export AMI_GENERATED_BY_PACKER="$ami" && envsubst < ami.tf.template > ami.tf
```

Once the script is done, it has created an ami.tf file, which may look like this:
```
# "ami.tf" was automatically generated from the template "ami.tf.template".
variable "ami" {
  default     = "ami-aa92a441"
  description = "The latest AMI."
}
```
Finally, put that file next to your existing Terraform setup. Then you can then access the AMI like this:
```
resource "aws_launch_configuration" "foo" {
  image_id = "${var.ami}"
  ...
}
```
