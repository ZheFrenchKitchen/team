# Install GitLab

---


---


```shell

cat /etc/lsb-release 

sudo apt-get update  

sudo apt-get install mailutils  

# Si tu configues rien tu peux revenir au choix de config avec 

sudo dpkg-reconfigure postfix
https://packages.gitlab.com/gitlab/gitlab-ce
```

```

setting synchronous mail queue updates: false
setting myhostname: compute0.crlc.intra
setting alias maps
setting alias database
changing /etc/mailname to compute0.crlc.intra
setting myorigin
setting destinations: $myhostname, compute0.crlc.intra, localhost.crlc.intra, , localhost
setting relayhost: 
setting mynetworks: 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
setting mailbox_size_limit: 0
setting recipient_delimiter: +
setting inet_interfaces: all
setting inet_protocols: all
WARNING: /etc/aliases exists, but does not have a root alias.

Postfix (main.cf) is now set up with a default configuration.  If you need to 
make changes, edit /etc/postfix/main.cf (and others) as needed.  To view 
Postfix configuration values, see postconf(1).

After modifying main.cf, be sure to run 'service postfix reload'.

Running newaliases
Traitement des actions différées (« triggers ») pour libc-bin (2.27-3ubuntu1.4)
```
curl  https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo -E os=ubuntu dist=bionic bash
```shell
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo -E bash

<pre>Detected operating system as Ubuntu/bionic.
Checking for curl...
Detected curl...
Checking for gpg...
Detected gpg...
Running apt-get update... done.
Installing apt-transport-https... done.
Installing /etc/apt/sources.list.d/gitlab_gitlab-ee.list...done.
Importing packagecloud gpg key... done.
Running apt-get update... done.

The repository is setup! You can now install packages.</pre>

```
You can download directly here.  

https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_14.2.0-ce.0_amd64.deb


You need to change first wget to work behind proxy....

```
sudo nano /etc/wgetrc
sudo dpkg -i gitlab-ce_14.2.0-ce.0_amd64.deb

Thank you for installing GitLab!
GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

Help us improve the installation experience, let us know how we did with a 1 minute survey:
https://gitlab.fra1.qualtrics.com/jfe/form/SV_6kVqZANThUQ1bZb?installation=omnibus&release=14-2


```

```
sudo -E EXTERNAL_URL="https://compute0" apt-get install gitlab-ce
```

Pr regarder si le DNS est bien configurer, il doit renvoyer status: NOERROR

```
dig compute0


```
 <<>> DiG 9.11.3-1ubuntu1.15-Ubuntu <<>> compute0
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22946
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;compute0.      IN  A

;; ANSWER SECTION:
compute0.   0 IN  A 127.0.1.1
compute0.   0 IN  A 10.100.0.93

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Fri Aug 20 16:29:52 CEST 2021
;; MSG SIZE  rcvd: 69
```


```
echo "This is the body of the email" | mail -s "This is the subject line" jpvillemin@gmail.com

```

sudo nano /etc/gitlab/gitlab.rb

external_url 'http://compute0.crcl.intra/gitlab'

sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart

sudo more /var/log/mail.log
sudo nano /etc/postfix/main.cf


smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = compute0.crlc.intra
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
mydestination = compute0.crlc.intra, compute0.crlc.intra, localhost.crlc.intra, , localhost
relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all

```
```
(base) jp@ubuntu:~$ set +H
(base) jp@ubuntu:~$ echo -ne "XQeR2?t3;n!6" | base64
WFFlUjI/dDM7biE2
(base) jp@ubuntu:~$ echo -ne "jean-philippe.villemin@inserm.fr" | base64
amVhbi1waGlsaXBwZS52aWxsZW1pbkBpbnNlcm0uZnI=
(base) jp@ubuntu:~$ 
```

openssl s_client -connect smtp.inserm.fr:587 -starttls smtp
EHLO compute0.crlc.intra
AUTH LOGIN
amVhbi1waGlsaXBwZS52aWxsZW1pbkBpbnNlcm0uZnI=
WFFlUjI/dDM7biE2
MAIL FROM:jean-philippe.villemin@inserm.fr
rcpt to:jpvillemin@gmail.com
DATA
Subject:test message
This is the body of the message!
Blablah
.
quit

nc -v -u smtp.inserm.fr 587
echo "Corps de mail" | mail -s "Sujet" -a "From: jean-philippe.villemin@inserm.fr" jpvillemin@gmail.com

###! Docs: https://docs.gitlab.com/omnibus/settings/smtp.html
###! **Use smtp instead of sendmail/postfix.**

 gitlab_rails['smtp_enable'] = true
 gitlab_rails['smtp_address'] = "smtp.inserm.fr"
 gitlab_rails['smtp_port'] = 587
 gitlab_rails['smtp_user_name'] = "jean-philippe.villemin@inserm.fr"
 gitlab_rails['smtp_password'] = "XQeR2?t3;n!6"
 gitlab_rails['smtp_domain'] = "smtp.inserm.fr"
 gitlab_rails['smtp_authentication'] = "login"
 gitlab_rails['smtp_enable_starttls_auto'] = true
 #gitlab_rails['smtp_tls'] =false
 #gitlab_rails['smtp_pool'] = false

