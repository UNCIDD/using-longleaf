# using-longleaf

* Open On-demand link: https://ondemand.rc.unc.edu
* 

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
* When to use Open On Demand vs command-line? It depends. Some things are only possible with the command-line, but if you are not that confident in a unix shell, you should get around with OOD.

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

## 2. Transfert files
* Via git (for code, this would be very slow for data and you’d run to GitHub quota really fast)
* Using rsync (scp is deprecated 😥). Start from:
```bash
rsync -av --prune-empty-dirs --include='*/' --include="*.rds" --exclude='*' chadi@longleaf.unc.edu:/work/users/c/h/chadi/flepiMoP/Flu_USA this/local/destination/folder
```
with `a` for recursive, `v` for verbose (view progress),  `--include/exclude=` for pattern matching (if you come from scp, way exclude/include work is different, the above gets all files ending in `.rds` from the blue longleaf folder to the green local folder). Since I used this pattern I remove the empty directory otherwise the whole folder structure is copied. Use google/chatGPT to find the good command.
* Via Open On-demand

## 2. Load modules (software environment)
* See all modules available for you to load
  * `module avail` for example: module avail python
  * `module keyword`
  * `module spider`
* Add a module to my session:   `module load <module name>`
* See all loaded in **my** session:  `module list`
* Remove module from session: `module rm <module name>`
* Save current session's list of modules to load automatically every time I log in and for every job: `module save`


## 3.
Run the job:
```batch
sbatch my_batch_script.run
```
This command return immediately, you can go and run other jobs, party and all until it finishes. But you might want to monito

## X. Slurm script

* `--mem=50g` memory limit
* `--time= (-t)`   --time= 1-22:33:44 for 1 day, 22 hours, 33 minutes and 44 seconds of timelimit
* `--partition= (-p) <partition name>`
* `--output= (-o) <filename>` where is located the file output.
    * `-o /nas/longleaf/home/<onyen>/class_jobs/myJob%j.out`
* `-J (--jobname) <name>` 
* `--ntasks (-n) <number of tasks>     -n 1`
 used for parallel (non-serial) jobs
* `-N (--nodes=)`
* `-N 1` for all longleaf jobs [dogwood allows more]


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
sacct -j $SLURM_JOB_ID --format='JobID,user,elapsed, cputime, totalCPU,MaxRSS,MaxVMSize, ncpus,NTasks,ExitCode'

```
As this job is long, I have added two sbatch command to send me an email at the start and the end of the job (you could also add `FAIL`, `ALL`, `TIME_LIMIT`, `ARRAY_TASKS`, ...).
The last `sacct` command prints a bit of diagnostic after your script.

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
Note that here, I capture the output to my program using `> out_fit256.out 2>&1` to the file `out_fit256.out`, this is useful if you want to just one file (and it works with R jobs too)

#### Job-array
A job array launches the same jobs several times. This is extremely powerful.

```bash
#!/bin/bash
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -p jlessler
#SBATCH --mem=64G
#SBATCH -t 00-01:00:00
#SBATCH --array=8-11,17
#SBATCH --gres=gpu:1

/nas/longleaf/home/chadi/.conda/envs/diffusion_torch6/bin/python -u main.py --spec_id ${SLURM_ARRAY_TASK_ID} > out_train_${SLURM_ARRAY_TASK_ID}.out 2>&1
```

This will launch 5 jobs with `${SLURM_ARRAY_TASK_ID}` taking values 8, 9, 10, 11, 17. The memory, CPU, GPU, and all are specified per batch. I run this on the ACCIDDA patron node (hence the partition: `jlessler`. 

My main.py script handles getting a number (the array_id) and executing given functions (here, it would fetch some meta-parameters combination).


##### Advanced: a job that depends on another (e.g postprocessing for several batch scripts)
```bash
$ sbatch my_array_job.sbatch
Submitted batch job 5405375
$ sbatch --dependency==afterany:5405575 postprocess.sbatch
Submitted batch job 5405376
```
will execute `postprocess.sbatch` after all the array jobs in `my_array_job.sbatch` terminate. This also work for non-array job


## Monitor and Diagnosis


### During my run

#### Status
You may either use Open On Demand (under Jobs > Active jobs), or `squeue` to print the job queue. To print only the jobs submitted by your user, type
```bash
$ squeue -u <onyen>
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          52178012  a100-gpu reproduc    onyen PD       0:00      1 (Resources)
          52178013  a100-gpu finetuni    onyen PD       0:00      1 (Priority)
          52078265  a100-gpu slurm_bs    onyen  R 2-14:58:08      1 g141606
          52077597  a100-gpu slurm_bs    onyen  R 2-15:06:04      1 g141605
          52176006  a100-gpu  jupyter    onyen  R    5:28:52      1 g141606
       52092063_28  a100-gpu estimate    onyen  R 1-17:54:50      1 g141605
       52147221_59  a100-gpu       e2    onyen  R 1-03:41:09      1 g141604
       52147221_58  a100-gpu       e2    onyen  R 1-04:07:46      1 g141604
       52147221_57  a100-gpu       e2    onyen  R 1-04:19:23      1 g141604
       52147221_55  a100-gpu       e2    onyen  R 1-04:20:29      1 g141607
```
where the JOB_ID contains the job ID. We see that the last 5 jobs here are array jobs (see below: one jobs launch several subjobs). We also see the job status:
* R: running
* PD: pending
and the time it's been running. The last column provides either the node on which a running job is currently running (e.g g141604). Note that several jobs sometimes run on the same node. And if it is in queue (PD), the reason why. If you don't see your job in `squeue`, it means that it has terminated.


#### Logs
If you have  regular prints/progress bars and all, this command prints the log file (either the default slurm_JOBID.out or the redirected one) for your command and updates it live:
```bash
tail -f your_log.out
```
This is where you’ll see any error. Note that this command only shows the live change, sometime you might want to show the full log (e.g if the process has been terminated) using:
```bash
cat your_log.out
```

### After my job is done
run seff (only after the job is finished)
```bash
$ seff 36460183
Job ID: 36460183
Cluster: longleaf
User/Group: chadi/users
State: CANCELLED (exit code 0)
Nodes: 1
Cores per node: 256
CPU Utilized: 16-12:57:33
CPU Efficiency: 21.88% of 75-14:24:00 core-walltime
Job Wall-clock time: 07:05:15
Memory Utilized: 950.36 GB
Memory Efficiency: 127.56% of 745.00 GB
```
Here we see that this job has been memory constrained so the CPU time was not efficiently utilized, and should be re-run with more memory



#### Quality of life things
Get an

### Appendix A: command-line
* Get around
  * `cd /relative/or/absolute/path/to/dir/` go to a specific directory
  * `cd ..` go back one directory
  * `cd` without arguments bring you to home (sometimes)
  * `pwd` tells you where you are
 

### Advanced
There are limits on the number of resources a single person or job can request. Some of the limits are set at the user level (across all their jobs), most are set at the partition (and "qos") levels.
```bash
scontrol show partition general
sacctmgr show qos format=name%15,mintres,grptres,maxtres%20,maxtrespernode,maxtrespu%20,maxjobs,mintres,MaxSubmitJobsPerUser,maxtrespa
sacctmgr show user where name=<ONYEN> withassoc format=user,DefaultQOS,account%20,qos
```

### Get help
Please don’t waste too much time, these problems are hard when you’ve not been introduced to them. Ask to the:
* `#computing channel` on the UNC-IDD Slack
* research@unc.edu (business hours, M-F)

Both are very helpful if you provide enough details! Always provide the full error message + your onyen and the job ID if known.
SLURM is universal, so Google and LLMs can help







