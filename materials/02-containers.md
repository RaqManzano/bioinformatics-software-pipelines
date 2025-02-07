---
title: "Container Virtualisation"
---

## Software Containers

Software containerization is a way to package software and its dependencies in a single file. 
A software container can be thought of as a small virtual machine, with everything needed to run that software stored inside that file. 
Software containers ensure reproducibility, allowing the same analysis to run on different systems. 
They can run on a local computer or on a high-performance computing cluster, producing the same result.
The software within a container is isolated from other software, addressing the issue of incompatible dependencies between tools (similarly to Mamba environments).

For these reasons, software containerisation solutions, such as [_Docker_](https://www.docker.com/) and [_Singularity_](https://docs.sylabs.io/guides/latest/user-guide/), are widely used in bioinformatics.
While these two container software solutions share many similarities, we will focus our attention on _Singularity_, as it is more widely used in HPC cluster systems (but it can also be used on a regular computer). 
However, it's worth keeping in mind that images created with Docker can be compatible with Singularity and _vice versa_.

:::{.callout-note collapse=true}
#### Docker vs singularity

There are some key differences between Docker containers and Singularity containers. 
The most important being the necessary *permission level* of the containers. 
Docker containers run as root (admin) by default, which means that they have full access to the host system. 
While this can be advantageous in some cases, it can also pose security risks, particularly in multi-user environments. 
Singularity, on the other hand, runs containers as non-root users by default, which can improve security and prevent unauthorized access to the host system. 
Singularity is specifically designed for use in HPC environments and can run on a wide variety of platforms and systems without root access.

**TL;TR:**

- Docker is well-suited for building and distributing software across different platforms and operating systems.
- Singularity is specifically designed for use in HPC environments and can provide improved security and performance in those settings.
:::


### Singularity installation

Typically, Singularity is pre-installed on HPC servers by the system administrators, and **we recommend that you use the version installed by your system admins**. 

Although it is possible to install it yourself (e.g. with Mamba), we have found this to be a less reliable way to setup _Singularity_ on a HPC.
This is because it requires further configuration to interact with the filesystem.


### Singularity images

Although you can build your own Singularity images, for many popular software there are already pre-built images available from public repositories. 
Some popular ones are: 

- [depot.galaxyproject.org](https://depot.galaxyproject.org/singularity/)
- [Sylabs](https://cloud.sylabs.io/)

For example, let's consider the [SeqKit program](https://bioinf.shenwei.me/seqkit/), which is a toolkit for manipulating FASTA/Q files. 
If we search on either of those websites, we will see this software is available on both. 
In this case, the version on Sylabs ([here](https://cloud.sylabs.io/library/bhargava-morampalli/containers/seqkit)) is older than the one on the Galaxy server (at the time of writing we have 2.8.0 available). 

Therefore, let's consider the file on the Galaxy server.
First, go to [depot.galaxyproject.org](https://depot.galaxyproject.org/singularity/) and search for the software of interest (use <kbd>Ctrl</kbd> + <kbd>F</kbd> to find the text of interest). 
When you find the software and version of interest, right-click the file and click "Copy Link". 
Then use that link with the `singularity pull` command: 

```bash
# create a directory for our singularity images
mkdir images

# download the image
singularity pull images/seqkit-2.8.0.sif https://depot.galaxyproject.org/singularity/seqkit%3A2.8.0--h9ee0642_0
```

Here, we are saving the image file as `seqkit-2.8.0.sif` (`.sif` is the standard extension for singularity images). 
Once we have this image available, we are ready to run the software, which will see in practice with the exercise below. 


### Exercises

:::{.callout-exercise}

To illustrate the use of Singularity, we will use the `seqkit` software to extract some basic statistics from the sequencing files in the `rnaseq/reads` directory. 
If you haven't done so already, first download the container image with the commands shown above. 

The way to run a command within a singularity container is: 

```bash
singularity run PATH-TO-IMAGE YOUR COMMANDS HERE
```

- Write a command to run `seqkit stats reads/*.fastq.gz` using the singularity image we downloaded earlier.


:::{.callout-answer}
The Singularity command is: 

```bash
singularity run --pwd . images/seqkit-2.8.0.sif seqkit stats reads/*.fastq.gz
```

If we run this, it produces an output like this: 

```
file                                     format  type   num_seqs      sum_len  min_len  avg_len  max_len
reads/SRR7657872_1.downsampled.fastq.gz  FASTQ   DNA   1,465,993  219,898,950      150      150      150
reads/SRR7657872_2.downsampled.fastq.gz  FASTQ   DNA   1,465,993  219,898,950      150      150      150
reads/SRR7657874_1.downsampled.fastq.gz  FASTQ   DNA   1,379,595  206,939,250      150      150      150
reads/SRR7657874_2.downsampled.fastq.gz  FASTQ   DNA   1,379,595  206,939,250      150      150      150
reads/SRR7657876_1.downsampled.fastq.gz  FASTQ   DNA   1,555,049  233,257,350      150      150      150
reads/SRR7657876_2.downsampled.fastq.gz  FASTQ   DNA   1,555,049  233,257,350      150      150      150
reads/SRR7657877_1.downsampled.fastq.gz  FASTQ   DNA   1,663,432  249,514,800      150      150      150
reads/SRR7657877_2.downsampled.fastq.gz  FASTQ   DNA   1,663,432  249,514,800      150      150      150
```

:::
:::
