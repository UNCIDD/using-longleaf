# using-longleaf

## Workflow
### Checklist for submitting a slurm job
- [ ] Do you know where are your data files on Longleaf (name and paths)? 
- [ ] Do you know where is your app’s script  on Longleaf (name and path)?
- [ ] Test that you have the correct modules loaded
- [ ] Test that your app works locally (on a small data sample if necessary)
- [ ] Test your app’s command by typing it and hitting Crtl^C a few SECONDS later.  Fix typos and not found errors.
- [ ] Add checkpoints and print to your code to see movement while it's running, or do intermediate files.
- [ ] Include full path references to files and scripts in your SLURM (e.g.: /nas/longleaf/home/myonyen/scripts/abc.py)
- [ ] Use `seff <job_id>` command on an old jobs (yours or labmate’s)
- [ ] Once you type `sbatch …`  Make a note of job id and description of what it is doing (vs your other jobs).
- [ ] Last step, efficiency! Type: `seff <job id>` to inform next time best memory and time settings.
- [ ] Tailor SLURM for next round of jobs based on what memory/time previously successful jobs actually used/needed.

### Tips
* Keep each project in its own folder/set of folders.
* It’s easier if your file names don’t contain a space, dot or `()`.
* Do not put big data file`s your home directory; put those on scratch or proj.
* If you think you’ll want to share files with a labmate, keep all files for  that work in your PI’s `/proj/` space where all labmates 
    can see each other’s files.
* Every resource on a cluster is shared (storage, compute, ...) so be mindful of others and follow best practices.
* Only ask for resources you need in your SLURM  more resources = longer wait

## A. Create an account

Follow the steps listed on the [Request a Cluster Account](https://help.rc.unc.edu/request-a-cluster-account) page and select Longleaf Cluster under subscription type. You will receive an email notification once your account has been created.

## B. Storage on longleaf
For an occasional user, start working from the `home` directory and move later to `users` if you run against your quota. If you anticipate using Longleaf for several projects, start by getting set up on `users`.

### `home` space
* Your home directory will be in `/nas/longleaf/home/<onyen>`
* backed up
* quota: 50 GB soft limit and a 75 GB hard limit ⚠️

### `/users/<o>/<n>/<onyen>` storage
* Capacity expansion to your home directory.
* OK to compute against, however as IO increases, consider copying or moving to /work for processing
* OK to hold inactive data sets like a near-line archive
* 10 TB quota per user
### `/work/users/<o>/<n>/<onyen>` storage
* Built for high-throughput and data-intensive computing (very fast)
* for data **actively being computed on and processed**
* NOT intended to be a personal storage location
* Be fair and don't store much (in other universities, there would be an auto-delete policy)
* 10 TB quota per user.

### `/proj` storage Proj Space  
* equivalent to home directory for a team 
* `/proj` space is available to PIs (only, email research@unc.edu) upon request.
* Shared datasets, code, ...

## 1. Connect to longleaf
Either from the UNC campus network, or via VPN if you are not on campus (For VPN instructions, search [the UNC help](https://help.unc.edu) for VPN.

```bash
ssh <onyen>@longleaf.unc.edu
```
This gets you to your home folder (/nas/longleaf/home/<onyen>) on a login node


## 2. Load modules (software environment)
* See all modules available for you to load
  * `module avail` for example: module avail python
  * `module keyword`
  * `module spider`
* Add a module to my session:   `module load <module name>`
* See all loaded in **my** session:  `module list`
* Remove module from session: `module rm <module name>`
* Save current session's list of modules to load automatically every time I log in and for every job: `module save`



## X. Slurm script

### Examples (copy these and adapt)
#### Single cpu R job, general partition, 
7-day runtime limit, 10 GB memory limit
```bash
#!/bin/bash
#SBATCH -p general
#SBATCH -N 1
#SBATCH -t 07-00:00:00
#SBATCH --mem=10g
#SBATCH -n 1
#SBATCH --mail-type=BEGIN, END
#SBATCH --mail-user=YOURONYEN@email.unc.edu

module purge
module load r/4.2.1

Rscript mycode.R
```
As this job is long, I have added two sbatch command to send me an email at the start and the end of the job (you could also add `FAIL`, `ALL`, `TIME_LIMIT`, `ARRAY_TASKS`, ...)

#### Multi-core R job
Submits a 24-core, single-node R job. Here your R script needs to handle the parallelization itself (e.g. because it calls stan, or `doParallel`)
general partition, 2-hour runtime limit, 3 GB memory limit.
```bash
#!/bin/bash
#SBATCH -p general
#SBATCH -N 1
#SBATCH -t 02:00:00
#SBATCH --mem=3g
#SBATCH -n 24
module purge
module load r/4.2.1
Rscript mycode.R
```

if you need more memory, replace `#SBATCH --mem=3g` by e.g:
```bash
#SBATCH --qos=bigmem_access 
#SBATCH --mem=500g
```
#### example flepiMoP job
1 node (N) 1 task (n) to the general partition  for 2 days and 20 hours, with this task needing 256 cpus
```bash
#!/bin/bash
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -p general
#SBATCH --mem=450g
#SBATCH -c 256
#SBATCH -t 02-20:00:00
module purge
flepimop-calibrate -c config_SMH_Flu_2024_R1_allflu_medVax_H3_training_monthly_emcee.yml --nwalkers 1000 --jobs 500 --niterations 5000 --nsamples 250 --id SMH_Flu_2024_R1_allflu_medVax_H3_training_monthly_emcee-20241004_120000-BIG  > out_fit256.out 2>&1
```

#### Job-array
% sbatch job1.sbatchSubmitted batch job 5405575% sbatch --dependency=after:5405575 job2.sbatchSubmitted batch job 5405576Other options:sbatch --dependency=after:5405575 job2.sbatchsbatch --dependency=afterany:5405575 job2.sbatch<img width="402" alt="image" src="https://github.com/user-attachments/assets/f166148f-75cc-4ed9-be33-795b64052488">



#### Quality of life things
Get an

### Appendix A: command-line
* Get around
  * `cd /relative/or/absolute/path/to/dir/` go to a specific directory
  * `cd ..` go back one directory
  * `cd` without arguments bring you to home (sometimes)
  * `pwd` tells you where you are
 





