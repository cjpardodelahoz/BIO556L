# Bayesian Phylogenetic Inference

## Contents

* **[Posterior sampling of gene trees using MrBayes and SLURM arrays](#MB)**
* **[Sampling from the Prior using MrBayes](#MBP)**
* **[ToL inference with empirical profile mixtures of amino acid frequencies in PhyloBayes3](#PBE)**
* **[ToL inference with the CAT-GTR model of PhyloBayes3](#PBI)**


<a name="MB"></a>
## Posterior sampling of gene trees using MrBayes and SLURM arrays

For the next several labs dealing with sources of incongruence, we will be using a dataset of Nostocalean cyanobacteria that I worked on as part of my PhD. The Advance Access version of the paper is now published in *Systematic Biology*. We will use the results of this bayesian analyses for some of our future exercices on phylogenetic networks and divergence time estimation. 

The first part that we will do today is to get a sample of the posterior distrubtion of trees from X protein coding genes.

### Setting up a MrBayes block

MrBayes works with the NEXUS format, and just like PAUP, we can use commands interactively, or write them as a `mrbayes` block in the NEXUS. We are going to do this inference for multiple genes, but using the same `mrbayes` block, so we will first create the block in a text file.

Create a new directory this week's scripts and open a new file to store the bayes block:

```sh
mkdir scripts/week8
mkdir -p misc_files/week8
nano misc_files/week8/bayes_block_na.txt
```

In the editor, copy and paste the following block:

```
begin mrbayes;
set autoclose=yes nowarnings=yes;
lset nst=Mixed rates=gamma;
prset Revmatpr=Dirichlet(1.0,1.0,1.0,1.0,1.0,1.0) 
        Statefreqpr=Dirichlet(1.0,1.0,1.0,1.0) 
        Shapepr=Exponential(1.0) 
        Topologypr=Uniform 
        Brlenspr=Unconstrained:GammaDir(1.0,0.100,1.0,1.0);
mcmc ngen=1000000 samplefreq=100 savebrlens=yes
	;
sumt;
sump;
quit;
```

We will go over these options in class. You can [download the MrBayes manual](https://nbisweden.github.io/MrBayes/manual.html) if you want to learn more. You can also get help within MrBayes when you run it interactively by typing `help <command>`. The majority of settings that you might want to play with are options of the commands `lst` which sets the model likelihood settings, `prset` which is for setting priors, and `mcmc` which is for the mcmc sampling.



### Script to run MrBayes on a single alignment

Copy the `week8` directory from `source` to your class activities folder. This has the alignments for the 32 genes that we will use, already in the NEXUS format:

```sh
cp -r ../source/week8 class_activities
```

Let's inspect one of the files:

```sh
nano class_activities/week8/mrbayes_files/28at1161_subset1_ng.nex
```

Let's create a file for a script to run a single alignment using the bayes block we created previously:

```sh
mkdir log/week8
nano scripts/week8/mrbayes_single.sh
```

In the editor, copy the script:

```
#!/bin/bash

#SBATCH --mem-per-cpu=4G  # adjust as needed
#SBATCH -c 1 # number of threads per process
#SBATCH --output=log/week8/mrbayes_single.out
#SBATCH --error=log/week8/mrbayes_single.err
#SBATCH --partition=scavenger

# Path to MrBayes
export PATH=/hpc/group/bio556l-s23/apps/MrBayes-3.2.7a/src:${PATH}
# Run MrBayes
mb class_activities/week8/mrbayes_files/28at1161_subset1_ng.nex \
 misc_files/week8/bayes_block_na.txt
```

### Script to run MrBayes on mulitple alignments using SLURM array

We are going to turn the script we just made into one that will execute a job for every one of the 32 alignments. For this, we just need to have a file that has a list of the paths to all alingments that we want to run. we can generate it redirecting the output of `ls` into a file:

```sh
ls -d class_activities/week8/mrbayes_files/* > \
 misc_files/week8/alignment_paths.txt
```

We can now use that list to generate a variable with the file names in the array script. Open the script we just created: 

```sh
nano scripts/week8/mrbayes_single.sh
```

and modify it so it looks like this (we will go over this in class):

```sh
#!/bin/bash

#SBATCH --array=1-32
#SBATCH --mem-per-cpu=4G  # adjust as needed
#SBATCH -c 1 # number of threads per process
#SBATCH --output=log/week8/mrbayes_array_%A_%a.out
#SBATCH --error=log/week8/mrbayes_array_%A_%a.err
#SBATCH --partition=scavenger

# Path to MrBayes
export PATH=/hpc/group/bio556l-s23/apps/MrBayes-3.2.7a/src:${PATH}
# Variable with alignment paths
alignment=$(cat misc_files/week8/alignment_paths.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)
# Run MrBayes
mb ${alignment} misc_files/week8/bayes_block_na.txt
```
When you close the editor, save the modified file as `mrbayes_array.sh`.

Now we can submit this job usign sbatch:

```sh
sbatch scripts/week8/mrbayes_array.sh
```

We can monitor the status of each job within the array using the jobID:

```sh
sacct -j JOBID
```

Let's also look at the STDOUT file of one of the jobs to familiarize ourselves with the format:

```sh
nano log/week8/mrbayes_array_JOBID_1.out
```

Let's also inspect the file that stores the sampled trees `.t`, and the sampled parameter values `.p`:

```sh
nano class_activities/week8/mrbayes_files/118at1161_subset1_ng.nex.run1.p
nano class_activities/week8/mrbayes_files/118at1161_subset1_ng.nex.run1.t
```

<a name="MBP"></a>
## Sampling from the Prior using MrBayes

Sampling from the prior is a critical step when doing bayesian inference. Examining the prior samples allows to check for situations where interactions between our priors can generate marginal prior distributions that are different from what we intendedn. Camparing prior and posterior samples also gives us a sense of the strength of the signal in the data for particular parameters.

### A MrBayes block to sample from the prior

Now, let's create a new mrbayes block file with the setting to sample from the prior

```sh
nano misc_files/week8/bayes_block_na_prior.txt
```

In the editor, paste the block:

```
begin mrbayes;
set autoclose=yes nowarnings=yes;
lset nst=Mixed rates=gamma;
prset Revmatpr=Dirichlet(1.0,1.0,1.0,1.0,1.0,1.0) 
        Statefreqpr=Dirichlet(1.0,1.0,1.0,1.0) 
        Shapepr=Exponential(1.0) 
        Topologypr=Uniform 
        Brlenspr=Unconstrained:GammaDir(1.0,0.100,1.0,1.0);
mcmc data=no ngen=1000000 samplefreq=100 nruns=1 nchains=1
	filename=class_activities/week8/mrbayes_files/prior;
quit;
```

Now, let's make a script to sample form the prior:

```sh
nano scripts/week8/mrbayes_prior_array.sh
```

In the editor, paste the following:

```sh
#!/bin/bash

#SBATCH --array=1
#SBATCH --mem-per-cpu=4G  # adjust as needed
#SBATCH -c 1 # number of threads per process
#SBATCH --output=log/week8/mrbayes_prior_%A_%a.out
#SBATCH --error=log/week8/mrbayes_prior_%A_%a.err
#SBATCH --partition=scavenger

# Path to MrBayes
export PATH=/hpc/group/bio556l-s23/apps/MrBayes-3.2.7a/src:${PATH}
# Variable with alignment paths
alignment=$(cat misc_files/week8/alignment_paths.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)
# Run MrBayes sampling from the prior
mb ${alignment} misc_files/week8/bayes_block_na_prior.txt
```

Save it and submit the job:

```sh
sbatch scripts/week8/mrbayes_prior_array.sh
```

### Examine the MCMC samples with Tracer

While we wait for these reuns to finish, let's download Tracer, a Java application that we can use to look at the distribution of log likelihoods and posterior probabilities across the samples from the mcmc runs. You can [download it from here](https://github.com/beast-dev/tracer/releases/tag/v1.7.2).

When the runs are done, let's go ahead and pull the week8 directory into our local copy of class activities to look at the results:

```sh
scp -r cjp47@dcc-login.oit.duke.edu:/hpc/group/bio556l-s23/cjp47/class_activities/week8 \
 class_activities
```

We will use the Tracer application to look at the tracer plots to check for convergence of the chains and among runs.


