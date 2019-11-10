---
published: false
title: 'Configure your dev Windows machine with Ansible'
cover_image: ''
description: 'How to use the power of Ansible for configuring your own Windows environment'
tags: ansible, ansibleplaybook, windows
series:
canonical_url: 'https://worldwildweb.dev/configure-your-dev-windows-machine-with-ansible/'
---

Ansible is well known it the IT operations fields with its fantastic automation abilities.
You can do whatever you want with Windows too if it's a Powershell, bat script or one of the more than one hundred? modules.
I will use it to configure my personal machine and save the hustle every time I step on new one.
It's not a big deal to install a few programs but I'm sure this will repay in the long term. Can be pretty useful for configuring multiple machines too.
Using Ansible to target localhost on Linux is like click-click-go, but It's different when it's comes to Windows.
We need to install WSL on Windows, Ansible on WSL, Enable WinRm on Windows and finally control it from WSL.
And all this happens on your localhost.

# Install Ansible on WSL

Enable WSL:
`Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`

Install Ubuntu distribution but you may choose to install whatever distro you want:
`Invoke-WebRequest -Uri https://aka.ms/wsl-ubuntu-1604 -OutFile Ubuntu.appx -UseBasicParsing`

Star your WSL and update packages:
`sudo apt-get update`

Install Ansible:

```
sudo apt install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```

Edit your ansible hosts file at `/etc/ansible/hosts` and add `[local] 127.0.0.1`
which hostname we will target later.

# Enable WinRM

By default WinRM works only for Private or Domain networks. You can skip that by providing parameter to `Enable-PSRemoting -SkipNetworkProfileCheck` but I don't suggest doing that. Instead make your trusted network private.
Then enable WinRM: `Enable-PSRemoting` running it in Powershell.
Enable Basic Auth: `Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true`
Enable Unencrypted connection: `Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true`

# Run the playbook

Using ansible pull run the playbook which install Chocolatey and a few packages from it. Full details can found in the [repo](https://github.com/gmarokov/ansible-win-postinstall). Provide your username and password for Windows.

`ansible-pull –U https://github.com/gmarokov/ansible-win-postinstall.git -e ansible-user=your_win_user ansible_password=your_win_user_password`

# Conclusion

I'm sure you as developer make a lot of tweaks to your os, me too. Having a few more tweaks to add, would be great to share some ideas and extend it even further.
