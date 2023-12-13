---
date: 2023-12-13
categories: [Umbrella HPC Cluster, Maintenance]
authors: [e.loomeijer@tue.nl, a.van.hoof@tue.nl, a.c.m.bertens@tue.nl]
---

# 2024 Q1 Umbrella HPC Cluster maintenance

TU/e Umbrella HPC Cluster has a scheduled downtime for maintenance from **Monday the 12th 09:00 CET** to **Wednesday the 14th 17:00 CET** of **February 2024**. The cluster will be unavailable during this time. Please make sure that your jobs are finished before the start of the maintenance or that they are able to continue after they were (hard) killed/cancelled.

Please read further for more information about the maintenance and expected changes.

!!! danger

    All running Jobs on Monday 12 February 2024 09:00 will be cancelled!

!!! warning

    The Operating System (OS) in the cluster will be changed from CentOS Linux 7 to Rocky Linux 8.

<!-- more -->

## What will happen?

- OS Upgrade/Replacement: The current OS running in the cluster is CentOS Linux 7 which will reach end of life (EOL) on June 30, 2024. It is therefore decided to upgrade the OS to [Rocky Linux 8](https://rockylinux.org/){:target=_blank}. To upgrade the OS there can be no running jobs on the cluster and all nodes need to reboot.
- Cluster Manager Replacement: The Current Cluster Manager Bright Cluster Manager will be replace by TrinityX. To upgrade this software there can be no running jobs on the cluster and all nodes need to reboot.
- Maintenance: Upgrade of the Cluster Scheduler (SLURM). The Scheduler (SLURM) has a newer version available that fix bugs and has additional functionality and features.
- Security: Firmware Updates (hardware)

Because a part of the nodes (especially the Login and Storage Nodes) have been continuously running for a long time, their Operating System and Firmware(s) are out-of-date and need a refresh. OS and firmware upgrades will address security issues, for nodes that are accessible from the TU/e Campus directly this is an urgent issue. For this the nodes need to reboot without any jobs running on those nodes.

## What is the impact?

After the Upgrade/Replacement of the OS from CentOS7 to Rocky8 it is not guaranteed that you applications are still able to run, especially self compiled C/C++/FORTRAN code can have dependencies on the OS that are no longer valid.

- This maintenance involves all Nodes for all departments being part of the TU/e HPC Umbrella Cluster. Head Nodes, Login Nodes, Storage Nodes, Compute Nodes and GPU Nodes. See also [https://supercomputing.tue.nl/documentation/specifications/](../../documentation/specifications.md)
- During the Downtime/Maintenance no Compute Jobs will be running nor can be started or scheduled.
- All running Jobs on Monday 12 February 2024 09:00 will be cancelled.
- All login nodes will be rebooted killing any processes running and the nodes are unavailable during maintenance

## What do you need to do?

- Make sure your jobs are finished before Monday 12 February 2024 or that they are able to continue after they were (hard) killed/cancelled.
- Do not try to use the cluster in any way during the maintenance window.
- Check if the applications/code that you are using is still able to execute on the new OS after the maintenance.

!!! danger

    Data (incl. home directories) in the HPC Cluster is NOT backed up. The HPC Cluster is not a solution for archiving your work!

    You are FULLY responsible for your own data management!

## Questions?

For questions and remarks please contact [hpcsupport@tue.nl](mailto:hpcsupport@tue.nl)
