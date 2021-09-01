# Install GitLab-EE.

---

You can download directly here.  
You need to change first wget to work behind proxy and download the file....

```shell
sudo nano /etc/wgetrc

# In file : 
# You can set the default proxies for Wget to use for http, https, and ftp.
# They will override the value in the environment.
https_proxy = http://proxywsg.crlc.intra:3128
http_proxy = http://proxywsg.crlc.intra:3128

# Download :
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_14.2.0-ce.0_amd64.deb/download.deb
```

```shell
# install
sudo dpkg -i gitlab-ce_14.2.0-ce.0_amd64.deb
```

Configure all you need here.
```shell

# first, you need to get hostname
hostname -f

#Then open config file
sudo nano /etc/gitlab/gitlab.rb

```

```shell

# All stuffs you need to modify :

external_url 'http://compute0.crcl.intra/gitlab'


###! Docs: https://docs.gitlab.com/omnibus/settings/smtp.html
###! **Use smtp instead of sendmail/postfix.**

 gitlab_rails['smtp_enable'] = true
 gitlab_rails['smtp_address'] = "smtp.inserm.fr"
 gitlab_rails['smtp_port'] = 587
 gitlab_rails['smtp_user_name'] = "adresse@inserm.fr"
 gitlab_rails['smtp_password'] = "password"
 gitlab_rails['smtp_domain'] = "smtp.inserm.fr"
 gitlab_rails['smtp_authentication'] = "login"
 gitlab_rails['smtp_enable_starttls_auto'] = true

### Email Settings

gitlab_rails['gitlab_email_from'] = 'adresse@inserm.fr'

```

If you need to remove everything...

```shell
# Remove gitlab (be aware that linked files can be also removed)

sudo gitlab-ctl stop
sudo gitlab-ctl uninstall
sudo gitlab-ctl cleanse
sudo dpkg -P gitlab-ce

# Some people remove also these dirs. I did not.
/opt/gitlab/
/var/opt/gitlab/
/etc/gitlab/
/var/log/gitlab/

# You may have trouble after reinstall

#I didt his and it worked :
sudo systemctl restart gitlab-runsvdir
```


Each time you change something in this file, you need to do reconfigure ( see below) :
```shell

sudo gitlab-ctl reconfigure
#sudo gitlab-ctl restart (can be done also not mandatory)
``` 

**Note**: that when you first install gitlab, you have a root password creates here that will deleted in 24h : 

> /etc/gitlab/initial_root_password.
> https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.


# How to backup ?

---


We created a /data/GITLAB/. This directory should be backup by IT guy.

1. **git-data** (directory with the code we pushed, be carreful this cripted)
2. **gitlab-ce_14.2.0-ce.0_amd64.deb** (the initial package used to install gitlab)
3. **gitlab.rb** -> /etc/gitlab/gitlab.rb (symlink for configuration)
4. **gitlab-secrets.json** -> /etc/gitlab/gitlab-secrets.json (symlink for configuration)

```shell 
mkdir /data/GITLAB/git-data

sudo nano /etc/gitlab/gitlab.rb
# Do this in order to save pushed code not on the system disk ....
git_data_dirs({ "default" => { "path" => "/data/GITLAB/git-data" } })
```

```shell 
# Symbolik link for conf files
sudo ln -s /etc/gitlab/gitlab.rb /data/GITLAB/gitlab.rb 
sudo ln -s /etc/gitlab/gitlab-secrets.json /data/GITLAB/gitlab-secrets.json
```

Gitlab offers several to backup : Repositories and Config.
We discuss that later.  

https://docs.gitlab.com/omnibus/settings/backups.html


## Storing  a backup of your configuration files.


I went with the solution on the symlinks of the config that sould be rsynced by IT guit in /data/GITLAB/

But it worth mentionning what gitlab proposes.
This adding timestamp each times...don't want to copy 1000 files...so finally didn't end up with this solution.

```shell 
sudo gitlab-ctl backup-etc --backup-path /data/GITLAB/ 

```


```shell
sudo nano /etc/gitlab/gitlab.rb

gitlab_rails['backup_keep_time'] = 604800
#--delete-old-backups, which will delete any backups older than the current time minus the backup_keep_time, if backup_keep_time is greater than 0.
# Not used finally

# reload your config file
sudo gitlab-ctl reconfigure

```


## Storing a backup of your repositories and GitLab metadata.


```shell
sudo nano /etc/gitlab/gitlab.rb

#Default : 
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"

#Change To :
gitlab_rails['backup_path'] = "/data/GITLAB/"
```

```shell
sudo gitlab-ctl reconfigure

# backup your repositories and GitLab metadata
sudo gitlab-backup create BACKUP=dump GZIP_RSYNCABLE=yes
```

## Configuring cron to make daily backups

```
sudo crontab -e -u root

```
> There, add the following line to schedule the backup for everyday at 4 AM:
```
0 4 * * * /opt/gitlab/bin/gitlab-backup create BACKUP=dump GZIP_RSYNCABLE=yes CRON=1
```
The CRON=1 environment setting directs the backup script to hide all progress output if there aren’t any errors. This is recommended to reduce cron spam.

**Note**: I do that because, if we need to recover everything I think this is better to follow gitlab guidance.
We did the backup also but it's crypted and I don't now how it will behave to restore data from these files. So I do both.

# How to restore ?

---

https://docs.gitlab.com/ee/raketasks/backup_restore.html#restore-for-omnibus-gitlab-installations

I did it on my virtual machine and it worked great :)

```shell 

wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_14.2.0-ce.0_amd64.deb/download.deb

sudo dpkg -i gitlab-ce_14.2.0-ce.0_amd64.deb

# Then replace with you backup files.
sudo cp gitlab.rb /etc/gitlab/gitlab.rb
sudo cp gitlab-secrets.json /etc/gitlab/gitlab-secrets.json

# If you work on different server, don't forget to redifine hostname, dir for backup etc etc in gitlab.rb

sudo gitlab-ctl reconfigure 
sudo gitlab-ctl start

sudo chown git.git dump_gitlab_backup.tar

tar -xvf dump_gitlab_backup.tar

sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
# Verify
sudo gitlab-ctl status

sudo gitlab-backup restore 

sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
sudo gitlab-rake gitlab:check SANITIZE=true

sudo gitlab-rake gitlab:doctor:secrets

```

# How to retrieve your password ?

I add troubles to access to root user with my restored gitlab.

Sometimes e-mail via smtp can be buggy...
You can do what follow if you have troubles.


https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root

```shell 

sudo gitlab-rails console
 user = User.find_by_username 'root'
 user.password = 'secret_pass'
 user.password_confirmation = 'secret_pass'
 user.send_only_admin_changed_your_password_notification!
 user.save!

```


### EnCrypted data... ?


EnCrypted data...
The pushed code is encrypted.

https://docs.gitlab.com/ee/administration/repository_storage_types.html#translate-hashed-storage-paths


```shell 
 sudo ls ./git-data/repositories/@hashed/6b/86
6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.git  6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.wiki.git
```
You can retrieve the name of the project using this....

```shell 
sudo gitlab-rails console
ProjectRepository.find_by(disk_path: '@hashed/6b/86
6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b').project
```

> **Notes** : 

```


```
