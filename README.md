

<h1>Hybrid Cloud Network Using a Samba server within a Red Hat EC2 instance hosted on AWS</h1>
Purpose of this is to set up a hybrid cloud network, using a Samba server within a Red Hat EC2 instance  hosted on AWS Cloud.<br />




<h2>Environments and Technologies Used</h2>

- AWS Subscription (free tier used for this walkthrough)
- A Red Hat Server (An EC2 instance)
- Windows Operating system (Winndows 11 Pro used for this walkthrough
  

<h2>Operating Systems Used </h2>

- Red Hat 9
- Windows 11 Pro (23H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Patch the server to ensure the latest updates are applied
- Install the Samba server dependence packages as well as the Samba server
- Open up the Samba Ports on the firewall
- Create the Samba Share directory
- Create the Samba User (required when clients attempt to connect to the samba share)
- Configure the Samba smb.conf file
- Start the network daemon
- Map to the samba share from a windows system


<h2>Prerequisites</h2>

- AWS Subscription (free tier used for this walkthrough)
- A Red Hat Server (An EC2 instance)
  - This includes set up of the host key
- Method of accessing the server (Windows Terminal or Putty. Note this example will use the Windows Terminal)
- Knowledge of the following
  - Creating and connecting to an AWS instance (**Placeholder for link to guide**)
  - Obtaining sudo permisions or using the root user
  - Basic knowlege of linux operating system


<h3>Step 1 - Patch the server to ensure the latest updates are applied</h3>

Run the following command
#yum update -y

<h3>Step 2 - Install the Samba server dependence packages as well as the Samba server</h3>

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/d735ea1e-7c7d-427d-ab4f-06cfc624f56b)

<h4>Additional Context</h4>

The RHEL9 EC2 server is sitting inside the AWS clound and has a firewall. This firewall uses Port22 which is open for Secure Shell (SSH) protocol (eg tools such as Putty).

To set up the file server, Samba will be installed. This enables access to the server and ability to crete shares that other systems can access.

However, Samba itself has its own ports (4), which will have to be opened.

<h3>Step 3 - Open up the Samba Ports on the firewall</h3>

Install and configure the local server firewall daemon (must be setup before ports are open

#yum install firewalld -y                       NOTE - daemon is called firewalld

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/ade3280f-bb14-4579-9b88-5b7d083e4dcb)

Start the daemon

#systemctl start firewalld

#systemctl enable firewalld  (to make it permanent and survive a reboot)

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/f5723a43-dcc3-4d42-a35a-2556466360f2)

To verify #systemctl status firewalld 

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/62f48506-3d5b-4c5c-8411-ebc9f76615ca)

Open up the Samba Ports

There are 4 Samba ports to be opened;
- Port 134
- Port 138
- Port 139
- Port 445

#firewall-cmd --permanent --zone=public --add-service=samba  (OPENS UP ALL SAMBA PORTS AT ONCE)

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/d01ed948-16b3-4547-8fc8-9e255e3cf1b4)

verify samba ports re opened with 
#firewall-cmd --list-all

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/e9ddb737-5ba4-4f71-8e8a-fe5487b8e58c)

Server can now receive traffic!

**NOTE** There is an alternative method of opening ports (one by one) shown below;
- #firewall-cmd --permanent --add- port=137/tcp
- #firewall-cmd --permanent --add- port=138/tcp
- #firewall-cmd --permanent --add- port=139/tcp
- #firewall-cmd --permanent --add- port=445/tcp
- #firewall-cmd --reload
- #firewall-cmd --list-all


<h3>Step 4 - Create the Samba Share directory</h3>

#mkdir /class39-samba

#chmod 777 /class39-samba      (This gives read/write/execute permissons to everyone for the directory)

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/ab0e45dc-2852-4922-a498-8fa4a0c0fe20)

<h3>Step 5 - Create the Samba User</h3>

This will be required when clients attempt to connect to the samba share.

#useradd sambauser

#smbpasswd -a sambauser      (This will assign a password to the account and also adds the user account to the samba security database)

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/a585a96d-bd77-4ff2-b8f3-23551fece578)

<h3>Step 6 - Configure the Samba smb.conf file</h3>

There is a configuration file **smb.conf**. This file determines how the Samba server functions

#vi /etc/samba/smb.conf               (vi command is a text editor)

enter the following details 




[aws-samba39]                          _(share label as seen by client systems)_

comment = class39 aws samba-share      _(description of the share)_

path = /class39-samba                  _(share directory)_

force user = sambauser                 _(all users will use this to authenticate clients)_

writeable = yes                        _(grants read/write permission o the share for clients)_

:wq!                       _(to save the entries)_




<h3>Step 7 - Start the network daemon</h3>

#systemctl start smb

#systemctl enable smb


Grant connection access to the server


#chcon t samba_share_t /class39-samba

This command is used to change the SELinux (Security-Enhanced Linux) security context of a file or files/directories.


![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/38a7aed1-71de-4503-81ce-98aea069bf9c)


<h3>Step 8 - Map to the samba share from a windows system</h3>


Open up Windows Explorer (This walkthrough is aon a windows system)

Enter the following 

\\<public IP of Samba Server> into folder explorer        (AWS server IP address is available on the instance page_

Connect with the username and passward set up eariler in Step 5

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/f33a2346-da0d-442d-82a9-0d3a27efa610)

The following shares will now be visible.

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/b171cc7d-5465-4399-99b4-1f4b1aaff75a)


Right click on the aws samba and click  map to network drive. Drive T was used for this example.


![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/3cbcf9d0-01b2-4e9f-9705-aa9818eaf962)


The share has now been set up.

To summarise, you have now set up a share between a Windows OS on premise and a Samba server in Red Hat, hosted in the AWS Cloud.

You are now able to share files between both systems. 

For example, a picture file was added in the share from thee Windows side

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/26578212-7c54-45ca-958d-234cb606aaf8)


And now on the server

![image](https://github.com/EmmanuelAOlu/hybridcloud-aws--rhel9-samba/assets/173094278/e3f5dd03-3fd1-4d25-969c-2f6c0e86767b)


