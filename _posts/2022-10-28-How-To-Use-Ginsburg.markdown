---
layout: post
title:  "Using Columbia University's Ginsburg HPC"
date:   2022-10-28 12:00:00 -0800
categories: educational
---

# Introduction
Ginsburg is the name of Columbia University's newest high performance computing cluster (HPC). As a Data Science Master's student at the university I've been using Ginsburg for my work in the [Jovanovic Lab](https://placeholder.com). There's plenty of very basic stuff in [Ginsburg's documentation](https://placeholder.com). I want to cover some more involved things which could improve your quality of life while using or developing code, dramatically decrease the amount of time it takes to run code, or make some things feasible which were previously infeasible.

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
We have a bunch of files called `.bed` files. These are the result of some analysis done on RNA-sequencing (the details are not important). We want to annotate them using a program called `annotator`. However, there are 50 files and each file takes 15-20 minutes to run. If we ran them all one after the other it would take 12-16 *hours* to run. Let's see how we can submit multiple jobs concurrently.

Here is the directory structure of our folder

```
Add structure later
```

Here is our file `template.sh`
```
<template.sh>
```

Here is our file `submit_jobs.py`
```
<template.sh>
```

Here is our file `submit.sh`
```
<submit.sh>
```

# Building Custom Singularity Containers
Containers are like miniature computers inside your computer that can package your application in a very isolated way. Ginsburg like most HPCs does not allow users to build or run Docker containers which is the most popular containerization software. This is mainly because Docker requires root privileges on the computer which is risky on a shared resource like an HPC.

# Developing a Python Package on Ginsburg
I've recently been trying to improve a package called [SECAT](link) for analyzing SEC-SWATCH Mass Spectrometry data. The package requires a lot of memory and compute to run in a reasonable amount of time. Specifically it requires more memory than I have available on my computer and it heavily utilizes multiprocessing to parallelize parts of the code. What might take hours on a local machine can be done in less than 20 minutes on the computer. Since I'm editing the code very often, hours long runs are impractical. How can we use Ginsburg to solve this problem?

## Step 1.
First we want to create a singularity container to contain all the dependencies of our project since it uses both R and Python. In our case, there is an existing Docker container which can do this. We want to build the container on our local machine. Go to the directory with the `Dockerfile` and run

```
 docker build . -t <tag name>
```

`<tag name>` is whatever tag you want to use to easily identify your container.

## Step 2.
Save the image as a `tar` file.

```
docker save <tag name> -o <filename>.tar
```

`<tag name>` is the same `<tag name>` from [Step 2](#step-2-1) and `filename` is whatever you want your output file to be called. If I used `darveshgorhe/secat-dev` as my tag name and `secat` as my filename I would run 

```
docker save darveshgorhe/secat-dev -o secat.tar
```

## Step 3.
Upload the `tar` file to Ginsburg using `scp`. You can also use the Globus tool to upload the `tar` file wherever you want.

```
scp <filename>.tar '<UNI>@ginsburg.rcs.columbia.edu:<path to store tar file>'
```

## Step 4.
On Ginsburg, build your container from the tar file as a sandbox container

```
singularity build --sandbox <sandbox name> docker-archive://<filename>.tar
```

## Step 5. 
Check that the sandbox works properly.

