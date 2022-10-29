---
layout: post
title:  "Using Columbia University's Ginsburg HPC"
date:   2022-10-28 12:00:00 -0800
categories: educational
---

# Introduction
Ginsburg is the name of Columbia University's newest high performance computing cluster (HPC). As a Data Science Master's student at the university I've been using Ginsburg for my work in the [Jovanovic Lab](https://placeholder.com). There's plenty of very basic stuff in [Ginsburg's documentation](https://placeholder.com) for how to do the basics. Here I want to cover some more involved things which could improve your quality of life while using or developing code, dramatically decrease the amount of time it takes to run code, or make some things feasible which were previously infeasible.

# Glossary
*Host Machine*: The machine you're logging into Ginsburg from. This is probably a laptop or desktop computer. You could also login via another server, but this is usually unlikely.

*Remote Machine*: This is Ginsburg in our case. But generally, it's a computer that you want to access from a distance over a network (not necessarily the internet).

# How to Avoid Typing Your Password Each Time 
This seems like a pretty pedantic thing, but if you're logging in and out of Ginsburg all the time, it can get pretty annoying. Fortunately there's a pretty simple solution. Your computer checks a folder called `~/.ssh/` whenever you login via `ssh`. Specifically it checks whether a matching key is available on the remote machine. It's not exactly a copy but the details are not super important.

## Step 1.
First we want to check if the key is already available

```
ls ~/.ssh/
```

If the files `id_ed25519` and `id_ed25519.pub` are already there, skip to [Step 3](#step-3)

## Step 2. 
Assuming the keys are not there, we'll generate them with the following command

```
ssh-keygen -t ed25519
```

Follow the prompts (you can leave everything blank) and it should look something like this.

```
Generating public/private ed25519 key pair.
Enter file in which to save the key ($HOME/.ssh/id_ed25519):     
Enter passphrase (empty for no passphrase):  
Enter same passphrase again:  
Your identification has been saved in $HOME/.ssh/id_ed25519.
Your public key has been saved in $HOME/.ssh/id_ed25519.pub.
```

## Step 3.
Now we want to put the public key onto Ginsburg with the following command. Make sure to replace `<UNI>` with your actual UNI. This part will still prompt you to put in your password.

```
cat ~/.ssh/id_ed25519.pub | ssh <UNI>@ginsburg.rcs.columbia.edu 'cat - >> ~/.ssh/authorized_keys' 
```

This command basically opens one of the files we just created, sends it over to Ginsburg, and tells Ginsburg to recognize the host computer (i.e. your laptop, desktop, etc) whenever we use `ssh`.

## Step 4.
Double check that everything works by logging out of Ginsburg (you can just type `exit` and then hit Enter to leave Ginsburg).

# Performing Multiple Jobs Concurrently
A really common use case in biology labs is performing some time-consuming processing/analysis on multiple files. More generally, sometimes we want to run some code repeatedly with different parameters and the inputs of each run are not dependent on the outputs of another run. As a result we don't care how quickly each thing finishes as long as we don't have to do them one after another. 

I've found the easiest way to do this is to create a template bash script and use python to 'configure' each template and submit that configuration as a job. I'll show this with a concrete example.

## Problem
