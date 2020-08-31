# Lab 400: Using Rsync for application file sync
<!-- Comment out table of contents
## Table of Contents
[Introduction](#introduction)
-->

## Introduction

Welcome to an introduction of **rsync**. **Rsync** is a file copying tool that exists on most linux environments. It is capable of copying files locally and to/from another host over a remote shell. Its differentiating feat is its **delta-transfer algorithm** that focuses on syncing the changes observed in files. This optimizes the sync by reducing the amount of data sent over the network. **Rsync** is used extensively in **backups**, **disaster recovery**, and **mirroring** scenarios. 

**For this lab specifically, we will be working in the terminal exclusively. As a result, we've included the commands to run as we walk you through the lab steps.** 

**In the first part of this lab, we will sync files locally on our machines to get an introductory grasp of rsync.** 

**In the second and final part of the lab, we will simulate how an administrator would sync files between a local machine and a remote server.**

**This lab will leverage the existing infrastructure that was created previously from the terraform script.** 

### Objectives
- Understand the syntax of **rsync**.
- Sync files locally.
- Sync files between a local host and a remote server.

### Extra Resources
-   To learn more about rsync, please refer to the following Linux documentation: [rsync](https://linux.die.net/man/1/rsync)
-   To learn more about specific rsync use cases, please refer to the following tutorial: [rsync usage](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories-on-a-vps).

## Part 1. Syncing files between two folders on the same local machine.

### **Step 1:** Create two folders. Name one "primary_sync" and the other "standby_sync".

```
<local_machine>$ mkdir primary_sync standby_sync

<local_machine>$ ls 
```

### **Step 2:** Create a file in "primary_sync" and name it "primary.txt".

```
<local_machine>$ touch primary_sync/primary.txt

<local_machine>$ ls primary_sync 
```

### **Step 3:** Verify that only "primary_sync" has a file in it and "standby_sync" is empty.

```
<local_machine>$ ls primary_sync/ standby_sync/
```

### Before we use rsync to sync the file between the folders, a little about the option flags we'll be using:
#### "-a" represents "archive". It allows us to sync all files in the source directory recursively while preserving any symbolic links, special and device files, modification times, group, owner, and permission.
#### "-v" represents "verbose". This flag lets us know what's happening when the command is run.
#### "-P" represents the combination of the "progress" and "partial" flags which allow us to see the transfer progress bar as well as the resumption of interrupted transfers.
#### "-n" represented "dry-run". This flag shows what content would have been transferred, useful to test out connectivity to the target host as well as target folder access. 

### **Step 4:** Great, now let's execute our first sync between the two folders. Let's include the "-n" flag first to make sure everything is in place before we actually sync the file.

```
<local_machine>$ rsync -avP primary_sync/ standby_sync -n

<local_machine>$ ls primary_sync/ standby_sync/
```

### **Step 5:** Removing the "-n" flag will sync the file over to the "standby_sync" folder.
```
<local_machine>$ rsync -avP primary_sync/ standby_sync 
 
<local_machine>$ ls primary_sync/ standby_sync/
```
### Nice work! You've successfully used rsync to sync a file between two local folders.




## Part 2. Syncing files between a local host and a remote server.

### **Step 1:** Add the public key to your bastion server to your machine's key chain (authentication agent).
```
<local_machine>$ ssh-add -k ~/<bastion_server_public_key>
```

### **Step 2:** Next SSH into the remote server. There are two new flags to consider:
#### "-A" enables us to forward our connection from an authentication agent. We populated this in the previous step.
#### "-J" allows us to connect to the target machine (application server) after connecting to the "jump host" (bastion server).
```
<local_machine>$ ssh -A -J opc@<bastion_public_ip> opc@<app_server_private_ip>
```

### **Step 3:** Now let's begin syncing files from our local machine to the remote server. Leave the terminal window that's connected to the remote server open and open a new terminal window. In the terminal that's connected to your local machine, create an application file folder and name it "app_files".
```
<local_machine>$ mkdir app_files

<local_machine>$ ls
```

### **Step 4:** Next, create an empty file in the application file folder and name it "app_file_1".
```
<local_machine>$ touch app_files/app_file_1

<local_machine>$ ls app_files
```

### **Step 5:** Now navigate back to the terminal window connected to the remote server. Verify that "app_files/app_file_1" does not exist on the server.
```
opc@<remote_server>$ ls 
```

### **Step 6:** We will now do a dry-run sync of the folder and its file from our local machine to the remote server. The syntax will differ a bit from Part 1 as we are leveraging a new option flag:
#### "-e" allows us to execute shell commands. We'll leverage this to hop into the remote server from the bastion server.
```
<local_machine>$ rsync -avP "ssh -A -J opc@<bastion_public_ip>" app_files opc@<app_server_private_ip>:/home/opc/ -n
```

### **Step 7:** After testing the sync with the "-n" flag, remove it and re-run the command. You should now see that the "app_files" folder and its contents are synced to the remote server. Navigate to the remote server terminal and verify.
```
opc@<remote_server>$ ls
```

### Congratulations, you successfully simulated the synchronization of files between a local machine and a remote server!



## (Optional) Part 3. Setting up periodic and automated syncs using Cron.
### Cron is a Linux tool that enables periodic repetition of commands.  

### **Step 1:** Run the following command to open a text editor. This editor will contain all of the **rules** for command repetitions. 
```
<source_machine>$ crontab -e
```

### Configuring the crontab rule:
```
Syntax: 'm h dom mon dow command'
m: minutes (0-59)
h: hours (0-23)
dom: day of the month (1-31)
mon: month (1-12)
dow: day of the week (0:sunday - 6:saturday)
command: command to execute
```
#### Example for automated sync job for every month, every day, every hour at 30 minutes:
```
30 * * * * rsync -avP source_folder <user>@<remote_server_ip>:target_folder
```
 
### **Step 2:** Save the file and close the editor. Then verify that the crontab saved.
```
<source_machine>$ crontab -l
```
### Great, the sync will now automate for the proposed period of time!


## Summary

-   In this lab, you learned how to use rsync to sync files on a local machine as well as from a local machine to a remote server.

-   **You are ready to move on to the next lab!**

[Back to top](#introduction)

