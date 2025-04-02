Contents:  
===========

1. [HPC infrastructure](https://github.com/LeahRoberts/HPC_notes/main/README.md#hpc-infrastructure) 
2. [Getting access](https://github.com/LeahRoberts/HPC_notes/main/README.md#getting-access) 
3. [Where is everything?](https://github.com/LeahRoberts/HPC_notes/main/README.md#where-is-everything?) 
4. [How to install miniforge](https://github.com/LeahRoberts/HPC_notes/main/README.md#how-to-install-miniforge) 
5. Submitting jobs
6. interactive jobs
7. How should I submit my jobs?
8. Who do I contact if I have issues? 


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

Ssubmit is a wrapper for slurm written by Michael Hall to assist in easily submitting jobs to slurm. Ssubmit can help you rapidly submit (many) little jobs. I use it in conjunction with sbatch scripting. 

```
# to run the same script as above, we simply do:
ssubmit -S '#!/bin/bash --login' -t 5m -m 1g multiqc "echo "Hello world" > newfile.txt"

# to activate a conda environment, we use:
ssubmit -S '#!/bin/bash --login' -t 5m -m 1g multiqc "conda activate fastp; multiqc ./fastqc"
```

