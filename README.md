# using-longleaf

Follow the steps listed on [Request a Cluster Account](https://help.rc.unc.edu/request-a-cluster-account) page and select Longleaf Cluster under subscription type. You will receive an email notification once your account has been created.

## A. Storage on longleaf
### home space
Your home directory will be in `/nas/longleaf/home/<onyen>` and is backed up via snapshots.
Your home directory has a quota which you will want to monitor occasionally: 50 GB soft limit and a 75 GB hard limit.

### `/users/<o>/<n>/<onyen>` storage
Think of it as a capacity expansion to your home directory.
High capacity storage
OK to compute against, however as IO increases, consider copying or moving to /work for processing
OK to hold inactive data sets like a near-line archive

10 TB quota 
### `/work/users/<o>/<n>/<onyen>` storage
* built for high-throughput and data-intensive computing, and intended for data actively being computed on, accessed and processed by systems such as Longleaf
* NOT intended to be a personal storage location; /work is for data actively being processed with high IO requirements
* There is a 10 TB quota per user.

### `/proj` storage Proj Space
“/proj” space is available to PIs (only, email research@unc.edu) upon request. An home directory for a team


## 1. Connect to longleaf
```bash
ssh <onyen>@longleaf.unc.edu
```




