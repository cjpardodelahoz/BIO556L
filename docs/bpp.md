# Joint Species Tree Estimation and Species Delimitation with BPP

## Contents

* **[Inferring a concatenated tree with maximum likelihood](#ML)**
* **[Inferring a summary species tree with ASTRAL](#AS)**

This lab is based on a tutorial by the BPP developers that was published as [chapter](https://hal.inria.fr/PGE/hal-02536475) on the book [Phylogenetics in the Genomic Era](https://hal.inria.fr/PGE/hal-02535070v1) (which I highly recommend). The book chapter has a thorough description of the steps and configurations which we will go over in class but I will not repeat them here. I  only made sligth modifications to accomodate it for our cluster.

Check out the [BPP GitHub repository](https://github.com/bpp/bpp) for more resources on this tool, including publications and software manual.


<a name="ML"></a>
## Inferring a concatenated tree with maximum likelihood


### Getting the data and preparing the directory structure

We'll log into to the DCC and download the data that we will use, which consists of sequence alignments of two nuclear loci from the genus *Phrynosoma* of horned lizards sampled in western North America ().

```sh
ssh YOURNETID@dcc-login.oit.duke.edu
cd /hpc/group/bio556l-s23/YOURNETID
# Make directory for tutorial data
mkdir -p week13/data
cd week13/data
# Download the data
wget http://abacus.gene.ucl.ac.uk/ziheng/data/HornedLizardsData.tgz
tar -xvzf HornedLizardsData.tgz
# 
cd HornedLizardsData
mkdir -p A11/r1 A11/r2
```

### Joint species delimitation and species tree inference

We have to examine the `lizards.bpp.A11.ctl` file and change the unmber of threads to 1 so it can run on the DCC. Open the file with the editor and set `threads = 1 1 1`, then save.

```sh
nano lizards.bpp.A11.ctl
```

Request reseources for an interactive session:
```sh
srun -c 1 --mem=8G -p scavenger --pty bash -i
```

Let's copy the modified ctl file to each of the run directories and run two independent BPP analyses with a different seeds:

```sh
# Copy ctl files
cp lizards.bpp.A11.ctl A11/r1
cp lizards.bpp.A11.ctl A11/r2
# Put BPP on PATH
export PATH=/hpc/group/bio556l-s23/apps/bpp-4.6.2-linux-x86_64/bin:${PATH}
# Start run 1
cd A11/r1 
bpp --cfile lizards.bpp.A11.ctl
# Start run 2 
cd ../r2 
bpp --cfile lizards.bpp.A11.ctl
```