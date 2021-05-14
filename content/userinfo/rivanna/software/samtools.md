+++
type = "rivanna"
categories = [
  "HPC",
  "software",
  "bio"
]
date = "2021-05-14T08:37:46-05:00"
tags = [
  "multi-core",
]
draft = false
modulename = "samtools"
softwarename = "Samtools"
title = "Samtools on Rivanna"
author = "RC Staff"
+++

# Description
{{% module-description %}}
<br>
**Software Category:** {{% module-category %}}

For detailed information, visit the [{{% software-name %}} website]({{< module-homepage >}}).

# Available Versions
To find the available versions and learn how to load them, run:
```
module spider {{% module-name %}}
```

The output of the command shows the available {{% software-name %}} module versions.

For detailed information about a particular {{% software-name %}} module, including how to load the module, run the `module spider` command with the module's full version label. __For example__:
```
module spider {{% module-firstversion %}}
```

{{< module-versions >}}

# Build Your Own Version
Users may build their own versions of {{% software-name %}} if they wish to use a different compiler or software version. Instructions are available on the [{{% software-name %}} website]({{< module-homepage >}}).

# Convert BAM to SAM with Samtools
`samtools view` can convert human-readable `SAM` files to binary `BAM` files. Below is a simple command to convert `SAM` files to `BAM` files. Th `-S` option specifies that the input is in SAM format, while the `-b` option outputs to a BAM file:
```
samtools view -bS example.sam > example.bam
```

To preview the first five lines of the new `BAM` file:
```
samtools view example.bam | head
```

Most downstream analyses require your `BAM` files to be sorted, which can be achieved by:
```
samtools sort example.bam -o example_sorted.bam
```

If you would like to visualize your `BAM` file using some viewer like IGV, you will need to create an index file
```
samtools index example_sorted.bam
```

Finally, `samtools flagstat` is a good way to get simple statistics from a `BAM` file including QC, duplicates, mapped reads, and many others
```
samtools flagstat example_sorted.bam
```

# SLURM Script Example
To run {{% software-name %}} on Rivanna, a script similar to the following can be used.

```
#!/bin/bash
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -t 1:00:00
#SBATCH -p standard
#SBATCH -A mygroup

module purge
module load samtools

samtools view -bS example.sam > example.bam
```

To speed up your code, use multiple cpus per task. Here, we ask for 8 with the `--cpus-per-task` option, but only specify 7 in our `samtools` command to leave one for the manager process:
```
#!/bin/bash
#SBATCH -N 1
#SBATCH -n 1
#SBATCH --cpus-per-task=8
#SBATCH -t 1:00:00
#SBATCH -p standard
#SBATCH -A mygroup

module purge
module load samtools

samtools view -bS -@ 7 example.sam > example.bam
```
<br>
