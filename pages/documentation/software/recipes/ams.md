---
title: AMS
tags: [Software, Module]
---
The [Amsterdam Modeling Suite](https://www.scm.com){:target=_blank} is used by academic and
industrial researchers in all areas of chemistry, materials science, and
engineering.

Various versions are available, this SLURM sbatch file is known to work
for `ams/2022.101`:

```shell
#!/bin/bash

#SBATCH --job-name=test_ams
#SBATCH --output=test_ams-%j.out 
#SBATCH --error=test_ams-%j.err
#SBATCH --partition=tue.default.q
#SBATCH --nodes=1
#SBATCH --ntasks=16
#SBATCH --time=1-00:00:00

module purge
module load slurm
module load foss/2022a
module load ams/2022.101

<AMS CODE HERE>
```
On some nodes, AMS is not running due to the processor type. Adding the following option the the AMS startup command might help:
```shell
-C "amd|haswell|cascadelake|broadwell"
```
