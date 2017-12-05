# AWS DIRECTIONS HELP
These steps will walk you through where to create credentials (Access Key ID and Secret Access Key) on AWS to use for provisioning VMs with Ansible.

## Login
Login to the AWS Console on [https://aws.amazon.com/](https://aws.amazon.com/)

![login](login-window.png)

## Go to IAM
Under Services click on IAM to reach the IAM Management Console

![iam](iam.png)

## Go to Users
On the left pane click on the users link

![users](users.png)

Now you need to either add a new user

![add-user](add-user.png)

OR find the user you want to add credentials <- this is more common

## Add Credentials
Make sure to save this!! (But don't upload to GitHub or someone will start mining Bitcoins on your account)

![create-key](create-key.png)

You now have your Access Key ID and Secret Access Key!

[Return to aws_lab_setup directions](../README.md)