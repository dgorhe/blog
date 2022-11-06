---
layout: post
title:  "Using Columbia University's Ginsburg HPC"
date:   2022-10-28 12:00:00 -0800
categories: educational
---

# Introduction
Ginsburg is the name of Columbia University's newest high performance computing cluster (HPC). As a Data Science Master's student at the university I've been using Ginsburg for my work in the [Jovanovic Lab](http://jovanoviclab.com/). There's plenty of very basic stuff in [Ginsburg's documentation](https://confluence.columbia.edu/confluence/display/rcs/Ginsburg%3A+Getting+Started). I want to cover some more involved things which could improve your quality of life while using or developing code, dramatically decrease the amount of time it takes to run code, or make some things feasible which were previously infeasible.

---

# Create an Alias for Interactive Sessions
Let's start with something easy. Oftentimes, we want interactive environments for coding but you need more resources than your local machine has. Ginsburg is useful for this because you can get a *lot* of resources (hopefully) quickly and at a moments notice. What's annoying is that the command might look something like this:

```
srun --pty -t 0-03:00 -A mjlab --mem=64gb -N 1 -c 32 /bin/bash
```

You can look at the [documentation](https://confluence.columbia.edu/confluence/display/rcs/Ginsburg+-+Submitting+Jobs#:~:text=shell%20will%20wait.-,Basic%20Job%20Directives,-The%20following%20table) for what each part means, but the bottom-line is that this is extremely annoying to type repeatedly. It's especially annoying because you probably want the same resources each time you start an interactive session. We can solve this by creating an alias for this command. Open the `.bashrc` file in your respective home directory. This file contains a bunch of settings for the shell that Ginsburg shows to you (i.e. the terminal when you login). Use your text editor of choice. In my case, I'll use vim.

```
vim ~/.bashrc
```

The file will look something like this

```
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/burg/opt/anaconda2-2019.10/anaconda2/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/burg/opt/anaconda2-2019.10/anaconda2/etc/profile.d/conda.sh" ]; then
        . "/burg/opt/anaconda2-2019.10/anaconda2/etc/profile.d/conda.sh"
    else
        export PATH="/burg/opt/anaconda2-2019.10/anaconda2/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

You can ignore most stuff but toward the top where all the `alias ...` commands are, we'll add our alias. In my case, I'm calling it `sesh`

```
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
alias sesh='srun --pty -t 0-03:00 -A mjlab --mem=64gb -N 1 -c 32 /bin/bash'

...
```

I'm not showing the remainder of the file again to save some space, but I didn't change or delete any of it. You can always come back and change this, but think of this as a default. Now when you run 

```
sesh
```

you'll start an interactive session with all the settings you put in that alias. 

---

# How to Avoid Typing Your Password Each Time (Option 1)
This seems like a pretty pedantic thing, but if you're logging in and out of Ginsburg all the time, it can get pretty annoying. Fortunately there's a pretty simple solution. Your computer checks a folder called `~/.ssh/` whenever you login via `ssh`. Specifically it checks whether a matching key is available on the remote machine. It's not exactly a copy but the details are not super important.

### Step 1.
First we want to check if the key is already available

```
ls ~/.ssh/
```

If the files `id_ed25519` and `id_ed25519.pub` are already there, skip to [Step 3](#step-3)

### Step 2. 
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

### Step 3.
Now we want to put the public key onto Ginsburg with the following command. Make sure to replace `<UNI>` with your actual UNI. This part will still prompt you to put in your password.

```
cat ~/.ssh/id_ed25519.pub | ssh <UNI>@ginsburg.rcs.columbia.edu 'cat - >> ~/.ssh/authorized_keys' 
```

This command basically opens one of the files we just created, sends it over to Ginsburg, and tells Ginsburg to recognize the host computer (i.e. your laptop, desktop, etc) whenever we use `ssh`.

### Step 4.
Double check that everything works by logging out of Ginsburg (you can just type `exit` and then hit Enter to leave Ginsburg).

---
# How to Avoid Typing Your Password Each Time (Option 2)
If option 1 doesn't interest you, there's another option, `sshpass`. `sshpass` simply forwards your password to the ssh prompt automatically. If you're using a public private key pair (i.e. option 1), at some point the remote machine has to read your private key. If you trust the server, this is not an issue, but if you're not sure, using `sshpass` could be more secure. Passwords are typically hashed on the host end (i.e. your laptop, desktop, etc) so you're never giving your sensitive credentials to the remote machine. However, it seems like the maintainer of `sshpass` is no longer actively maintaining the software. This could mean that there are unresolved bugs which could impact the security of the software. Both options are equally as convenient once you set them up.

The easiest way to install `sshpass` is via Homebrew, a package manager for MacOS. As of writing this, there is no Windows version available. Please look at the Homebrew homepage for instructions on [installing Homebrew](https://brew.sh/). Once you have Homebrew installed, in your terminal simply type:

```
brew install hudochenkov/sshpass/sshpass
```

Once it's installed in your `.bashrc` or `.zshrc` add the following alias 

```
alias ginsburg='sshpass -p '<PASSWORD>' ssh <UNI>@ginsburg.rcs.columbia.edu'
```

You can replace `ginsburg` with whatever alias name you would like. Also, replace `<PASSWORD>` with your actual password and `<UNI>` with your actual UNI. With all that set up, when you type in the following and hit enter, you should be logged into the Ginsburg cluster.

```
ginsburg
```

---
# Performing Multiple Jobs Concurrently
A really common use case in biology labs is performing some time-consuming processing/analysis on multiple files. More generally, sometimes we want to run a program multiple times with different parameters and the inputs of each run are not dependent on the outputs of another run. As a result we don't care how quickly each thing finishes as long as we don't have to do them one after another. 

I've found the easiest way to do this is to create a template bash script and use python to 'configure' each template and submit that configuration as a job. I'll show this with a concrete example.

### Problem
We have a bunch of files called `.bed` files. We want to annotate them using a program called `annotator`. However, there are 50 files and each file takes 15-20 minutes to run. If we ran them all one after the other it would take 12-16 *hours* to run. Let's see how we can submit multiple jobs concurrently.

### Solution
Here is the directory structure of our folder

```
├── submit_jobs.py
├── submit.sh
└── template.sh
```

`template.sh` is the file which is actually sent as an individual batch job. `submit_jobs.py` creates a group of jobs based on `template.sh`. `submit.sh` is simply a convenience script for configuring 1 or more groups of jobs. The reason for this structure is that `submit_job.py` looks at all the `bed` files in a folder. If we want to run multiple folders, we have to run `submit_job.py` multiple times each with different parameters. If you knew that you only needed to run 1 folder at a time, you could simply type out the contents of `submit.sh` into your terminal and hit enter. Let's look at each file in more detail.

Here is the `template.sh` file
```
#!/bin/sh
#SBATCH --account=mjlab        
#SBATCH --job-name=annotator-single-run
#SBATCH --cpus-per-task 4
#SBATCH --time=01:00:00            
#SBATCH --mem-per-cpu=20G
 
module load anaconda/2-2019.10
source /burg/opt/anaconda2-2019.10/anaconda2/etc/profile.d/conda.sh
conda activate annotator

annotator --input ${INPUT} --output ${OUTPUT} --gtfdb ${GTFDB} --species hg38
```

The various `#SBATCH` lines tell Ginsburg how to identify this job and what kind of resources we want. The middle chunk of lines loads the appropriate version of Anaconda and activates a conda environment we've created for the annotator program. Conda is simply a way of creating isolated python containers. Each container is typically only used for one program and it contains only the other programs it depends on to run. So in our case, the old version of annotator requires Python 2.7 and older versions of packages which are stored in the `annotator` conda environment.

Here is our file `submit_jobs.py`
```
import sys
import os 
from argparse import PARSER, ArgumentParser

# Argument parser to decide where to look for inputs, outputs, GTF database, and logs
parser = ArgumentParser()
parser.add_argument("--input", help='Directory containing bed files')
parser.add_argument("--output", help='Directory to write outputs of annotator')
parser.add_argument("--logs", help='Directory to write log files')
parser.add_argument("--gtfdb", help='Location of gtfdb')
args = parser.parse_args()

if __name__ == "__main__":
    # Turn command line arguments into absolute paths
    input = os.path.abspath(args.input)
    output = os.path.abspath(args.output)
    gtfdb = os.path.abspath(args.gtfdb)
    logs = os.path.abspath(args.logs)

    try:
        assert os.path.exists(input) and os.path.exists(output) and os.path.exists(gtfdb) and os.path.exists(logs)
    except AssertionError:
        print("One of the file paths you entered can't be found")
        print("Input absolute path:", input, "\n")
        print("Output path:", output, "\n")
        print("GTFDB absolute path:", gtfdb, "\n")
        print("Logs absolute path:", logs, "\n")
        sys.exit(0)
        
    
    # Create lists of absolute paths for inputs and outputs
    bed_files = [os.path.join(input, f) for f in os.listdir(input)]
    output_files = [os.path.join(output, os.path.basename(bed).split(".")[:-1][0]) + ".txt" for bed in bed_files]

    # Iterate through each combination of bed file and output file and submit a job
    for bed, out in zip(bed_files, output_files):
        # Take the bed file name, strip the .bed extension and replace it with .log
        logfile = os.path.basename(bed).split('.')[:-1][0] + ".log"
        
        # Turn logfile into an absolute path
        logpath = os.path.join(logs, logfile)
        
        # Build command in components
        command = "sbatch "
        command += f"-A mjlab "
        command += f"-o {logpath} "
        command += f"--export=INPUT={bed},OUTPUT={out},GTFDB={gtfdb} "
        command += f"template.sh"
        os.system(command)
```
At the top we initialize an argument parser to read in some command line arguments. Then we check if the file paths that were put in are actual file paths. We then perform some string manipulation to rename the output files as `.txt` versions of the original file names. Finally, in the main for-loop, we construct a command line command as a string for each bed file we want to annotate. This line is the most important.

```
command += f"--export=INPUT={bed},OUTPUT={out},GTFDB={gtfdb} "
```

Notice that `INPUT`, `OUTPUT`, and `GTFDB` are the same variables we specfied in our `template.sh` script. The final string gets passed to `os.system()` which runs the command as if you were typing it out in the terminal. An example command might look something like this

```
sbatch -A mjlab -o ./logs --export=INPUT=./bedfiles,OUTPUT=./out,GTFDB=hg41.gtf.db template.sh
```


Here is an example of the `submit.sh` file
```
ROOT="/burg/mjlab/users/ew2579/"

python submit_jobs.py \
	--input "$ROOT/spidr/encode_data/downsampled_peaks_filtered" \
	--output "$ROOT/darvesh_annotator/output/encode/downsampled_peaks_filtered" \
	--logs "$ROOT/darvesh_annotator/logs/encode/downsampled_peaks_filtered" \
	--gtfdb "$ROOT/spidr/reference_files/gencode.v41.annotation.gtf.db"
```

`ROOT` is specified at the top to save some typing. The final command to run this would be the following:

```
bash submit.sh
```

### Outcome
This would run the `python` command with the proper arguments for all the directory paths. The python program in turn creates various batch jobs by parsing each file in the directory. The various batch jobs are independent of one another so they're all running separately. The result is that what would've taken 12-16 hours now only takes 30 to 45 minutes to run.

---
# Building Custom Singularity Containers
Containers are like miniature computers inside your computer that can package your application in a very isolated way. Ginsburg like most HPCs does not allow users to build or run Docker containers which is the most popular containerization software. This is mainly because Docker requires root privileges on the computer which is risky on a shared resource like an HPC.

### Step 1.
First we want to create a singularity container to contain all the dependencies of our project since it uses both R and Python. In our case, there is an existing Docker container which can do this. We want to build the container on our local machine. Go to the directory with the `Dockerfile` and run

```
 docker build . -t <tag name>
```

`<tag name>` is whatever tag you want to use to easily identify your container.

### Step 2.
Save the image as a `tar` file.

```
docker save <tag name> -o <filename>.tar
```

`<tag name>` is the same `<tag name>` from [Step 2](#step-2-1) and `filename` is whatever you want your output file to be called. If I used `darveshgorhe/secat-dev` as my tag name and `secat` as my filename I would run 

```
docker save darveshgorhe/secat-dev -o secat.tar
```

### Step 3.
Upload the `tar` file to Ginsburg using `scp`. You can also use the Globus tool to upload the `tar` file wherever you want.

```
scp <filename>.tar '<UNI>@ginsburg.rcs.columbia.edu:<path to store tar file>'
```

### Step 4.
On Ginsburg, build your container from the tar file as a sandbox container

```
singularity build --sandbox <sandbox name> docker-archive://<filename>.tar
```

---
# Developing a Python Package on Ginsburg
I've recently been trying to improve a package called [SECAT](https://github.com/grosenberger/secat) for analyzing SEC-SWATH mass Spectrometry data. The package requires a lot of memory and compute to run in a reasonable amount of time. Specifically, it requires more memory than I have available on my computer and it heavily utilizes multiprocessing to parallelize parts of the code. What might take hours on my laptop can be done in less than 20 minutes on the computer. Since I'm editing the code very often, running it on my local computer is impractical. How can we use Ginsburg to solve this problem?

### Step 1.
Make sure you have a singularity sandbox container with all the proper dependencies. If you're not sure how to make one, look at the [previous section](#building-custom-singularity-containers).

### Step 2.
First clone what ever repository you want with 

```
git clone <git repository link>
```

By default this copies the contents of the repository into a folder in your current working directory with the same name as the repository. So if I was in `/home/` and I ran `git@github.com:grosenberger/secat.git` it would create a folder called `secat` in `/home/` and the contents of the repository would be in `/home/secat`. If you want to copy it somewhere else, run

```
git clone <git repository link> <desired folder>
```

Where `<desired folder>` is where you want the contents of the repository to be copied to. Once this is done, navigate to the repository you downloaded. In the case of the example you could do `cd /home/secat` to enter into the repository folder. We're going to install the python package in editatable mode. Editable mode tells your Python interpreter to look at the folder you specified rather than the default location of your packages.

```
pip install --editable .
```

The `.` tells `pip` to look in the current folder for the python package, specifically the `setup.py` file. If this runs successfully, you should see a `.egg-info` file generated. This is what tells `pip` to look in the current folder rather than where it normally looks for packages.

Ideally you want to do this in the same directory as your sandbox so the file tree might look something like this:

```
sandbox
├── bin -> usr/bin
├── boot
├── dev
├── ...
python_package
├── Dockerfile
├── LICENSE
├── README.md
├── ...
```

### Step 3. 
Check that the sandbox + editable python package works properly. In my case I would edit some of the help documentation and then type in `singularity exec sandbox secat --help` to ensure that everything was installed properly. If it works you should see your changes reflected in the output of your command. Just running the program could also be a good way to check, but make sure your edits are reflected in the program's execution. Deleting an import and checking if it works is another quick way to check.


# Glossary
*Host Machine*: The machine you're logging into Ginsburg from. This is probably a laptop or desktop computer. You could also login via another server, but this is usually unlikely.

*Remote Machine*: This is Ginsburg in our case. But generally, it's a computer that you want to access from a distance over a network (not necessarily the internet).