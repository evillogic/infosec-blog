---
title: Git Setup CheatSheet
date: 2019-08-23
draft: false
description: Git cheatsheet for dummies. How to set up your local machine, new repositories, and gitignore
---
## Setting up your user

`git config --global user.name "Your name here"`  
`git config --global user.email "your_email@example.com"`  
add your ssh key (in ~/.ssh/id_rsa.pub) to your github account  
`ssh -T git@github.com`

credit to [kbroman](https://web.archive.org/web/20220520135811/https://kbroman.org/github_tutorial/pages/first_time.html)

## Setting up a new repository

Create an empty repository on github and clone it using the git address specified near the top of the page

`git clone git@github.com:username/new_repo.gitg`   
`cd new_repo   git commit -m "First Commit!"`  
`git push -u origin master`

credit to [kbroman](https://web.archive.org/web/20220520135811/https://kbroman.org/github_tutorial/pages/init.html)

## Setting up a .gitignore file

`vim .gitignore`  
`git rm -r --cached .`  
`git add .`  
`git commit -m "Added .gitignore"`  
`git push`

I had issues with this in powershell for some reason, using the Windows Subsystem for Linux worked like a charm though.

To make a new commit

`git add .`  
`git commit -m "next commit"`  
`git push`

I also highly recommend using `git status` before creating a new commit to check that everything is working correctly.