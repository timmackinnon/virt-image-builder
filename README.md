# Introduction
This repository contains instructions and code to configure a __Packer__ based image builder machine which can be used to build __qemu__ images for __KVM__ or __Xen__ virtual machines or __Vagrant__ box files for __VirtualBox__.

This code is geared towards Linux and the RedHat family of operating systems - CentOS, Oracle Linux or RedHat Enterprise Linux (e.g. a system using the yum package manager). The code uses __Ansible__ to configure the image builder machine.

Once configured, this image builder machine can then be used to build virtualization images.

# Does your machine support virtualization?
First things first ... we need to check if your system supports virtualization:
```
$ egrep "(svm|vmx)" /proc/cpuinfo
```

If your system supports virtualization, you will get either __vmx__ (Intel-VT technology) or __svm__ (AMD-V support) in the output.

# Setting up Ansible
__Ansible__ can be installed with your favourite package manager or you can install it in a Python Virtual Environment using the __requirements.txt__ file in this repository.

## Set up Ansible in a Python virtualenv (requires Python3)
If Python3 is the default Python version on the machine where this code will run:
```
virtualenv venv
venv/bin/pip install -r requirements.txt
```

If Python3 is not the default Python version on the machine where this code will run:
```
virtualenv --python=<path to Python3> venv
venv/bin/pip install -r requirements.txt
```

Check that __ansible__ has been installed correctly:
```
venv/bin/ansible --version
```

# Configuring your hosts file
Uncomment the host entry and modify it to reflect the IP address or hostname of the target host. For example:
```
[builders]
#builder0 ansible_host=a.b.c.d
```
could be changed to:
```
[builders]
builder0 ansible_host=1.1.1.1
```

Modify the __ansible_ssh_user__ to reflect the user that will be used to connect to the target host. For example:
```
[builders:vars]
ansible_ssh_user=FIXME
```
could be changed to:
```
[builders:vars]
ansible_ssh_user=builder
```

`NOTE` - __ansible_ssh_user__ is assumed to be a privileged user that can install packages, etc. - either __root__ or a user with passwordless __sudo__ to root.

Modify the __ansible_ssh_private_key_file__ entry, if not using your default RSA key, to point to the key that will be used to connect to the target host.

# Run the setup.yml playbook
```
venv/bin/ansible-playbook setup.yml
```

This playbook will perform the following set of granular operations:

* install the __@Base__ group
* install the __@Core__ group
* install the __@Platform Development__ group
* install the __@Additional Development__ group
* install the __@Virtualization Hypervisor__ group
* install the __@^Server with GUI__ environment
* install a set of miscellaneous packages outlined in the __group_vars/all.yml__ file
  - included in this miscellaneous set of packages is __screen__ and __tigervnc__
* install __VirtualBox__
  - version defined in __group_vars/all.yml__ file
* install __Vagrant__
  - version defined in __group_vars/all.yml__ file
* install __Packer__ and set up __packer__ in the __PATH__ of the __ansible_ssh_user__ via the __~/.bash_profile__ file
  - version defined in __group_vars/all.yml__ file
