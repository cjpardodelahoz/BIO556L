# Choosing Models of Sequence Evolution

## Contents

* **[ModelFinder with an upartitioned dataset](#MF)**
* **[PartitionFinder with a partitioned dataset](#MFM)**
* **[Protein models and the domains of the Tree of Life](#MADD)**

<a name="MF"></a>
## ModelFinder with an unpartitioned dataset

### Setup

We will run this first set of analyses in an interactive session on the DCC. Let's log into the cluster, navigateto to our class directory, and request resources for an interactive session. Open a terminal an type:

```sh
# Log into DCC
ssh YOURNETID@dcc-login.oit.duke.edu
# Navigate to your class directory
cd /hpc/group/bio556l-s23/YOURNETID
# Request an interactive session with 2 cpus and a total of 8Gb of RAM
srun -c 2 --mem=8G -p scavenger --pty bash -i 
```

Now, copy the the contents of the week6 directory under `source/` to your class activites folder:

```sh
cp -r ../source/week6 class_activities
```

### Inspecting the alignment

We are going to use an alignment of the minichromosome maintenance complex component 7 gene (MCM7) from species of the lichen-forming fungi family Lecanoraceae. This alignment was kindly provided by Ian Medeiros and it was used in his [paper in *Frontiers in Microbiology*](https://www.frontiersin.org/articles/10.3389/fmicb.2021.774839/full) in 2021.

Open a new terminal tab or window and pull the contents of the week6 directory into your computer so we can look at the alignment with Mesquite:

```sh
# On a terminal window in your mirrow class activites directory on your local machine
scp -r YOURNETID@dcc-login.oit.duke.edu:/hpc/group/bio556l-s23/YOURNETID/class_activities/week6 class_activities/
```
Now, open the alignment in `class_activites/week6/alignments/Lecanoraceae_v8_MCM7.nex` with Mesquite. You will see that this alignment was saved with Bird's Eye view display setting. to deactivate it, go to **Display > Bird's Eye View**. This is a protein coding gene, but the sequences also includes a non-coding region flaking 5\` end of theCDS. The non-coding regions are marked as excluded. You will also see that there is a region of the 3\` end of the CDS that is excluded. 

We need to export only the included characters. We will do it in FASTA format. Go to **File > Export ...** then select **FASTA (DNA/RNA)** as the export format  and click **_OK_**. In the **Export FASTA Options** dialog box, make sure that all boxes are unticked except for the **Include gaps** option, and set the End of line character to **Current System Default**. Save the file to `class_activites/week6/alignments/Lecanoraceae_v8_MCM7.fas`.

Now, let's push the alignment in FASTA format to the DCC:

```sh
scp class_activities/week6/alignments/Lecanoraceae_v8_MCM7.fas YOURNETID@dcc-login.oit.duke.edu:/hpc/group/bio556l-s23/YOURNETID/class_activities/week6/alignments
```

**Tip:** When specifying the target path argument in a `cp` or `scp` command, you write the path to the parent directory where you want your file or directory to be copied to (as done above). This will make the copy have the same name as the orginal. You can also make the copy have a different name by specifying the a path that includes the new name.

### Unpartitioned model selection

Go back to the terminal tab/window where you are logged in the cluster. By now you SLURM should have allocated resources for the interactive session that we requested at the beginning. 

Make a directory to store output from the model selection analysis:

```sh
mkdir class_activities/week6/model_selection
```

Load the module for IQ-tree v1.6.12 and get the help menu:

```sh
module load IQ-TREE/1.6.12 
iqtree -h
```

You should get something that looks like this:

```sh
IQ-TREE multicore version 1.6.12 for Linux 64-bit built Sep 10 2020
Developed by Bui Quang Minh, Nguyen Lam Tung, Olga Chernomor,
Heiko Schmidt, Dominik Schrempf, Michael Woodhams.

Usage: iqtree -s <alignment> [OPTIONS]
...
```

We are now going to use ModelFinder, which is implemented within IQ-Tree, to find the best substition model for the MCM7 nucleotide alignment:

```
iqtree -s class_activities/week6/alignments/Lecanoraceae_v8_MCM7.fas \
 -pre class_activities/week6/model_selection/lecanoraceae_mcm7_unpart \
 -m MF

```

The `-s` flag is usef to specify the path to the alignment. The `-pre` flag specifies the path prefix that will be used to store the output files. In this case, all output files will be written in the directory `class_activities/week6/model_selection`, and all files names will start with `lecanoraceae_mcm7_unpart`

IQ-TREE will print the log to the *STDOUT*. We can also check the output files written in the specify directory:

```sh
ls class_activities/week6/model_selection/lecanoraceae_mcm7_unpart*
```

Here, I used the path prefix followed by `*`, which is a *wildcard* character in BASH. It means *any character, any number of times*. Therefore, the command above will list all files that start witht the prefix we specified, which correspond tot he IQ-Tree outputs.

Let's look at the `.iqtree` file, which contains the IQ-tree report:

```sh
nano class_activities/week6/model_selection/lecanoraceae_mcm7_unpart.iqtree
```

<a name=MFM></a>
## PartitionFinder with a partitioned dataset

### Making a partition file

To choose different models for different partitions, and partition schemes, we need to specify the character sets that correspond to different partitions. Ideally, we should define the partitions in the most granular and biologically meaningful way possible. This will typically involve separating intros from CDS, and splitting CDS into the three codon positions. For the MCM7, we will create a partition file that specifies the caracters in each of the 3 codon positions.

In the terminal tab that for which the working directory is on your local machine, create a new file for the partitions:

```sh
touch class_activities/week6/alignments/mcm7_partitions.txt
```

Open it with the nano editor, and add define the partittions as follows:

```
DNA, first = 1-537\3
DNA, second = 2-537\3
DNA, third = 3-537\3
```

Remember that we exported the FASTA file version of the MCM7 alignment and removed the excluded sites. So, all included sites are coding. Since the exported alignment has 537 sites, all partitions are defined in terms of 537 characters. Also, if you look at the original alignment in Mesquite, you will see that the included nucleotide is coding, and it corresponds to codon position 1. That is why the first character in the partition named "first" starts at position 1. 

Close, and save the file. This partition definition is currently in the RAxML format, but you can see that the codon position definitions use the same syntax as the NEXUS format.

Push the partition file to the cluster:

```
scp class_activities/week6/alignments/mcm7_partitions.txt YOURNETID@dcc-login.oit.duke.edu:/hpc/group/bio556l-s23/YOURNETID/class_activities/week6/alignments
```

### Choosing the best model and partition scheme

We are now going to run a model selection analyses that will also test for for the best partition scheme and best model for each selected partition using PartitionFinder2, which is implemente in IQ-Tree:

```
iqtree -s class_activities/week6/alignments/Lecanoraceae_v8_MCM7.fas \
 -pre class_activities/week6/model_selection/lecanoraceae_mcm7_part \
 -spp class_activities/week6/alignments/mcm7_partitions.txt \
 -m MF+MERGE -nt 2
```

The two new options here are `-spp` which we used to indicate the partition file. We are now using the `-m MF+MERGE` option which tells IQ-Tree to run a PartitionFinder analysis and find the best model for each partition using ModelFinder. Finally, we are now specifying how many cores we want IQ-Tree to use for this using the `-nt` option. This analysis will use 2 cpus.

We it's done, we can once again inspect the iqtree report:


```sh
nano class_activities/week6/model_selection/lecanoraceae_mcm7_part.iqtree
```

An we can also look at the resulting best scheme and models in the NEXUS format.

```sh
nano class_activities/week6/model_selection/lecanoraceae_mcm7_part.best_scheme.nex
```


<a name=MADD></a>
## Protein models and the domains of the Tree of Life

### Prepare for batch job submission

For this set of analyses we are going to use scripts to submit our job tasks instead of running them in an interactive session. Let's start by creating some directories to organize the scripts and outputs of our analyses:

```sh
mkdir -p scripts/week6
mkdir -p log/week6
mkdir -p class_activities/week6/trees
```

The `scripts/week6` folder is where we will store our scripts, `log/week6` is where we will ask SLURM to write files containing the STDOUT and STDERR of the jobs, and the `class_activities/week6/trees` is where we will store the iqtree output, which will include maximum likelihood trees this time. And not just any tree, we're talking about the *Tree of Life* (ToL).

### Maximum Likelihood ToL with severe model violation

We are going to be using a concatenated protein alignment of 35 core genes and 81 taxa from bacterial, archaea, an eukaryotic lineages that was assembled by [Da Cunha et al., 2017](https://journals.plos.org/plosgenetics/article/file?id=10.1371/journal.pgen.1006810&type=printable) and later re-analyzed by [Williams et al,. 2020](https://www.nature.com/articles/s41559-019-1040-x). Both studies were concerned with figuring out the number of domains of life, with two competing hypothesis. The 3D (three domain) hypothesis, where Bacteria, Archaea, and Eukarya each comprise their own domain; and the 2D (two domain) hypothesis, where Eukarya evolved from within Archaea forming one domain, and Bacteria is separate.

First, we are going to infer a maximum likelihood tree for this dataset under a a very basic–and in all likelihood innapropriate– model of amino acid exchangeabilities (BLOSUM62), and without considering any sort of heterogeneity.

Let's create a file where we will write our script, then open it with the nano editor:

```sh
touch scripts/week6/tol_blosum62.sh
nano scripts/week6/tol_blosum62.sh
```

In the editor, type the following:

```sh
#!/bin/bash

#SBATCH --mem-per-cpu=4G  # adjust as needed
#SBATCH -c 8 # number of threads per process
#SBATCH --output=log/week6/tol_blosum62.out
#SBATCH --error=log/week6/tol_blosum62.err
#SBATCH --partition=scavenger

# Load IQ-tree module
module load IQ-TREE/1.6.12
# Run Iq-tree
iqtree -s class_activities/week6/alignments/35_noEF2.fas \
 -pre class_activities/week6/trees/tol_blosum62 \
 -m BLOSUM62 -nt 8
```

Exit and sava the file with the same name. Then, submit the job with the script:

```
sbatch scripts/week6/tol_blosum62.sh
```

This should return an ID for the job. You can use that ID to monitor the status of the job using SLURM commands such as `sacct`, for example:

```
sacct -j JobID
```

See the [SLURM Quick Start User Guide](https://slurm.schedmd.com/quickstart.html) for other common commands to monitor the status and performance of your jobs.

### Maximum Likelihood ToL with less severe model violation

While that is running, let's submit another job where we get a maximum likelihood tree but using a better matrix of amino acid exchangeabilities (LG), but without considering any sort of heterogeneity.

Again, create a file to write the script and open it with nano:

```sh
touch scripts/week6/tol_lg.sh
nano scripts/week6/tol_lg.sh
```

In the editor, copy and paste the following script:

```sh
#!/bin/bash

#SBATCH --mem-per-cpu=4G  # adjust as needed
#SBATCH -c 8 # number of threads per process
#SBATCH --output=log/week6/tol_lg.out
#SBATCH --error=log/week6/tol_lg.err
#SBATCH --partition=scavenger

# Load IQ-tree module
module load IQ-TREE/1.6.12
# Run Iq-tree
iqtree -s class_activities/week6/alignments/35_noEF2.fas \
 -pre class_activities/week6/trees/tol_lg \
 -m LG -nt 8
```

Close the editor, save the file, and submit the job:

```
sbatch scripts/week6/tol_lg.sh
```

Once the two jobs are completed, pull the trees folder to check the reports and resulting trees (remember to do this on the terminal tab where the working directory is on your local machine):

```
scp -r YOURNETID@dcc-login.oit.duke.edu:/hpc/group/bio556l-s23/YOURNETID/class_activities/week6/trees class_activities/week6
```