###! **Can be: 'none', 'peer', 'client_once', 'fail_if_no_peer_cert'**
###! Docs: http://api.rubyonrails.org/classes/ActionMailer/Base.html
# gitlab_rails['smtp_openssl_verify_mode'] = 'peer'

# gitlab_rails['smtp_ca_path'] = "/etc/ssl/certs"
# gitlab_rails['smtp_ca_file'] = "/etc/ssl/certs/ca-certificates.crt"

### Email Settings

#gitlab_rails['gitlab_email_enabled'] = true

##! If your SMTP server does not like the default 'From: gitlab@gitlab.example.com'
##! can change the 'From' with this setting.
gitlab_rails['gitlab_email_from'] = 'jean-philippe.villemin@inserm.fr'
# gitlab_rails['gitlab_email_display_name'] = 'Example'

 alertmanager['admin_email'] = 'jean-philippe.villemin@inserm.fr'


user['username'] = "gitlab" # defaults to git
sudo gitlab-ctl remove-accounts
sudo gitlab-ctl uninstall



sudo gitlab-ctl stop
sudo gitlab-ctl uninstall
sudo gitlab-ctl cleanse
sudo dpkg -P gitlab-ce


 Je fais pas le remove de ca.
opt/gitlab/
/var/opt/gitlab/
/etc/gitlab/
/var/log/gitlab/

Si ca marche pas
sudo systemctl restart gitlab-runsvdir

/etc/gitlab/initial_root_password.
Username: root
Password: puGtxpibmgFyP1dRsriIOsNidbHr9B8dl/edoJSdk5w=
https://docs.gitlab.com/ee/security/reset_user_password.html#reset
-your-root-password.


#How to backup ?


https://docs.gitlab.com/omnibus/settings/backups.html

sudo gitlab-backup create BACKUP=dump GZIP_RSYNCABLE=yes

## Storing  a backup of your configuration files

```
/etc/gitlab/gitlab-secrets.json
/etc/gitlab/gitlab.rb
```

Create /data/GITLAB (rsync by eric).

In /etc/gitlab/gitlab.rb, uncoment the following line :

```
sudo nano /etc/gitlab/gitlab.rb
```

```
gitlab_rails['backup_keep_time'] = 604800
```

```
sudo gitlab-ctl reconfigure
sudo gitlab-ctl backup-etc --backup-path /data/GITLAB/ GZIP_RSYNCABLE=yes

--delete-old-backups, which will delete any backups older than the current time minus the backup_keep_time, if backup_keep_time is greater than 0.
```

```
sudo ls /etc/gitlab
gitlab.rb  gitlab-secrets.json  trusted-certs
sudo ln -s /etc/gitlab/gitlab.rb /data/GITLAB/gitlab.rb 
sudo ln -s /etc/gitlab/gitlab-secrets.json /data/GITLAB/gitlab-secrets.json

```

## Storing a backup of your repositories and GitLab metadata

Default :

```
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
```
Change To :

```
gitlab_rails['backup_path'] = "/data/GITLAB/"
```

sudo gitlab-ctl reconfigure
sudo gitlab-backup create BACKUP=dump GZIP_RSYNCABLE=yes


#Configuring cron to make daily backups
The CRON=1 environment setting directs the backup script to hide all progress output if there aren’t any errors. This is recommended to reduce cron spam.

```
sudo crontab -e -u root

```
There, add the following line to schedule the backup for everyday at 4 AM:
```
0 4 * * * /opt/gitlab/bin/gitlab-backup create BACKUP=dump GZIP_RSYNCABLE=yes CRON=1
```


#How to restore ?

https://docs.gitlab.com/ee/raketasks/backup_restore.html#back-up-gitlab
Disabling prompts during restore
sudo GITLAB_ASSUME_YES=1 gitlab-backup restore


#Save code in data (not system disk)


git_data_dirs({ "default" => { "path" => "/data/GITLAB/git-data" } })

sudo rsync -av /var/opt/gitlab/git-data/repositories /data/GITLAB/git-data/
# Prevent users from writing to the repositories while you move them.
sudo gitlab-ctl stop

# Note there is _no_ slash behind 'repositories', but there _is_ a
# slash behind 'git-data'.
sudo rsync -av /var/opt/gitlab/git-data/repositories /data/GITLAB/git-data/

# Start the necessary processes and run reconfigure to fix permissions
# if necessary
sudo gitlab-ctl reconfigure

# Double-check directory layout in /mnt/nas/git-data. Expected output:
# repositories
sudo ls  /data/GITLAB/git-data/

# Done! Start GitLab and verify that you can browse through the repositories in
# the web interface.
sudo gitlab-ctl start

C'est crypté...

https://docs.gitlab.com/ee/administration/repository_storage_types.html#translate-hashed-storage-paths

 sudo ls ./git-data/repositories/@hashed/6b/86
6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.git  6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.wiki.git

Tu peux retrouver le nom du projet....
```
sudo gitlab-rails console
ProjectRepository.find_by(disk_path: '@hashed/6b/86
6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b').project

```
sudo gitlab-ctl reconfigure after restoring a configuration backup
```