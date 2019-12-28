---
title: OpenVPN Server for AWS
summary: OpenVPN Server with unlimited users.
---

# OpenVPN Server for AWS

<img align="left" style="float: left; margin: 0 10px 0 0;" src="https://github.com/0x4447-office/0x4447_webpage_documentation/blob/master/docs/img/assets/openvpn.png?raw=true">

This is an OpenVPN server with no limits on how many accounts or active connections you can have. The limits depend on the Instance type you select rather than on arbitrary soft limits.

This means that the limiting factor is the performance of the server itself. If your user starts complaining about the connection being too slow, or you see the CPU time is above 75 percent, just change the instance type to a bigger one to accommodate the new traffic and users.

The setup also has built-in resilience; the EC2 Instance UserData takes in two environment variables, one for the Elastic IP ID, and the other for the EFS ID. If the first ID is present, the instance will always get the same IP address at boot time. If the EFS ID is present, we will mount that drive and keep the OpenVPN user database on it with all the `.ovpn` profiles files for your user. So even if your instance is terminated, as long as you provide the same EFS ID at boot time, all data and OpenVPN configuration will be present.

Secure your connection now.

# Our Diferenciating Faktor

We also want to let you know that this is not a regular product; what we build for the marketplace is what we use ourselves on a day-to-day basis. One of our signature traits is that we hate repetitive tasks that can easily be automated. If we find something repetitive in our day-to-day use of our products, rest assured that we'll automate the repetition.

We want to give you a good foundation for your ideas.

# Documentation

After the instance is up and running, you'll have some manual work to do. Bellow you'll find all the necessary details.

**WARNING**: text written in capital letters needs to be replaced with real values.

# The First Boot

Grab a cup of caffe since the first boot will be slowwer then what you are use to. This is due to the certificate that we need to geenrate for OpenVPN. Since at boot time there isn't much goin on in the system this process can take around 12 min depending on the instance type. But not to warry, this happens only when the certificate is not found in the system.

# Resilience

Our OpenVPN Server has build in resilience to make sure that you don't loose all your users, or lose connectiving by a chaning IP. For this to work you'll need to allocate an Elastic IP, and create a EFS Drive. And take note of the IDs that you'll get so you can use them in the EC2 UserData section.

# Manual Work

### Elastic IP Role

Before you can set the UserData, you have to attach a Role to the Instance with a Policy that has this Document:

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": "ec2:AssociateAddress",
			"Resource": "*"
		}
	]
}
```

This Policy Document will give the instance the ability to attach the Elastic IP to itself. The `"Resource": "*"` is on purpouse not becasue of lazyness, the `AssociateAddress` actions is not resource specific.

### EFS Security Group

When you create a new EFS, you'll deploy it in the same VPC and Subnet the instance will exists, but alos, you'll have to attach a Security Group, make sure the port `2049` is open twoards the drive, oterwise the EC2 Instance won't be able to mount the drive.

### Bash Script for UserData

Once you have evrything setup. You can replace the place holder values with the real IDs. Make srue to replace the values in all CAPS, with the real data.

```bash
#!/bin/bash
EFS_ID=fs-REPLACE_WITH_REAL_VALUE
EIP_ID=eipalloc-REPLACE_WITH_REAL_VALUE

echo EFS_ID=$EFS_ID >> /home/ec2-user/.env
echo EIP_ID=$EIP_ID >> /home/ec2-user/.env
```

Explanation:

1. Set the ID of the EFS drive.
1. Set the ID of the Elastic IP.
1. Append the EFS Drive ID to the .env file
1. Append the ElasticIP ID to the .env file

### Understand how UserData works

It is important to note that the content of the UserData field will be only executed once, when the Instacne starts for the first time. Meaning it won't be trigered if you stop and start the instacne. If you chose to not enable resiliance, and skip the UserData script at boot time, you won't be able to later on update the UserData with the script and expect the Elastic IP and the EFS drive to be mounted. You have to options: 

- Either you follow [this link](https://aws.amazon.com/premiumsupport/knowledge-center/execute-user-data-ec2/) for a work around 
- Or your start a new Instacne, this time with the right UserData, and then copy over from the old isntance to the new one all the configuration files.

# User Management

## How to create a OpenVPN user

1. Run this command and answer all the questions:

	`sudo bash /opt/0x4447/openvpn/ov_user_add.sh "USER_NAME"`

2. Every time you do so, a new `.ovpn` file will be created in the `openvpn_users` folder located in the `ec2-user` folder.  You can copy the new file to your local computer using the `SCP` command like so:

	`scp -i /cert/path/ SERVER_IP:/home/ec2-user/openvpn_users/USER_NAME.ovpn .`

3. Share the file with the selected user.

## How to delete a OpenVPN user

Just run the following command and set the right user name:

`sudo bash /opt/0x4447/openvpn/ov_user_delete.sh "USER_NAME"`

## How to list all the OpenVPN users

Since every time you create a user, a `.ovpn` configuration file is created. You can just list the content of the `openvpn_users` folder, like so:

` ls -la /home/ec2-user/openvpn_users`

The output is the list of all the users you have available for your OpenVPN server.

# Understant our files:

In our appliance we create files needed to make the server works, and here is the full list with an explanation:

- `/etc/openvpn/easy-rsa/keys`: this folder holds all the user that you create, and OpenVPN uses this `.pem` files for authentication.
- `/home/ec2-user/openvpn_users`: holds all the OpenVPN cnfiguration files for your users to be used in their OpenVPN clients.
- `/home/ec2-user/.env`: is where you should store the IDs to enable resiliance.

# Before you go in production

Be sure to test the server to make sure it behaves the way we advertise it, not becasue we don't belive it works correctly, but to make sure you are confortable with the product and knows how it works. Especially the resiliance mode.

Make sure to make a test user, see that all works, and then termiante the instace and start a new one with the correct UserData, and see if after the instacne booted you can still connect to the OpenVPN without any chagnes on the client side.

# Don't forget to backup your data

Make sure you regullary backup your EFS drive. One simple solution would be to use [AWS backup](https://aws.amazon.com/backup/).

# OpenVPN Clients

- Desktop
    - [Windows](https://openvpn.net/client-connect-vpn-for-windows/)
    - [MacOS](https://openvpn.net/client-connect-vpn-for-mac-os/)
    - [Linux](https://openvpn.net/vpn-server-resources/how-to-connect-to-access-server-from-a-linux-computer/)
- Mobile
    - [iOS](https://apps.apple.com/us/app/openvpn-connect/id590379981)
    - [Android](https://play.google.com/store/apps/details?id=net.openvpn.openvpn&hl=en)