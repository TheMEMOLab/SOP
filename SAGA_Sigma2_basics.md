# SOP - Basic usage of SAGA Sigma2 high performance computing (HPC) cluster for the MEMO group users.

v1.1, August 27, 2021
Author: Arturo Vera-Ponce de Leon & Carl M. Kobel

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
#SBATCH --account=nn9864k
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
module load Anaconda3/2019.03 #Load Anaconda 
module load BLAST+/2.11.0-gompi-2020b #Load blast

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
**Remember always to add the ```--account=nn9864k``` to the scripts**


To submit the job:

```bash
(base) [auve@login-3.SAGA auve]$ sbatch myblast.sh
```

**3.2 Monitorate the job progress**

A user can monitorate the status of the Job by the command ```squeue``` :

```bash
(base) [auve@login-3.SAGA auve]$ squeue
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

To check the status of all the jobs from a single user you can type:

```bash
base) [auve@login-3.SAGA auve]$ squeue -u $USER
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```

### 4. Conda environments 

**4.1 Create a specific conda environment:**

If specific software is not installed in SAGA, the easiest way to install most of the bioinformatics software is by creating Conda environments. 
The Sigma2 documentation fully describes how to do this [here](https://documentation.sigma2.no/software/userinstallsw/python.html). However, the following is a brief summary of how to create an envrioment in SAGA.

1. Load the miniconda3 module and activate conda

```bash
[auve@login-3.SAGA nn9864k]$ module load Miniconda3/4.9.2
[auve@login-3.SAGA nn9864k]$ conda activate
(base) [auve@login-3.SAGA nn9864k]$
```

2. Create an evironment and install software.

*Let's use as example the instalation of the taxonomy classification tool GTDBTk:*

The following command will create a Conda environment named: GTDBTK-1.5.0 and install on it all the dependencies to run gtdbtk tool.

```bash
$ conda create -y --prefix /cluster/projects/nn9864k/shared/condaenvironments/GTDBTK-1.5.0 -c conda-forge -c bioconda gtdbtk=1.5.0
```
If everything is OK it will start running and displaying something like this:
```bash
Collecting package metadata (current_repodata.json): |done
Solving environment: failed with repodata from current_repodata.json, will retry with next repodata source.
Collecting package metadata (repodata.json): done
Solving environment: done
## Package Plan ##

  environment location: /cluster/projects/nn9864k/shared/condaenvironments/GTDBTK-1.5.0

  added / updated specs:
    - gtdbtk=1.5.0


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    colorama-0.4.4             |     pyh9f0ad1d_0          18 KB  conda-forge
    dendropy-4.5.2             |     pyh3252c3a_0         308 KB  bioconda
