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




