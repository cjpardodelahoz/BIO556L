# Phylogenetic Network Inference with SNaQ

Species Networks applying Quartets (SNaQ) is a program implemented in a julia package called Phylonetworks ([Solís-Lemus et al 2017](https://academic.oup.com/mbe/article/34/12/3292/4103410)). It is an extension of the MultiSpecies Coalescent Model to reticulated species trees that relates frequencies of quartets topologies with phylogenetic network topologies using pseudolikelihood scores ([Solís-Lemus and Ané 2016](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1005896)).

We are going to run through one of the mini tutorials from the authors to get acquited with how to run SNaQ and some of the principles of phylogenetic network inference.

### Julia and Phylonetworks setup

Let's log into the DCC, request resources for an interactive session, and use my installation of Julia: 


```sh
ssh YOURNETID@dcc-login.oit.duke.edu
srun -c 2 --mem=8Gb -p scavenger --pty bash -i
export PATH=/hpc/home/cjp47/julia-1.5.2/bin/:$PATH
julia
```
Now, we need to install the required packages in your home directories:

```julia
# This is the Package manager
using Pkg
# These are to handle tables and CSV files
Pkg.add("CSV")
Pkg.add("DataFrames")
# This is the package for the phylogenetic network analyses, which contains the snaq function
Pkg.add("PhyloNetworks")
# This is for plotting the networks (might be tricky to install on DCC because it relies on R)
# Pkg.add("PhyloPlots")
```

We now need to load the packages we just installed. The first time you load a package in Julia, it does the package building, so it can take several minutes:

```julia
using CSV, DataFrames, PhyloNetworks
```

### Using Phylonetworks 

Once we are at that point, we are going to follow the tutorial from the SNaQ developers which has a straighforward dataset that introduces the main funcitonalities. [Go to the tutorial](https://crsl4.github.io/PhyloNetworks.jl/latest/man/inputdata/).