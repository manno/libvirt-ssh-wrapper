# libvirt-ssh-wrapper - remotely manage libvirt instances

## Description

Wrapper to allow management of virtual machines  via ssh.

Only one system account is needed. VM owners are recognized by their SSH key.
Besides rebooting, the script allows users to create an encrypted VNC connection to their virtual machine.

The script has been tested with KVM machines in libvirt 0.9

## Install

The installation involves creating an account, adding several ssh keys and restricting these
keys to specific libvirt virtual machines.

1) Put the wrapper script into /usr/local/sbin

Just copy the script and make it executable.

    cp libvirt-manage /usr/local/sbin/libvirt-manage
    chmod 755 /usr/local/sbin/libvirt-manage

2) Create the configuration file

Put the wrapper scripts configuration file in /etc/libvirt/user_vm.yaml

    ---
    vnc_host: localhost
    server_name: libvirt.example.com
    auth_file: /home/vmuser/.ssh/authorized_keys

3) Create the management account

    adduser vmuser

4) Restrict the management accounts login shell

    chsh vmuser /bin/rbash

5) Put the new user into a group for vm owners

    addgroup --system vmowner
    adduser vmuser vmowner

6) Setup sudo

Allow members of the vmowner group to run the wrapper script as root

    Cmnd_Alias VMMAN = /usr/local/sbin/libvirt-manager
    Defaults:%vmowner !env_reset
    %vmowner ALL=NOPASSWD: VMMAN

7) Setup SSH

Create an .authorized_keys file in the $HOME/.ssh folder of the management user

    mkdir -p ~vmuser/.ssh/
    touch ~vmuser/.ssh/authorized_keys
    chown -R vmuser ~vmuser/.ssh
    chmod -R go-rwx ~/vmuser/.ssh

8) Add SSH keys

Add ssh public key entries for every VM. This will enforce the wrapper script and link a key to a single vm, as the command= options contains the users VM name.

    command="sudo /usr/local/sbin/libvirt-manager vmname",permitopen="localhost:5920",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDYBNgHlybOch9DA908IVSOp8owZn6OgFl+uD7ixokF/v/RR9atzAna9exLIpMEVouOYKlMciwnekZcODZlvDMFu2BJQN7mI9RcwR337Cm44NutVDwBdB2u9/y5/ry4XI/NSFjQBR9BL7stIDli5/G+hyx43L4m1Tvc5agqMXvXkrfc5XZzrp/PGMLthR2dvf1DmqM4EkZCgSpumidv0fyLfTYPt8ijTKWi3wvILzfzNb7lblt03xdS/g+er6avDDhlYtgMpfIc+U1ACTdWcp4laAa7ONSeG4hTM0eLp5T3YSxifvZ0ey2Z2IQKe0CqDQKFVa3hmKyq86YgJpAz1pvZ vmname-owner@example.com

9) libvirt and VNC

Check libvirt machines configuration for VNC settings. For example use dynamic vnc ports:

    <graphics type='vnc' port='-1' autoport='no' keymap='en-us'/>

## Usage

    ssh -4t vmuser@server_name help|start|shutdown|destroy|status|vnc

### VNC tunnel setup

The vnc subcommand is special, it needs to be executed twice.

First get the required command line from the remote server:

    ssh -4t vmuser@server_name vnc

Execute the same command with the tunnel options returned by the previous command.

    ssh -4t -L5900:localhost:5902 vmuser@server_name vnc