dendropy-4.5.2       | 308 KB    | ################################################################################################################################################### | 100% 
zstd-1.4.9           | 431 KB    | ################################################################################################################################################### | 100% 
lz4-c-1.9.3          | 179 KB    | ################################################################################################################################################### | 100% 
Preparing transaction: done
Verifying transaction: done
Executing transaction: - b'\n     GTDB-Tk v1.5.0 requires ~40G of external data which needs to be downloaded\n     and unarchived. This can be done automatically, or manually:\n\n     1. Run the command download-db.sh to automatically download to:\n        /cluster/projects/nn9864k/shared/condaenvironments/GTDBTK-1.5.0/share/gtdbtk-1.5.0/db/\n\n     2. Manually download the latest reference data:\n        https://github.com/Ecogenomics/GTDBTk#gtdb-tk-reference-data\n\n     2b. Set the GTDBTK_DATA_PATH environment variable in the file:\n         /cluster/projects/nn9864k/shared/condaenvironments/GTDBTK-1.5.0/etc/conda/activate.d\n\n\n'
done
#
# To activate this environment, use
#
#     $ conda activate /cluster/projects/nn9864k/shared/condaenvironments/GTDBTK-1.5.0
```
*Explaining the command:
conda create: Create a conda environent
-y : Do not ask for confirmation.
--prefix: Full path to environment location (i.e. prefix).
-c : Additional channel to search for packages.*

You can either install the conda environments in this shared location ```/cluster/projects/nn9864k/shared/condaenvironments/GTDBTK-1.5.0``` or in your personal ```nn9864k/$USER``` folder. The idea of the shared folder is to create a compendium of multiple bioinformatic tools many people in the MEMO group use and not to install them every time a user needs one. 

3. Activate the environment:

```bash
(base) [auve@login-3.SAGA auve]$ conda activate /cluster/projects/nn9864k/shared/condaenvironments/GTDBTK-1.5.0
(/cluster/projects/nn9864k/shared/condaenvironments/GTDBTK-1.5.0) [auve@login-3.SAGA auve]$
```


4. Test the software by displaying the help:

```bash
(/cluster/projects/nn9864k/shared/condaenvironments/GTDBTK-1.5.0) [auve@login-1.SAGA auve]$ gtdbtk --help

              ...::: GTDB-Tk v1.5.0 :::...

  Workflows:
    classify_wf -> Classify genomes by placement in GTDB reference tree
                     (identify -> align -> classify)
    de_novo_wf  -> Infer de novo tree and decorate with GTDB taxonomy
                     (identify -> align -> infer -> root -> decorate)

  Methods:
    identify -> Identify marker genes in genome
    align    -> Create multiple sequence alignment
    classify -> Determine taxonomic classification of genomes
    infer    -> Infer tree from multiple sequence alignment
    root     -> Root tree using an outgroup
    decorate -> Decorate tree with GTDB taxonomy

  Tools:
    infer_ranks -> Establish taxonomic ranks of internal nodes using RED
    ani_rep     -> Calculates ANI to GTDB representative genomes
    trim_msa    -> Trim an untrimmed MSA file based on a mask
    export_msa  -> Export the untrimmed archaeal or bacterial MSA file

  Testing:
    test          -> Validate the classify_wf pipeline with 3 archaeal genomes 
    check_install -> Verify third party programs and GTDB reference package.

  Use: gtdbtk <command> -h for command specific help

````

The software is now installed. The message shows what we need to do in order this software runs.

### 5. Optional: Mount the HPC as a "local drive" using *sshfs*

Mounting the HPC as a local drive on your personal computer, means that you can access the HPC file system directly through your local applications. The official sigma2/saga documentation doesn't offer much on how to do this, so here is a quick guide on how to set it up. Please be aware that due to the fundamental differences between NTFS and any POSIX-related file system, it is not possible to do this on a Windows based compter.



0. Optional but recommended: Install a personal ssh key set between your computer and saga https://documentation.sigma2.no/getting_started/create_ssh_keys.html?highlight=sshfs#login-via-ssh-keys 

   This will enable you to log in without manually typing a password each time.

1. On Linux and Macos you should install sshfs.

   On Macos you need [macFUSE](https://osxfuse.github.io/) as well. When installing macFUSE, be noted that you need to reboot your computer in order to activate the kernel extension.

2. Make an empty directory called saga

   ```shell
   mkdir ~/saga
   ```

3. Make an empty file named `~/mountsaga.sh` and paste the following contents into it.

   ```shell
   #!/bin/bash
   
   # Finally, mount the sshfs locally
   sshfs USERNAME@saga.sigma2.no:/cluster/home/USERNAME ~/saga \
       -o idmap=none -o uid=$(id -u),gid=$(id -g) \
       -o allow_other -o umask=077 -o follow_symlinks
   ```

​	Please change USERNAME to your user name on saga.

4. Make this script executable by running the following command:

   `chmod +x ~/mountsaga.sh`

5. Finally, mount the sshfs locally by running the above script:

   `~/mountsaga.sh`

This is the command you will use each time you want to mount the sshfs in the future.

If you wish to unmount the sshfs, you can use the following command:

   ```shell
   umount ~/saga
   ```

You can of course mount orion just the same way. Just be aware, that there is a bug in macFUSE which means that you can only mount a single sshfs in a specific directory. So if you want to mount saga and orion besides each other, you need to put one of them into a subfolder.
