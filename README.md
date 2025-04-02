Contents:  
===========

1. [HPC infrastructure](https://github.com/LeahRoberts/HPC_notes/blob/main/README.md#hpc-infrastructure) 
2. [Getting access](https://github.com/LeahRoberts/HPC_notes/blob/main/README.md#getting-access) 
3. [Where is everything?](https://github.com/LeahRoberts/HPC_notes/blob/main/README.md#where-is-everything) 
4. [How to install miniforge](https://github.com/LeahRoberts/HPC_notes/blob/main/README.md#how-to-install-miniforge) 
5. [Submitting jobs](https://github.com/LeahRoberts/HPC_notes/blob/main/README.md#submitting-jobs) 
6. [interactive jobs](https://github.com/LeahRoberts/HPC_notes/blob/main/README.md#interactive-jobs) 
7. [How should I submit my jobs?](https://github.com/LeahRoberts/HPC_notes/blob/main/README.md#how-should-i-submit-my-jobs) 
8. [Who do I contact if I have issues?](https://github.com/LeahRoberts/HPC_notes/blob/main/README.md#who-do-i-contact-if-i-have-issues) 


## HPC infrastructure

High Performance Computing (HPC) is the use of a large cluster of computers and supercomputers to process big data that could not otherwise be analysed with conventional compute resources. Bunya is the UQ HPC, and is a free resource to UQ students and staff. 

The general layout of a HPC is this: 
1. You log in to a "login node" on the HPC. This is like a waiting area, and is not meant for any analyses/heavy compute to be run. You can do things like open and write to text files, copy (small numbers) of files etc.
2. You submit jobs via a workload manager. There are several workload managers; Bunya uses [Slurm](https://slurm.schedmd.com/documentation.html). This enables you to request resources to run jobs, which are then queued for running and returned when completed. This organisation allows many users to all equally access the Bunya resources.
3. You run jobs in a `scratch` space, and store results in your RDM/s. The `scratch` space is usually a disk that is amenable to high I/O activity, which makes it suitable for running analyses (particularly if they generate multiple intermediate files). The RDM, however, has slow disk read/write speeds, and should not be used at all for running analyses. You should either sym link or copy raw files from the RDM to your scratch space, perform you analysis in scratch, and then copy _only the files you wish to keep_ back to your RDM. 

When you request and gain access to Bunya, you will have limited permissions and space to carry out analyses. The default space includes (you can check this with `rquota`): 

Type | Home | scratch | RDM
--- | ---| ---| ---
disk space | 50G | 150G | 1TB/RDM
file number | 1 million | 100,000 | 1 million

Your permissions will be restricted to your own allocations on Bunya as well as any read/write permissions you have based on your `groups` (run as command to see who you belong to!). 

You can request **special scratch space** using the following link: [Form to request special scratch space](https://forms.office.com/r/nKiVRrEiyE)

RCC-support have crafted a number of helpful resources for Bunya:
* [Bunya description and access links](https://rcc.uq.edu.au/systems/high-performance-computing/bunya)
* [Bunya guide on GitHub](https://github.com/UQ-RCC/hpc-docs/blob/main/guides/Bunya-User-Guide.md)
* TRAINING (highly recommended!) - contact rcc-support@uq.edu.au to request to join their monthly Bunya training session


## Getting access

To obtain access to Bunya, you need to complete [this form](https://forms.office.com/r/87rfgvxDnz). 

You will likely need to complete this with your supervisor, as it requests details about the project you will be doing, and what resources will be required. 

Once you have access, you will need to join the UQCCR group. You can request this [here](https://services.qriscloud.org.au/access/4b0d521402d34a219a77285ffc0b1ec6/member).

Once that is complete, you can start using Bunya!

## Where is everything? 

The main places you will be navigating to include: 

```
home dir: /home/<your username>
scratch: /scratch/user/<your username>
RDMs: /QRISdata/QXXXX # replace with your RDM number
```

_I can't find my RDM on Bunya, help?_
There are a few reasons this could be - check you have been invited and accepted the invitation to the RDM. It can take ~24 hours for it to become available to you. Secondly, RDMs are not linked to Bunya by default - this is a special request that needs to be made when the RDM is created (it is a checkbox asking if you need storage allocations to be accessible by Bunya). 

## How to install miniforge

As a disclaimer, Bunya does have miniconda installed, which can be activated using `module load miniconda`. 

However, if you want to install your own conda/mamba, I would recommend installing in either your `/home` or `/scratch` space. Follow the instructions provided [here on the Miniforge github](https://github.com/conda-forge/miniforge?tab=readme-ov-file#unix-like-platforms-macos-linux--wsl) to install. During the install, you can pretty much answer `Y` to everything. The only exception is at the end, it will ask if you want to `conda init`. Enter `N` or `no`. 

Once completed, you will have conda/mamba installed. To use, you will have to source the `conda.sh`/`mamba.sh` scripts before using `conda activate`: 

```
[uqlrobe6@bunya2 ~]$ # <--- your usual prompt
[uqlrobe6@bunya2 ~]$ source /home/uqlrobe6/miniforge3/etc/profile.d/conda.sh
[uqlrobe6@bunya2 ~]$ source /home/uqlrobe6/miniforge3/etc/profile.d/mamba.sh
[uqlrobe6@bunya2 ~]$ conda activate
(base) [uqlrobe6@bunya2 ~]$# <--- conda envs now available
```

Now you can create and activate conda environments. You will need to do this every time you (a) log in, or (b) change to a different worker node. 

## Submitting jobs

When you submit a job, all the paths will be relative to where you have submitted the script. I.e. if you submit the script in your home directory, the output (unless specified otherwise) will be written to your home directory. Similarly, if you are pointing to a particular file, you  need to give the relative filepath based on where you are submitting the job (or absolute filepath to avoid this problem).

### example for sbatch

Usually I write the script in a text file and then copy/paste over to a `.sh` file in Bunya. The sbatch script should look like this: 

```
#!/bin/bash --login
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1     # change this one for threads per task
#SBATCH --mem=10G             # change this to the amount of memory you need
#SBATCH --job-name=Test       # change this to the name of your run
#SBATCH --time=1:00:00        # change this for the amount of time needed (hours:minutes:seconds)
#SBATCH --partition=general
#SBATCH --account=a_uqccr
#SBATCH -o slurm-%j.output
#SBATCH -e slurm-%j.error

# module-loads-go-here
# you need to source your conda.sh and mamba.sh if you haven't already

# conda activate <env> goes here

# script stuff goes below:

echo "Hello world" > newfile.txt
```

Then to run, you do:
`sbatch <script_name.sh>`

To check the job is running, use `squeue --user=<your username>`. 
To check info on a job that has run, use `seff <jobID>`. This will tell you how many resources your job consumed vs. how much you requested, and can be good for optimising resource allocation. 

### example of ssubmit

[Ssubmit](https://github.com/mbhall88/ssubmit) is a wrapper for slurm written by Michael Hall to assist in easily submitting jobs to slurm. Ssubmit can help you rapidly submit (many) little jobs. I use it in conjunction with sbatch scripting. 

```
# to run the same script as above, we simply do:
ssubmit -S '#!/bin/bash --login' -t 5m -m 1g multiqc "echo "Hello world" > newfile.txt"

# to activate a conda environment, we use:
ssubmit -S '#!/bin/bash --login' -t 5m -m 1g multiqc "conda activate fastp; multiqc ./fastqc"

#You can combine ssubmit with for loops, to submit many jobs at once:

cat names.txt | while read name; do ssubmit -S '#!/bin/bash --login' -t 5m -m 200m ${name}_job "echo ${name} > ${name}_newfile.txt"; done
```

## Interactive jobs

Interactive jobs can be useful for when you want to quickly run things, or interactively debug a particular problem. Instead of submitting jobs to a distant worker node and getting the results and output back, you can enter a worker node with the requested resources in order to directly run your analysis.

The general command for an interactive job is: 
`salloc --nodes=1 --ntasks-per-node=1 --cpus-per-task=1 --mem=2G --job-name=TinyInteractive --time=01:00:00 --partition=general --account=a_uqccr srun --export=PATH,TERM,HOME,LANG --pty /bin/bash -l`

Change the `--cpus-per-task`, `--mem` and `--time` as required. 

There is also the onBunya resource (https://rcc.uq.edu.au/systems/software-platforms-workflow-tools/onbunya) - need to look into this more...

## How should I submit my jobs? 

Consider the job/s you are running, and how long each job will take, and what resources are required. What is the most efficient way to run my analysis? How many times will I need to run this analysis? 

For example, say you have 10 bacterial WGS samples, and you want to run assembly on each of them. You could submit a single sbatch script that run assembly sequentially on each of them. This will take probably 20G RAM, but likely a day or so to complete. Additionally, if one sample fails, it may cause the entire analysis to exit (so you would need to debug and restart).

Alternatively, you could submit a job for each of the 10 bacterial WGS samples and run then in parallel. This would overall take more RAM (~15G each) but would probably finish in 2-3 hours. A single assembly failing would not affect the outcome of the other jobs, as they have been submitted independently. You do need to consider where the output files are being written, to make sure your output is going to independent folders. 

If you need to run something big, and only once, you can be a bit cowboy with how to get it done (needs must, and all). But if you are running something lots of times, it is preferable to be more aquainted with the specific resources needed so you can be as efficient as possible. 

### I'm just going to request 100G of memory because I don't know how much it actually needs

This is _bad_ practice. Mainly because, for each job you run, the resources you request are *charged* against your account. This means that if you request 100G every time, even if you only use 2G, you are charged the full 100G. Why is this bad? The more resources you use, the lower your `fairshare` score becomes, meaning your job priority goes down. This mean when you go to submit more jobs, you will be placed at the bottom of the queue. You should try to get an estimate of how much RAM you need prior to running a large number of jobs. 


## Who do I contact if I have issues? 

Your first contact should be your supervisor. Make sure you have information on (a) what you tried to run, (b) the data you are using, and (c) what the error message is.

Your second contact (or for anything that seems to be HPC related) should be rcc-support (rcc-support@uq.edu.au). 

RCC support also runs a number of meetups to troubleshoot issues (https://rcc.uq.edu.au/training-support/meetups).
