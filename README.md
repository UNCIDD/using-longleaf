# using-longleaf

Follow the steps listed on [Request a Cluster Account](https://help.rc.unc.edu/request-a-cluster-account) page and select Longleaf Cluster under subscription type. You will receive an email notification once your account has been created.

## A. Storage on longleaf
### home space
* Your home directory will be in `/nas/longleaf/home/<onyen>`
* backed up
* quota: 50 GB soft limit and a 75 GB hard limit ⚠️

### `/users/<o>/<n>/<onyen>` storage
* Capacity expansion to your home directory.
* OK to compute against, however as IO increases, consider copying or moving to /work for processing
* OK to hold inactive data sets like a near-line archive
* 10 TB quota per user
### `/work/users/<o>/<n>/<onyen>` storage
* built for high-throughput and data-intensive computing (very fast)
* for data **actively being computed on and processed**
* NOT intended to be a personal storage location
* Be fair and don't store much (in other university, there would be an auto-delete policy)
* 10 TB quota per user.

### `/proj` storage Proj Space
* equivalent to home directory for a team 
* `/proj` space is available to PIs (only, email research@unc.edu) upon request.
* Shared datasets, code, ...

## 1. Connect to longleaf
```bash
ssh <onyen>@longleaf.unc.edu
```




