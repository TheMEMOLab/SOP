# SOP - Basic usage of SAGA Sigma2 high performance computing (HPC) cluster for the MEMO group users.

v1.0, May 06, 2021
Author: Arturo Vera-Ponce de Leon

### 1. What is this?
This document is intended to be a quick reference guide on basic usage of SAGA HPC cluster. For a complete reference please refere to documentation on [SAGA](https://documentation.sigma2.no/hpc_machines/saga.html) and
[Sigma2](https://www.sigma2.no/).

### 2. General usage.

 **2.1 How to obtain an account:**
 
To be abble to use the SAGA cluster you first need to apply for an account. Please use the following link [HPC user application form](https://www.metacenter.no/user/application/form/notur/) and fill the aplication form. 

* Chose the Project number **NN9864K**  in the application form as follow.
![pic1](https://github.com/TheMEMOLab/SOP/blob/main/images/application.png)

* Project manager: **Magnus Øverlie Arntzen**

**2.2 Login into SAGA**

To login into the cluster you need to establish a secure shell conexion to do this open the command line terminal and type:

```bash
ssh myusername@saga.sigma2.no
```
*Remeber to change myusername to the user name provided by Sigma2.

**2.3 $HOME and $PROJECT directories.**

As a user of SAGA you will have access to two main directories the $HOME and the $PROJECT. The $HOME directory has a limited space of **20 GiB / 100 K files**. Please only use the $HOME to login and keep small text and log files. For data transfer and temporary storage you can use the $PROJECT directory, this is allocated on ```/cluster/projects/nn9864k```. Please create a folder with your username under that directory by:

```bash
$ cd /cluster/projects/nn9864k
$ mkdir $USER
```

**The disk quota limit for this directory is 10 Tb. Please do not storage intermediate results and compress all your fastq and raw files using tools such pigz or tar**

**2.4 Display the disk usage**

To know how much space is used by the users you can use the ```dusage``` command as follows:

```bash
(base) [auve@login-3.SAGA auve]$ dusage 

+---------------------------+--------+----------+--------------+----------+-----------------+-----------+
|                      path |   pool |   backup |   space used |    quota |   files/folders |     quota |
|---------------------------+--------+----------+--------------+----------+-----------------+-----------|
|        /cluster/home/auve |      1 |      yes |     13.7 GiB | 20.0 GiB |          71 161 |   100 000 |
| /cluster/projects/nn9864k |      2 |      yes |     35.8 GiB | 10.0 TiB |          26 342 | 1 048 576 |
+---------------------------+--------+----------+--------------+----------+-----------------+-----------+

(*) this script is still being tested, unsure whether the backup information is correct
    please send suggestions/corrections to radovan.bast@uit.no
```

This will display the amount of data used in the $HOME and $PROJECT directory.

### 3. Submit a job

**3.1 Submit by a SLURM script**

To submit jobs, you need to write all the instructions you want the computer to execute. This is what an script is.

SLURM uses a bash (computer language) base script to read the instructions. The first lines, are reserved words that SLURM needs to read inorder to launch the program:

```
-p --partition <partition-name>       --pty <software-name/path>
--mem <memory>                        --gres <general-resources>
-n --ntasks <number of tasks>         -t --time <days-hours:minutes>
-N --nodes <number-of-nodes>          -A --account <account>
-c --cpus-per-task <number-of-cpus>   -L --licenses <license>
-w --nodelist <list-of-node-names>    -J --job-name <jobname>
``` 
We can indicate these options by using the #SBATCH word following by any of these flag (e.g -c 10 ; means 10 CPUs). 
The following is a generic example of a SLRUM script for BLAST analysis:

```bash
#!/bin/bash

## Job name:
#SBATCH --job-name=Blast
#
## Wall time limit:
#SBATCH --time=00:00:00
#
## Project:
#SBATCH --account=nn9055k
## Other parameters:
#SBATCH --cpus-per-task 12
#SBATCH --mem=60G
#SBATCH --nodes 1
## Reserve space in local disk for faster computation
#SBATCH --gres=localscratch:150G


######Everything below this are the job instructions######

## Set up job environment:
set -o errexit  # Exit the script on any error
set -o nounset  # Treat any unset variables as an error

module --quiet purge  # Reset the modules to the system default
module load Anaconda3/2019.03
BLAST+/2.11.0-gompi-2020b

##Actuvate conda environments
export PS1=\$
source ${EBROOTANACONDA3}/etc/profile.d/conda.sh
conda deactivate &>/dev/null


##Useful lines to know where and when the job starts

echo "I am running on:"
echo $SLURM_NODELIST   ##The node where the job is executed
echo "I am running with:"
echo $SLURM_CPUS_ON_NODE "cpus"  ###The number of cpus
echo "Today is:"
date

blastp -query $i -db $line -max_target_seqs 1 -dbsize 100000000 -num_threads 10 -outfmt 6 > $i.out



```
**Remember always to add the ```--account=nn9055k``` to the script**


To submit the job:

```bash
(base) [auve@login-3.SAGA auve]$ sbatch myblast.sh
```

**3.2 Monitorate the job progress**

A user can monitorate the status of the Job by the command ```squeue``` :

```bash
(base) [auve@login-3.SAGA auve]$ squeue |head
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           2790926    bigmem JHI1c_00   jhi023 PD       0:00      1 (Resources)
           2693856     accel  umb_sam manuelca PD       0:00      1 (Resources)
           2796035     accel Centaurs  brasser PD       0:00      1 (Priority)
           2693857     accel   umb_eq manuelca PD       0:00      1 (Priority)
           2797661     accel epvtest1 erlingpv PD       0:00      1 (Priority)
           2797962     accel goexplor vegarbjo PD       0:00      1 (Priority)
           2693859     accel   umb_eq manuelca PD       0:00      1 (Priority)
           2693861     accel   umb_eq manuelca PD       0:00      1 (Priority)
           2799386     accel   in5550  myrthel PD       0:00      1 (Priority)
  ```



