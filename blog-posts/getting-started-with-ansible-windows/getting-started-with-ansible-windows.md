---
published: true
title: 'Getting started with Ansible and configuring Windows hosts'
cover_image: ''
description: 'Getting started configuring Windows hosts with Ansible'
tags: ansible, windows, devops
series:
canonical_url: 'https://worldwildweb.dev/getting-started-with-ansible-and-configuring-windows-machines/'
---

Ansible is a configuration management, provisioning and deployment tool which is quickly gaining popularity in the DevOps areas. Managing and working on various platforms including Microsoft Windows.  
What makes Ansible stand out of other configuration management tools is that it’s agentless. Which means no software is required on the target host. Ansible uses SSH for communication with Unix based hosts and WinRM for Windows hosts.  
Recent announcement from Microsoft’s team is an upcoming fork of OpenSSH for Windows, which would make things ever smoother for DevOps teams managing Windows infrastructure.

In this post we will get started with Ansible by:

1.  Setup of the control machine
2.  Configure Windows server in order to receive commands from Ansible
3.  Install Chocolatey and SQL Server

_Ansible requires PowerShell version 3.0 and .NET Framework 4.0 or newer to function on older operating systems like Server 2008 and Windows 7._

If you covered the requirements, let’s get started with the first step.

## Setup Ansible control machine

As previously mentioned Ansible is agentless, but we need control machine — machine which talks to all of our hosts.

### Ansible can’t run on Windows but there’s a trick

Currently Ansible can only be installed on Unix based machines, but if you are using Windows as your primary OS, you can install Ubuntu subsystem. Read [this](https://docs.microsoft.com/en-us/windows/wsl/install-win10) for further installation details. If you are non Windows user please continue reading.

## Install Ansible

After the installation of Ubuntu subsystem on Windows (if you had so), lets proceed with the installation of Ansible by opening terminal.

Install Ubuntu repository management:
`$ sudo apt-get install software-properties-common`

Lets update our system:
`$ sudo apt-get update`

Add Ansible repository:
`$ sudo apt-add-repository ppa:ansible/ansible`

Then install Ansible:
`$ apt-get install ansible`

Add Python package manager:
`$ apt install python-pip`

Add Python WinRM client:
`$ pip install pywinrm`

Install XML parser:
`$ pip install xmltodict`

If every thing went OK you should be able to get the current version:
`$ ansible --version`

So far, so good. Lets continue with configuration of the tool.

## Configure Ansible

### Inventory — list of the hosts

**Inventory.yml** is the main configuration file of your hosts addresses separated in groups with descriptive names.

Let’s create that file and set the example below:
`$ vim inventory.yml`

Enter the IP/DNS addresses for your group:

```
[dbservers]
mydbserver1.dns.example 80.69.0.160

[webservers]
mywebserver1.dns.example 80.69.0.162
```

## Configure the connection

We are a few steps away from establish connection to the remote servers. Let’s configure the connection itself — credentials, ports, type of connection. The convention is to name the config file based on your group of hosts.

_If you want all of your inventory to use that same configuration file you can name it_ **_all.yml_**_. We will use_ **_all.yml_** _as all servers will have same credentials and connection type._

Let’s begin by creating folder:
`$ mkdir group_vars`

Create the file and edit it:
`$ vim group_vars/all.yml`

Add the configuration details:

```
ansible_user: ansible_user
ansible_password: your_password_here
ansible_port: 5985
ansible_connection: winrm
ansible_winrm_transport: basic
ansible_winrm_operation_timeout_sec: 60
ansible_winrm_read_timeout_sec: 70
```

This credentials will be used to access the remote hosts with connection set to WinRM basic authentication. We will create them in the next section.
_We use basic authentication but for your production environment you probably want to use more secure schema. See_ [_this_](https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html?highlight=windows) _article for more info._

## Configure Windows hosts

Our Windows hosts need to be configured before execute any commands on it. The following PowerShell script will do:

1.  Create the Ansible user we defined in **all.yml**
2.  Add the user to the Administrators group
3.  Set WinRM authentication to basic and allow unencrypted connections
4.  Add Firewall rule for WinRM with your control machine IP address

Open PowerShell on the host and execute the script:

```
NET USER ansible_user "your_password_here" /ADD
NET LOCALGROUP "Administrators" "ansible_user" /ADD
Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true
Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true
netsh advfirewall firewall add rule name="WinRM" dir=in action=allow protocol=TCP localport=5985 remoteip=10.10.1.2
```

After the execution is completed we can try to ping our host from the control machine to check that connection is OK. We ping only the DB servers:
`$ ansible dbservers -i inventory.yml -m win_ping`

## Write our first playbook

Getting back to our Ansible control machine to add a playbook — set of tasks or plays which together form the playbook.

The target is to install Chocolatey which is the community driven package manager for Windows. After that we will install SQL Server and reboot the server.

Ansible come with many modules for Windows with a lot of functionalities out of the box. They are prefixed with “win\_” like for example win_feature. You can check more [here](https://docs.ansible.com/ansible/latest/modules/list_of_windows_modules.html?highlight=windows) for your specific needs.

Let’s continue with the creation of the playbook file:
`$ vim configure-win-server-playbook.yml`

In the file describe the playbook as follows:

```
---
- hosts: dbservers
  tasks:
   - name: Install Chocolatey
     raw: Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

   - name: Install SQL Server
     win_chocolatey:
     name: sql-server-2017
     state: present

   - name: Reboot to apply changes
     win_reboot:
      reboot_timeout: 3600
```

Execute the playbook by typing:
`$ ansible-playbook dbservers -i inventory.yml configure-win-server-playbook.yml`

You will see each task running and returning status of execution and after reboot we are all ready!

## Conclusion

Ansible is really powerful tool. Microsoft and the community is doing really fantastic work for porting Ansible modules to Windows which are written in PowerShell. Yet the plan to have SSH feature on Windows is great too. No matter if your inventory is of physical or virtual servers, you should definitely try out Ansible on your infrastructure for saving time, money and of course avoid human mistakes by manually configure, deploy or provision those environments.
