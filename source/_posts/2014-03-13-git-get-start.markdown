---
layout: post
title: "git get start"
date: 2014-03-13 07:21
comments: true
categories:
---
config
    git config --global user.name "John Doe"
    git config --global user.email "johndoe@example.com"

how to get help
    git config --help
    git help config
    man git-config


start up
    git init
    git add .
    git commit -m "Depot Scaffold"

    add and commit
    git commit -a -m "Depot Scaffold"

compare with version before last commit and last commit
    git diff HEAD^ HEAD

    # show diff stat
    git diff --stat

    # show the tree-like view
    git log --graph --oneline --all

specify the file path
    git diff HEAD^ HEAD app/models/product.rb


git ammend the last commit
    git commit -amend

powerful edit commit command
    git rebase --interactive HEAD^5

git include delelted files
    git add -A


Creates a remote named "origin" pointing at your GitHub repository
    git remote add origin https://github.com/username/Hello-World.git

Sends your commits in the "master" branch to GitHub
    git push origin master

pull down changes
    git pull orgin master
