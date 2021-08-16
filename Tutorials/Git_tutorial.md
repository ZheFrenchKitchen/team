## How to use git


```shell

# Create git repository, the remote and push
git init  .
###########################
# Go on github , create a repository 
###########################

# add repo
git remote add origin your_repo.git 
# The files in your dir are tracked only. 
git add .
# Files are committed, means any changes will be followed now.
git commit -m "First commit"

# enable
git config --global https.proxy https://proxy.company.com:8888
# disable
git config --global --unset-all http.proxy

git branch -M main
git push -u origin -M main

# Classiquement...(proxy doesn't let us)
# you could configure .ssh.config as detailed here https://gitolite.com/git-over-proxy.html
# need to be root 
# so you need to switch to https before doing the final line

# SWITCH
git remote set-url origin https://github.com/ZheFrench/Tcd8.git

git push -u origin main



#List the files and their states.

git status -s

#Add only modified files.

git add -u

# Give you the list of remote repository
git remote -v

# Create you upstream dir
git remote add origin https://github.com/ZheFrench/Toulouse.git

# Rename the url if you need...
git remote set-url origin https://github.com/ZheFrench/Toulouse.git

# Send code
git push origin
#id:ZheFrench/pwd:Camille34

# If you previously delete the file not using git rm , the file is staying in the remote...do that and i will disapear during your next push
git rm --cached 'file name'

# You did a mistake and push
git revert --no-commit <commit>

# You get the list of your git
git log --oneline

# Push a tagged version
git tag -a v1.4 -m "my version 1.4"
git push origin v.1.4

# remove locally tag
git tag -d  v.1.4

# if you want to remove remotely
git push origin :refs/tags/v.1.4

# for the new token thing, I add to do some stuffs for git to ask me the new token 
https://stackoverflow.com/questions/68775869/support-for-password-authentication-was-removed-please-use-a-personal-access-to
git config --global credential.helper cache

```
