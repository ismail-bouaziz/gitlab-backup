# Introduction
Backup GitLab with an Ansible playbook and save the backup file in a backup VM on the OpenStack cloud.

# GitLab installations (RHEL / CentOS 7)

Install prerequisites:


$ sudo yum install -y curl policycoreutils-python-utils openssh-server perl

* Enable OpenSSH server daemon if not enabled: sudo systemctl status sshd:

$ sudo systemctl enable sshd
$ sudo systemctl start sshd

* If gitlab is configured to send mail to users, install postfix:

$ sudo yum install postfix
$ sudo systemctl enable postfix
$ sudo systemctl start postfix

* Add the GitLab package repository:

$ curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash

* Install the package, change '<version>' with your gitlab backup version. Make sure you have correctly set up your DNS, and change https://gitlab.example.com to the URL at which you want to access your GitLab instance.:

$ sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ee-<version>

# Restore for Omnibus GitLab installations

* First ensure your backup tar file is in the backup directory described in the gitlab.rb configuration gitlab_rails['backup_path']. The default is /var/opt/gitlab/backups. The backup file needs to be owned by the git user.

$ sudo cp 11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar /var/opt/gitlab/backups/
$ sudo chown git:git /var/opt/gitlab/backups/11493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar

* Restore prerequisites
You need to have a working GitLab installation before you can perform a restore. This is because the system user performing the restore actions (git) is usually not allowed to create or delete the SQL database needed to import data into (gitlabhq_production). All existing data is either erased (SQL) or moved to a separate directory (such as repositories and uploads).

To restore a backup, you must also restore the GitLab secrets.

These include the database encryption key, CI/CD variables, and variables used for two-factor authentication.

Without the keys, multiple issues occur, including loss of access by users with two-factor authentication enabled, and GitLab Runners cannot log in.

Restore:

/etc/gitlab/gitlab-secrets.json (Linux package)
/home/git/gitlab/.secret (self-compiled installations)
Rails secret (cloud-native GitLab)
This can be converted to the Linux package format, if required.
You may also want to restore your previous /etc/gitlab/gitlab.rb (for Omnibus packages) or /home/git/gitlab/config/gitlab.yml (for installations from source) and any TLS keys, certificates (/etc/gitlab/ssl, /etc/gitlab/trusted-certs), or SSH host keys

* Stop the processes that are connected to the database. Leave the rest of GitLab running:

$ sudo gitlab-ctl stop puma
$ sudo gitlab-ctl stop sidekiq
$ sudo gitlab-ctl status

* Next, restore the backup, specifying the timestamp of the backup you wish to restore:

$ sudo gitlab-backup restore BACKUP=11493107454_2018_04_25_10.6.4-ce

* Reconfigure, restart and check GitLab:

$ sudo gitlab-ctl reconfigure
$ sudo gitlab-ctl restart
$ sudo gitlab-rake gitlab:check SANITIZE=true



![GitLab](https://1000logos.net/wp-content/uploads/2023/04/Gitlab-logo-500x281.png)
