---
layout: post
title: "How to archive your github repositories"
published: true
categories:
  - Tutorial
tags:
  - git
---

Today I had to archive the old github repositories. One [blog post](http://skookum.com/blog/one-git-archive-script-to-rule-them-all/) on the net gives a cool script to do that. Here is just a tranpilation from `Bash` to `Powershell`. I just added two lines :  

- one to remind username and password in order to not be asked on each fetch
- one to forget them after the git stuff is done

#### So here is the script :

```bash
# Takes one parameter: a remote git repository URL.
#
# This is the stuff this script does:
#
# 1. Clones the repository
# 2. Fetches all remote branches
# 3. Compresses the folder
# 4. Deletes the cloned folder.
#
# Your remote repository is left untouched by this script.

# PowerShell version of
#   http://skookum.com/blog/one-git-archive-script-to-rule-them-all/

if(!($args.length -eq 1)) {
  Write-Host -Foreground red "Usage:`n  1- git repository URL"
  Exit 0
}

# Variable definitions
$gitUrl     = $args[0]
$pieces     = $gitUrl.split("/")
$gitName    = $pieces[$pieces.length -1]
$folderName = $gitName.split(".")[0]
$zipName    = "$folderName.gitarchive.$(date +%Y%m%d).zip"

# 
git config --global credential.helper cache

# Clone the repos and go into the folder
git clone --recursive $gitUrl

# go to repository folder
pushd $folderName

# Pull all branches
foreach ($branch in git branch -r | grep -v HEAD | grep -v master) {
  $trimedBranch = $branch.Trim()
  $temp         = $trimedBranch.split("/")
  $branchName   = $temp[$temp.length -1].Trim()
  git branch --track $branchName $trimedBranch
}

#Pull all remote data and tags
git fetch --all
git fetch --tags
git pull --all
git credential-cache exit
git gc

# come back to main folder
popd

# Create an archive of the directory
Add-Type -A System.IO.Compression.FileSystem
[IO.Compression.ZipFile]::CreateFromDirectory("$pwd\$folderName", "$pwd\$zipName")

# Remove the git clone
Remove-Item -Recurse -Force $folderName

Write-Host -Foreground Green "Done!"
Write-Host -Foreground Green "Your archived git repository is named $zipName"
```

Thanks to `[Mark Rickert](http://skookum.com/authors/mark-rickert/)`
