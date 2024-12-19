# Automating Backups in Linux Using rsync and crontab

This project shows how to use the rsync tool and crontab for periodic backups to automate backups between two AWS EC2 instances. SSH is used for safe data transfer between the source and destination computers in the automated and effective backup procedure.

## Table of Contents
1. [Project Overview](#project-overview)
2. [Prerequisites](#prerequisites)
3. [AWS Setup](#aws-setup)
4. [Installation and Configuration](#installation-and-configuration)
5. [Automating Backups](#automating-backups)
6. [Technologies Used](#technologies-used)

## Project Overview

This project makes it possible for two AWS EC2 instances to automatically and effectively backup one another. The backup script, which copies data to the destination instance via rsync, is executed by the source instance. Crontab is used to automate the backup procedure and run the backup on a regular basis. Password-less authentication and SSH for secure communication are included in the configuration.

## Prerequisites

Make sure the following conditions are satisfied before starting the project:

- AWS account with EC2 access.
- Two EC2 instances (one as the source, the other as the destination).
- Linux-based operating system on both EC2 instances (Amazon Linux 2 or Ubuntu).
- SSH access between the source and destination machines.
- `rsync` and `crontab` installed on both EC2 instances.

## AWS Setup

### 1. Log in to AWS Management Console
- Navigate to the [AWS Management Console](https://aws.amazon.com/console/).
- Log in using your credentials.

### 2. Launch EC2 Instances
- Create two EC2 instances:
    - **Source Instance**: `backup_source`
    - **Destination Instance**: `backup_destination`
- Use the following configuration for both instances:
    - **AMI**: Amazon Linux 2 or Ubuntu.
    - **Instance Type**: `t2.micro` (Free tier eligible).
    - **Key Pair**: Create or use an existing key pair.
    - **Security Group**: Allow **port 22** (SSH) and any other required traffic.
- Verify both instances are running and accessible via SSH.

## Installation and Configuration

### **Prepare EC2 Instances for Backup**

#### On the Source Machine:

1. Set the hostname:
    ```bash
    sudo hostnamectl set-hostname backup-source
    ```

2. Ensure all repositories are enabled:
    ```bash
    sudo yum repolist all
    ```

3. Check if `rsync` is installed:
    ```bash
    rpm -qa | grep rsync
    ```

#### On the Destination Machine:

1. Ensure repositories are enabled:
    ```bash
    sudo yum repolist all
    ```

2. Check and install `rsync` if not present:
    ```bash
    sudo yum install rsync -y
    ```

### **Data Setup**

#### On the Source Machine:

1. Create a backup directory and test files:
    ```bash
    mkdir -p /backup_source
    touch /backup_source/test{1..10}.txt
    cal > /backup_source/cal.txt
    vim /backup_source/demo.txt  # Add some text to the file
    ```

#### On the Destination Machine:

1. Create a destination directory:
    ```bash
    mkdir -p /backup_destination
    ```

## Automating Backups

### Copy Data Between Machines Using `rsync`

1. Run `rsync` from the **Source Machine** to copy data to the **Destination Machine**:
    ```bash
    rsync -av -e ssh /backup_source/* root@<destination_ip>:/backup_destination/*
    ```

2. Verify the files on the **Destination Machine**:
    ```bash
    ls /backup_destination
    ```

### Set Up Password-less SSH Authentication

1. On the **Source Machine**, generate an SSH key:
    ```bash
    ssh-keygen
    ```

2. Copy the public key to the **Destination Machine**:
    ```bash
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@<destination_ip>
    ```

3. Test password-less SSH:
    ```bash
    ssh root@<destination_ip>
    ```

### Automate Backup with `crontab`

1. Create a backup script on the **Source Machine** (`/root/backup.sh`):
    ```bash
    #!/bin/bash
    rsync -av -e ssh /backup_source/* root@<destination_ip>:/backup_destination/*
    ```

2. Make the script executable:
    ```bash
    chmod +x /root/backup.sh
    ```

3. Add the script to `crontab` for periodic execution:
    ```bash
    crontab -e
    ```
    Add the following line to run the script every minute:
    ```bash
    * * * * * /bin/bash /root/backup.sh
    ```

4. Verify `crontab` is set up:
    ```bash
    crontab -l
    ```

5. Monitor `crontab` execution logs:
    ```bash
    tail -f /var/log/cron
    ```

### Verify the Automation

1. Check if the files are periodically synced to the **Destination Machine**:
    ```bash
    ls /backup_destination
    ```

2. Confirm the process is working as expected.

## Technologies Used

- **AWS EC2**: To host both the source and destination instances.
- **rsync**:For effective data synchronization between the instances at the source and the destination.
- **crontab**: For automating the backup process at regular intervals.
- **Linux**: Amazon Linux 2 or Ubuntu as the operating system for both EC2 instances.

