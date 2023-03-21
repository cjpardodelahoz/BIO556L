# Species Trees and Gene Tree Heterogeneity

## Contents

* **[Inferring a concatenated tree with maximum likelihood](#ML)**
* **[Inferring a summary species tree with ASTRAL](#AS)**


<a name="ML"></a>
## Inferring a concatenated tree with maximum likelihood

We will continue using a dataset of Nostocalean cyanobacteria that we used for our excercise on [bayesian phylogenetics](https://cjpardodelahoz.github.io/BIO556L/bayes). It consists of 12 taxa and 32 nucleotide alignments of protein-coding core genes. The alignments were generated with MAFFT and sites with gaps were trimmed.


### Concatenating the nucleotide alignments

The alignments are in nexus format in `class_activities/week8/alignments/*.nex`. we are going to create a script that will use the [AMAS](https://github.com/marekborowiec/AMAS) tool to concatenate the sequence and generate a partition file. Open a new file for the script:

```sh
nano scripts/week8/concatenate_nostocales32.sh
```

Then, in the editor, type the following:

```sh
#!/bin/bash

#SBATCH --mem-per-cpu=8G  # adjust as needed
#SBATCH -c 1 # number of threads per process
#SBATCH --output=log/week8/concatenate_nostocales32.out
#SBATCH --error=log/week8/concatenate_nostocales32.err
#SBATCH --partition=scavenger

# Path to AMAS
export PATH=/hpc/group/bio556l-s23/apps/AMAS/amas:${PATH}
# Concatenate nucleic acids
AMAS.py concat -i class_activities/week8/alignments/*.nex \
 -f nexus -d dna \
 -p class_activities/week8/alignments/nostocales32_concat_part.txt \
 --out-format nexus \
 --concat-out class_activities/week8/alignments/nostocales32_concat.nex \
 --part-format raxml
```
Submit the job:

```sh
sbatch scripts/week8/concatenate_nostocales32.sh
```
When it's done, check out the concatenated alignment:

```sh
nano class_activities/week8/alignments/nostocales32_concat.nex
```
and the partition file:

```sh
nano class_activities/week8/alignments/nostocales32_concat_part.txt
```


### Run a maxmimum likelihood concatenated analysis

Now we can use the concatenated alignment and the gene-partition file to infer a tree, searching for the best partitions, and models, bootstraping--the whole shebang--using IQ-Tree:

```sh
nano scripts/week8/ml_concat_nostocales32.sh
```
Paste the the iqtree script in the editor:

```sh
#!/bin/bash

#SBATCH --mem-per-cpu=4G  # adjust as needed
#SBATCH -c 8 # number of threads per process
#SBATCH --output=log/week8/ml_concat_nostocales32.out
#SBATCH --error=log/week8/ml_concat_nostocales32.err
#SBATCH --partition=scavenger

# IQ-tree module
module load IQ-TREE/1.6.12
# Directory for output
mkdir -p class_activities/week8/trees/concat
# Run Iq-tree
iqtree -s class_activities/week8/alignments/nostocales32_concat.nex \
 -pre class_activities/week8/trees/concat/nostocales32_concat \
 -spp class_activities/week8/alignments/nostocales32_concat_part.txt \
 -m MFP+MERGE -rclusterf 10 -bb 1000 -nt 8
```

Submit the job:

```sh
sbatch scripts/week8/ml_concat_nostocales32.sh
```
This one will take some time, so let's go to the next section and then we will compare the results. 


<a name="AS"></a>
## Inferring a summary species tree with ASTRAL

[ASTRAL-III](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-018-2129-y) is a summary method that uses gene trees as input to build summary species trees that are *generally* consistent witht he multi-species coalescent model, thus accounting for incomplete lineage sorting (ILS). It basically decomposes the input trees into quartets and counts the frequencies of the three alternative topologies for each quartet, and then tries to find a tree that is consistent with the majority of the most frequent quartet resolutions.

### Prepare the input trees for ASTRAL

We are going to get an ASTRAL tree using the bayesian maximum clade credibility trees that we inferred in a [previous excercise](https://cjpardodelahoz.github.io/BIO556L/bayes) using this same dataset for nostocales. To run ASTRAl, we need all input gene trees in single text file in newick format. 

The results of the bayesian analyses are under `class_activities/week8/mrbayes_files/`. What we need for today are the consensus trees built from the posterior sample of trees for each locus, which are under `class_activities/week8/mrbayes_files/*.con.tre`. You should have 32 of those files. You can test that doing:

```sh
ls class_activities/week8/mrbayes_files/*.con.tre | wc -l
```

Let's first make a copy of those tree files and put them in a single directory:

```sh
# Directory to store the trees
mkdir -p class_activities/week8/trees/bayes_single
# Copy the trees
cp class_activities/week8/mrbayes_files/*.con.tre \
 class_activities/week8/trees/bayes_single
# Check that everything worked properly
ls class_activities/week8/trees/bayes_single | wc -l
```
Let's now look inside one the tree files to see what format we are working with: 

```sh
nano class_activities/week8/trees/bayes_single/118at1161_subset1_ng.nex.con.tre
```
As you can see, these trees are in NEXUS format, so we first need to convert them to newick. We will do this using the R package [*ape*](https://cran.r-project.org/web/packages/ape/ape.pdf). 

We are going to do this interactively so we can troubleshoot more easily, but you can also run this in batch mode. Let's request resources for an interactive session:

```sh
# Request an interactive session with 1 cpu1 and a total of 8Gb of RAM
srun -c 1 --mem=8G -p scavenger --pty bash -i
```

Then, load the module for R version 4.1.1 and start R

```sh
module load R/4.1.1-rhel8
R
```

After this, your command prompt will be inside the R application. Install the R packages "ape" and "stringr"

```R
install.packages("ape")
```
R might ask you two things if you have not done this before. The first is to confirm whether it is ok to create an R library in your home directory given that you do ont have root access. The second is to select a CRAN mirror. Say yes to the first and select any mirror 75.

Once the installation is finished, load the package:

```R
library(ape)
```
Now let's convert the tree files:

```R
# Get a list of the paths to the tree files
tree_paths <- list.files(path = "class_activities/week8/trees/bayes_single",                          pattern = ".tre", full.names = T)
# Iterate thorugh the paths, load the nexus tree, and save it as newickfor (tree_path in tree_paths) {  outfile <- gsub(".nex.con.tre", ".nw", tree_path)  tree <- read.nexus(file = tree_path)  write.tree(tree, file = outfile)}
# Quit R
quit()
```
Now, let's check that all the trees were converted by counting how many `.nw` files we have, and that they are newick format:

```sh
# Count .nw files
ls class_activities/week8/trees/bayes_single/*.nw | wc -l
# Check one to see format
nano class_activities/week8/trees/bayes_single/118at1161_subset1_ng.nw
```

Now that we have them all in the right format, let's put them all in a single file using `cat`. This will be our input for ASTRAL:

```sh
# Directory for ASTRAL analysis
mkdir -p class_activities/week8/trees/astral
# Compile gene trees
cat class_activities/week8/trees/bayes_single/*.nw > \
 class_activities/week8/trees/astral/nostocales32.trees
```
The resulting file should have 32 lines, one tree per line:

```sh
nano class_activities/week8/trees/astral/nostocales32.trees
```

Now we are ready to run ASTRAL.

### Running ASTRAL

ASTRAL typically runs very fast, so let's take advantage of our interactive session and run it interactively (although you can also run in in batch mode with more cores if needed):

```sh
# Load required java module
module load Java/1.8.0_60
# Run Astral
java -jar /hpc/group/bio556l-s23/apps/Astral/astral.5.15.5.jar \
 -i class_activities/week8/trees/astral/nostocales32.trees \
 -o class_activities/week8/trees/astral/nostocales32_astral.tree
```
By default, ASTRAL will annotate the branches only with the posterior probaility for the shown branches, but we can ask it to give us more information with the option `--branch-annotate`. For example, let's do an analysis where ASTRAL will give us the quartet score for each alternative quartet resolution around each branch:

```sh
# Run Astral
java -jar /hpc/group/bio556l-s23/apps/Astral/astral.5.15.5.jar \
 --branch-annotate 8 \
 -i class_activities/week8/trees/astral/nostocales32.trees \
 -o class_activities/week8/trees/astral/nostocales32_astral_qscores.tree
```


On your terminal that is on your local copy of the class directory, download all the results from today's analyses:

```sh
scp -r YOURNETID@dcc-login.oit.duke.edu:/hpc/group/bio556l-s23/YOURNETID/class_activities/week8/trees class_activities/week8/
```
And let's look at the trees with FigTree.

### Detecting the Anomaly Zone

We can use equation 4 from [Degnan and Rosenberg 2006](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.0020068) to test whether a pair of internodes fall within the anomaly zone given their coalescent lengths.

Let's use our ASTRAL tree to test for this. On your terminal that is logged in the cluster and with an interactive session running, start r and get load this function:


```R
# Start R
R
# Function to obtain a(x) from Degnan and Rosenberg 2006 Plos Biol.
deg_ros_a_of_x <- function(x) {
  num <- 3*(exp(1)^(2*x)) - 2
  den <- 18*((exp(1)^(3*x)) - (exp(1)^(2*x)))
  a_of_x <- log(2/3 + (num/den))
  return(a_of_x)
}
```

We can use that to comput the internole length limit that defines the anomaly zone.


