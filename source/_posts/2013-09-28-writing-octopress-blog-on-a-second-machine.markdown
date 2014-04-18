---
layout: post
title: "Writing Octopress blog on a second machine"
date: 2013-09-28 08:21
comments: true
categories: [生活]
---

A couple of days ago, I installed octopress on my own air, yesterday I wanted to test if I could write blogs from the machine in my company. With some effort, I did it. As a backup, I record it down here. This is specially for mac users.

###Clone existing repository into the new machine

Following command will clone your `source` branch to your local octopress folder.

```
$ git clone -b source git@github.com:username/username.github.com.git octopress
```
It was necessary to clone the `master` branch into your octopress/_deploy, but because of recent version of octopress, you first clone it somewhere else in your system, eg: on the desktop. 

```
$ git clone git@github.com:username/username.github.com.git _deploy 
```


###Set up your environment for octopress

That means in order to run octopress, precondition is to let your terminal not complain about the following commands: `ruby`, `gem`, `rbenv`, `rake`. So that you can run the rake command to configure your octopress.

```
$ cd octopress	
$ gem install bundler
$ rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
$ bundle install
$ rake setup_github_pages
```

In recent versions of Octopress, running `rake setup_github_pages` will remove your \_deploy folder and init a new repo inside of it, that's why in the above step you shouldn't clone your `maser` branch into your octopress/\_deploy, or else you will get an error when you do `rake deploy` at last.

Then you move your desktop local clone of `master` branch (_deploy folder) into your octopress folder.

###Pushing changes from multiple machines
Each time you want to do some changes, you need to do a pull op first.

```
$ cd octopress
$ git pull origin source  # update the local source branch
$ cd ./_deploy
$ git pull origin master  # update the local master branch
```

Now you can safely change and push, without worring the merge or push error.

```
$ rake generate
$ git add .
$ git commit -am "Some comment here." 
$ git push origin source  # update the remote source branch 
$ rake deploy             # update the remote master branch
```

### References:
http://wywon.com/blog/2012/11/25/build-octopress-on-another-pc/
http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/