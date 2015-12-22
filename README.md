# Ubuntu secure server setup using Ansible

## Overview
Contains a few Ansible playbooks to quickly set up a new Ubuntu server or VPS with security in mind. This will let you:

- Change the root password
- Add a new user with key-based authentication and sudo privileges
- Secure the server

## Prerequisites
Make sure your host has an up-to-date Ansible installed, and that your server has Python installed. Clone this repository locally with:

```
git clone https://github.com/alexsydell/ansible-ubuntu-server-setup.git
```

## Hosts
Normally Ansible expects a hosts file, but these playbooks are mainly intended to set up one host one time. Instead of creating a hosts file, you can pass in the host via `-i "[IP address or hostname],"` (note the comma at the end). The rest of the guide will use this method.

## Changing the root password
If your host gave you the root password when the server was provisioned, you may want to change it. Assuming you have root access at this point, you can do the following:

```
ansible-playbook change_root_pw.yml -i "YOUR_HOST," -k -u root
```

## Adding the sudo user
It's generally considered a bad idea to allow root login or password login via SSH. This step creates a new user that uses key-based authentication (you'll have to have your public key generated) and can `sudo`. You'll be prompted for the username and password. If you choose to use a username other than the default of `deploy`, make sure you pass it in as a variable for the next step (see below).

Note that this step doesn't actually prevent root or password-based login -- that'll happen in the next, once you can log in as this new user. Run the following:

```
ansible-playbook add_sudo_user.yml -i "YOUR_HOST," -k -u root
```

## Securing the server
This step will:

- Install `ufw` and configure it to block everything except SSH and ports 80 and 443
- Install and enable `fail2ban` to prevent many repeated SSH login attempts
- Install and configure unattended upgrates
- Limit `su` access to the `sudo` group, which your user will not be in by default
- Secure shared memory by removing the permission to execute programs among a few other things
- Disallow SSH password-based authentication
- Allow SSH access only for your user (you can manually add more in `/etc/ssh/sshd_config` on the `AllowUsers` line)
- (Optionally) change the SSH port if you pass one in as a variable via `--extra-vars "ssh_port=YOUR_PORT"`

*The playbook defaults to using `deploy` as your username. Make sure to override it with `--extra-vars "user=YOUR_USER"` if you used something else, or you'll block yourself from SSHing in.*

It's important to note that this probably won't make your server completely impenetrable, but it's good enough for most purposes.

Run the following:

```
ansible-playbook secure.yml -i "YOUR_HOST," --ask-become-pass
```

