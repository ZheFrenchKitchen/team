

## Make a virtual machine with VMware Workstation Player 16

---

I presume you already downloaded an iso image ubuntu-18.04.5-desktop-amd64.iso from here https://releases.ubuntu.com/18.04/.  

You will start WMware Workstation Player 16 with it.

When you try to connect, at first, one main problem is that your keyboard is qwerty whereas it's an azerty.  

If you want to change that, you need to do updates, but you need an internet connexion.  

In the next section, I will explain how to se VMware player and configure internet.

Also you will need to install open-vm-tools, open-vm-desktop, to copy paste stuffs between host and guest, and have a full screen for the linux guest.


### 1.How to setup Internet Connection inside linux VM

---

When you have access to internet inside the VM, you are safe and you can install whathever you want so that's the first thing to do.

Check this video to install the network correctly.  

> https://www.youtube.com/watch?v=H2j3nyl4muQ&ab_channel=IT%26Software  

You can configure 6 cores, 8Go Ram and 300Go Disk for the VM under linux if your machine is 16Go RAM with 12 cores for example.

You then look into Network/Proxy using "Show Applications" button search.  

You chose manual and you configure as show below.  
![alt text](https://github.com/ZheFrenchKitchen/team/blob/master/img/network.png "How to configure proxy in Ubuntu.")

You will use proxywsg.crlc.intra:3128

When you are outside the IRCM, you select disabled for proxy configuration.( but with this setup, in windows, you started the VPN before to start the VM)

Anyway, for now you should access internet by browser but you still can't update apt packages.  


### 2.How to setup Internet Connection for APT packages updates

---

https://www.serverlab.ca/tutorials/linux/administration-linux/how-to-set-the-proxy-for-apt-for-ubuntu-18-04/

_**Proxy**_ :

```shell
	sudo touch /etc/apt/apt.conf.d/proxy.conf
	sudo nano /etc/apt/apt.conf.d/proxy.conf
}
```
Acquire {
HTTP::proxy "http://proxywsg.crlc.intra:3128";
HTTPS::proxy "http://proxywsg.crlc.intra:3128";
}

Note : When your are home. In windows, disable proxy. Do the same in linux (in Network)
But for packages you need to change /etc/apt/apt.conf.d/proxy.conf

>Acquire {
  HTTP::proxy ""false";";
  HTTPS::proxy "false";";
}


### 3. Keyboard Configuration

---

You go region and langage and you chose French (azerty) . Reboot.
Change at the top right the icon of keyboard. English is the one by default. Select french.

### 4. Install open-vm-tools

---

In order to have full screen to copy past stuffs.

```shell
	sudo apt update
	nano apt upgrade
	sudo apt install open-vm-tools
	sudo apt install open-vm-tools-desktop
}
```

### 5. Connect to IRCM server from your VM

---

You should ask IT guy to have an account.  

#### CONNECT :

---

```shell
#Connection
ssh villemin@compute0.crlc.intra

#Password will be asked.
#If you want to connect without password requirement , you can send you rsa key to server (see https://www.ssh.com/ssh/copy-id#copy-the-key-to-a-server)
```

```shell
Mount SERVER : (we automate a mount later to access server in explorer, read below)
ssftp villemin@compute0.crlc.intra
```

https://www.tecmint.com/sshfs-mount-remote-linux-filesystem-directory-using-ssh/


#### Download R-studio

---

https://rstudio.com/products/rstudio/download/#download

```shell 
sudo apt -y install r-base
sudo apt install gdebi-core rstudio-1.4.1103-amd64.deb
```

####  Download Conda (To install bioinfo softwares)

---

You can do that on both your VM and server home.  
For example,with conda you can install your own R version and packages, STAR aligner, and a lot of other tools.  
No need to be root or an external IT guy.

https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html

bash Anaconda3-2020.11-Linux-x86_64.sh

### 6. Install IDE Eclipse (not mandatory, sublime text is also a good alternative)

---

An IDE is an environement to edit your code. (Rstudio is an IDE for R, Eclipse can be use whith all langages R,python,bash...)

Prob with proxy, read link below.

https://mkyong.com/web-development/how-to-configure-proxy-settings-in-eclipse/

```shell 
sudo apt install default-jre
sudo snap set system proxy.http="http://proxywsg.crlc.intra:3128"
sudo snap set system proxy.https="http://proxywsg.crlc.intra:3128"
```
You need to do that in order to what's next :
```shell 
sudo snap install --classic eclipse
```

Inside Eclipse :   

Windows NetWork Connection > Select Manuel and set proxy for http and https (not SOCKS)
Check for  Updates
http://www.pydev.org/updates

Click and drag to recognise Eclipse
https://marketplace.eclipse.org/content/statet-r

Probems with proxy when installing packages...due to this fucking proxy.

### 7. R notes

---

```shell 
Sys.getenv(https_proxy) is empty
Sys.setenv(https_proxy="http://proxywsg.crlc.intra:3128")
```

Modify some stuffs to use R 4 version not 3.4.4  
```shell 
sudo nano /etc/apt/sources.list  
```
ADD at the end deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran40/

```shell 
sudo apt update
sudo apt-get install r-base
```

```shell 
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install()
```

#### Install a package in R : 

---

```shell  
> BiocManager::install(c("edgeR"))
```
Note you can install R package also with conda...I did that on the remote server to install biocmanager because i had trouble to set up my own dir for the packages.


#### Share file between windows and VM.

---

That useful when you have something you created on linux (a png file you want to incorporate in publication written under word office ) that you want to retrieve in windows.

https://askubuntu.com/questions/29284/how-do-i-mount-shared-folders-in-ubuntu-using-vmware-tools

```shell
sudo vmhgfs-fuse .host:/SharedData /mnt/hgfs/ -o allow_other -o uid=1000

vmware-hgfsclient
```

my-shared-folder is SharedData created in windows in Documents directory (I think you need to configure that somewhere in WM Player since it will not find the path to the shared directory).

```shell
 sudo vmhgfs-fuse .host:/SharedData /mnt/hgfs/ -o allow_other -o uid=1000
```

Close shell and reopen :

If you want them mounted on startup, update /etc/fstab with the following:
Use shared folders between VMWare guest and host

>.host:/SharedData    /mnt/hgfs/    fuse.vmhgfs-fuse    defaults,allow_other,uid=1000     0    0

#### Connect remote server

ssh villemin@compute0 

#### Mount remote serveur

You can create a rsa key.  
Add it to the remote server in autorized key and then connect without password confirmation each time.  

**NB** : Eric the IT guy dit it for me but you can do it by yourself on bioinfo0.

Create a dir on desktop called serveur or anything else, and then do :

```shell
sshfs villemin@compute0:/data/ /home/jp/Desktop/serveur
# remove the dir (content is not erased)
sudo umout /home/jp/Desktop/serveur/
```

####  Auto-Mount is better !

---

You add that in your /etc/fstab. 

If you want to do that you need to create your id_rsa and push it to bioinfo0 before.

> villemin@compute0:/data/ /home/jp/Desktop/serveur fuse.sshfs defaults,_netdev,IdentityFile=/home/jp/.ssh/id_rsa,allow_other,follow_symlinks   0   0 

```shell
sudo mount -av (will mount every thing and ask for password)
```

#### Proxy for wget  (don't think it's mandatory, just with the network connection you configured before it should work) 


That didn't work the first time because the ftp_proxy was no configured.

> nano /etc/wgetrc (you can't modify that on server)  

```shell
https_proxy = http://proxywsg.crlc.intra:3128 
http_proxy = http://proxywsg.crlc.intra:3128
ftp_proxy = http://proxywsg.crlc.intra:3128
'''

Wget/Curl via proxy workaround...

```shell
wget -e use_proxy=yes -e http_proxy=http://proxywsg.crlc.intra:3128 https://github.com/ZheFrenchKitchen/RNASEQ-1/archive/master.zip
curl --proxy http://proxywsg.crlc.intra:3128 ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_36/gencode.v36.primary_assembly.annotation.gtf.gz -o gencode.v36.primary_assembly.annotation.gtf.gz

wget -e use_proxy=yes -e http_proxy=http://proxywsg.crlc.intra:3128 ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_36/gencode.v36.transcripts.fa.gz
``` 

#### Git (for experimented user...)

---

git config --global http.proxy http://proxywsg.crlc.intra:3128

It's boring when you switch from home to office because each time you need to reconfigure the proxy settings.

#### R version 4 R version 4.0.3 (2020-10-10) -- "Bunny-Wunnies Freak Out"


You need to set up some environements variable attached to your conda env. 

```shell
conda create --name r4-base
conda env config vars set R_LIBS_USER=/data/USERS/villemin/anaconda3/envs/r4-base/lib/R/library
conda env config vars set R_LIBS=/data/USERS/villemin/anaconda3/envs/r4-base/lib/R/library

# Seurat package hd5fr
 conda env config vars set  HDF5_USE_FILE_LOCKING='FALSE'

# This one will install r.4
 conda install -c conda-forge r-base

#### R version 4.1

You need to set up some environements variable attached to your conda env. 

```shell

conda create --name r4.1
conda env config vars set R_LIBS_USER=/data/USERS/villemin/anaconda3/envs/r4.1/lib/R/library
conda env config vars set R_LIBS=/data/USERS/villemin/anaconda3/envs/r4.1/lib/R/library

# Seurat package hd5fr
 conda env config vars set  HDF5_USE_FILE_LOCKING='FALSE'

First (in fact, update is useless) :

conda update -n base conda (do nothing...)

conda-forge r-base should install 4.1.1 but it puts 4.0.5 so to correct this bug use what follows.

# I tried to upgrade my version of conda 4.9.2 to 4.10.3
conda activate base
conda update --all
conda update -n base conda 

Collecting package metadata (current_repodata.json): failed

ProxyError: Conda cannot proceed due to an error in your proxy configuration.
Check for typos and other configuration errors in any '.netrc' file in your home directory,
any environment variables ending in '_PROXY', and any other system-wide proxy
configuration settings.

export http_proxy=http://proxywsg.crlc.intra:3128
export https_proxy=http://proxywsg.crlc.intra:3128

conda update -n base conda

```

#### Remote Jupiter Lab

```shell
conda install -c conda-forge jupyterlab
jupyter notebook --generate-config
jupyter notebook password

BiocManager::install('IRkernel')
#Depending on the version you set up
IRkernel::installspec(name = 'ir40', displayname = 'R 4.0')

# Screen and ctrl-a + ctrl-d to detach it and let it run in background forever
screen -S JUPITER
conda activate r4-base
# or modify stuffs in ~/.screenrc but need to check on internet what to modify
# Port 22 and http port must be open 
jupyter-lab --ip 0.0.0.0 --port 3980 --no-browser

# Direct http adress
http://compute0.crlc.intra:3980/lab

# On you local machine run this , for secure ssh tunelling, I don't remember if you really need this part...
ssh -N -f -L 8888:localhost:3980 villemin@compute0

# Now you can access to you remote files in your browser using : 
http://localhost:8888
```

#### apt-get behind no proxy

```shell
sudo apt-get -o Acquire::http::proxy=false <update/install> 

```
