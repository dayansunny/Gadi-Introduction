# Getting started with Gadi
 
Notes and example scripts from the NCI "Getting started with Gadi" seminar (24 November 2026). I wrote this as a beginner's reference for myself. Check the [Gadi User Guide](https://opus.nci.org.au/spaces/Help/pages/230621203/Gadi+User+Guide) for anything that matters, since queue limits and policies change.
 
Throughout, `USERNAME` is your NCI username and `PROJECT` is a project code you belong to (run `groups` to list yours).
 
## Logging in
 
```
ssh USERNAME@gadi.nci.org.au
```
 
The first connection asks you to confirm the host key, so type `yes`. Login is round-robin, so you land on a different login node each time. Your files are the same on all of them.
 
For large transfers use the data mover host instead:
 
```
scp -p file.nc USERNAME@gadi-dm.nci.org.au:/scratch/PROJECT/USERNAME/
scp -p USERNAME@gadi-dm.nci.org.au:/g/data/PROJECT/USERNAME/file.pdf ~/Downloads/
```
 
Run `scp` from your own machine, not from inside a Gadi session.
 
## Login nodes versus compute nodes
 
Login nodes are for editing files, building software, `git clone`, small transfers, and submitting jobs. Short test runs are fine, but each process is limited to 30 minutes of CPU time and 4 GiB of memory. Anything heavier goes in a PBS job.
 
Compute nodes have no internet access. Download everything a job needs (data, repositories, pip packages) before you submit it.
 
## Storage
 
| Area | Path | Notes |
|---|---|---|
| home | `/home/<institution code>/USERNAME` | 10 GiB, backed up |
| scratch | `/scratch/PROJECT/USERNAME` | Fast, temporary, not backed up. Files not accessed for 100 days are removed |
| global data | `/g/data/PROJECT/USERNAME` | Long-term project storage, not backed up |
| massdata | `mdss -P PROJECT ls -l` | Tape archive, accessed with `mdss` |
| applications | `/apps/...` | Software installed by NCI, read-only |
 
Quotas belong to the project and are shared by everyone in it. There is also a limit on the number of files (inodes), separate from the size limit.
 
Useful checks:
 
```
quota -s                       # home
lquota                         # scratch and gdata for all your projects
nci_account -P PROJECT         # compute allocation and storage
nci-files-report -p PROJECT -f scratch
nci-file-expiry list-quarantined
```
 
Deleted files on scratch and `/g/data` cannot be recovered. Home has hourly, daily and weekly snapshots in `~/.snapshot`.
 
## Software
 
```
module avail python3
module purge
module load python3/3.12.13
module list
```
 
Start from a clean environment (`module purge`) and always load a specific version, not the default.
 
For Python packages, make a virtual environment on a login node, somewhere with room (scratch or `/g/data`, not home):
 
```
python3 -m venv myenv
source myenv/bin/activate
pip install pandas
```
 
See `scripts/setup_venv.sh`.
 
## Running jobs
 
```
qsub job.sh            # submit
qstat -u USERNAME      # your jobs
qstat -u USERNAME -Esw # why is my job waiting?
qcat JOBID             # output so far
qdel JOBID             # cancel
```
 
Interactive session for testing:
 
```
qsub -I -P PROJECT -q normal -l walltime=00:30:00,ncpus=1,mem=4GB,storage=scratch/PROJECT
```
 
Example scripts:
 
- `scripts/basic_job.sh`: a minimal batch job
- `scripts/copyq_download.sh`: a long download in the `copyq` queue (the queue with internet access)
If a job needs `/g/data` or another project's scratch, list it in `#PBS -l storage=`, joined with `+`, for example `gdata/PROJECT+scratch/PROJECT`.
 
## Charging
 
One service unit (SU) is 30 minutes of one CPU in the `normal` queue. The charge is:
 
```
rate x max(CPUs, memory units) x hours
```
 
A memory unit is 4 GiB in `normal` and `copyq`, so a 1-CPU job asking for 12 GiB is billed as 3 CPUs. `express` costs three times as much as `normal`. If a project runs out of SU, new jobs wait until it gets more. Compute allocations are quarterly.
 
Check a running job's efficiency with `nqstat_anu JOBID` and ask for less memory next time if you are well under what you requested.
 
## When something goes wrong
 
- Job waits too long: not enough free CPUs, not enough SU, a project in `storage=` is over quota, or it is waiting for a licence.
- Job fails: read the `.e` and `.o` files. "No such file or directory" usually means a wrong path or a missing `storage=` entry. Also check memory and walltime against what the job used.
- Disk quota errors: `quota`, `lquota`, `nci-files-report`.
## Getting help
 
Email help@nci.org.au with your username, project code, and for job problems the job ID or the full paths to the script and the error and output files. Avoid attachments (screenshots are fine). For more compute or `/g/data` space, talk to your project's chief investigator first.
 
## Account
 
The NCI account is for life. Keep your contact details current and recertify once a year (new password and accepting the conditions of use). If you miss it the account is suspended for 120 days, then deactivated, and the helpdesk can revive it.
 
