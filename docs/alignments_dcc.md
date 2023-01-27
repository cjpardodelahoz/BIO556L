[Setup and intro to Unix basics](##setup)

[Requesting resources for an interactive session in the DCC](##resources)

<a name="setup"></a>
## Setup and intro to Unix basics

### Logging into the Duke Computer Cluster
Open a terminal (MacOS, Linux) or command prompt (Windows) and log into the Duke Computer Cluster by typing:

```sh
ssh YOURNETID@dcc-login.oit.duke.edu 
```
You will be asked for your password and MFA. Keep in mind that you won't see the characters of your password when you are typing. Don't worry, the terminal is still getting them!

### Where am I?
When you log in, you are using a login *node* in your `${HOME}` directory. You can type `pwd`(**p**rint **w**orking **d**irectory) and hit enter and you will see the full path to your `${HOME}`. It should be something like `/hpc/home/YOURNETID`.
#### How do I navigate directories?
Now let's navigate to the class directory using the `cd` (**c**hange **d**irectory) command and the *path* to the directory where you want to go:

```sh
cd /hpc/group/bio556l-s23/
```
If you type `pwd` again, you should see the path that you just navigated to. We will be doing all of the class work within this directory. 

You can think of navigation in the command line as moving up and down a tree of directories represented by their paths. To move down, you simply need to type the path downstream of where you are. For example, if you are in `/hpc` and you want to go to `/hpc/group/bio556l-s23`, you can do `cd group/bio556l-s23`. If you get there, and you want to move back up to `/hpc`, you can do `cd ../../`. When moving up, every `../` brings you up one directory level in that path.
### What is in this directory?
You can type `ls` and hit enter. This will **l**i**s**t the contents of the currernt directory. In this case, you will see another directory called `source`.
### Creating new directories
You can **m**a**k**e a new **dir**ectory using the command `mkdir`. Go ahead and make a directory with the same name as your netid:

```sh
mkdir YOURNETID
```
This directory will now be your class home. CD into that directory and create a new directory inside of it called `class_activities`:

```sh
cd YOURNETID
mkdir class_activities
```
**Tip**: You can create subdirectories even if you are not currently in the parent directory. You do so by using the path to the parent directory where you want to create the new subdirectory. For example, if you do `pwd` right now, you should be in `/hpc/group/bio556l-s23/YOURNETID`. If you do `ls`, you will see `class_activities`, the directory you just created. For example, to create a directory called week3 *inside* of class_activities, you can give the full path:

```sh
mkdir /hpc/group/bio556l-s23/YOURNETID/class_activities/week3
```
You can also give the *relative path*, which is relative to your current diretory. For example, since you are already in `/hpc/group/bio556l-s23/YOURNETID`, you can simply give the relative path:

```sh
mkdir class_activities/week3
```
**Any command that takes absolute paths, can also take relative paths.**

You can also create new directories even if the parent directories in the path don't exist. You di this using the `-p` flag of the `mkdir` command. For example, to create a directory called `scripts` (which does not exist yet) and a subdirectory inside of it called `week3`, you can do it in a single command:

```sh
mkdir -p scripts/week3
```
If you do `ls`, you will now see the scripts directory. And if you do `ls scripts/` it will list the contents of the `scripts` directory, where you will also see the subdirectory `week3`.
### Copying files
You can **c**o**p**y files using the command `cp PATH/TO/ORIGINAL PATH/TO/COPY`. This works for individual  files within a single directory. If you want to copy a directory and its contents, you can add the `-r` flag, which will copy files and subdirectories **_r**ecursively_. For today's class, we'll need the contents of the `/hpc/group/bio556l-s23/source/week3` directory. Go ahead and copy all the contents of that directory into your `class_activities` directory:

```sh
cp -r ../source/week3 class_activities/week3
```
### Inspecting and editing text files
There are multiple ways to look at the contents of files in the command line. We will start by using the `nano` text editor where we can inspect and edit the files. For example, let's have a look at one of the sequence files that we will use today:

```sh
nano class_activities/week3/seqs/betatub/peltigera_betatub.fasta
```
This will open the file in the `nano` editor, where you can edit, search, replace, etc. using the commands that are displayed at the bottom of the terminal. This particular file contains the set of beta-tubulin sequences in FASTA format that we used previously. You can exit the text editor by typing `ctrl+x`.

We now have the directory setup and a first stroke of Unix basics that will be enough to perform the alignment excercises of today's class. I will introduce useful commands as we increase the complexity of our computational tasks throughout the class.

<a name="resources"></a>
## Requesting resources for an interactive session in the DCC
Let's now get ready to run our first analyses on the DCC. The DCC works with a job scheduler called SLURM, which is a software used to allocate computational resources to different tasks and users. We will become proficient with the most important functions of SLURM by the end of this class. You can also check the [DCC user guide](https://dcc.duke.edu/dcc/).

To request resources that will be allocated to an interactive session, we do:

```sh
srun --pty bash -i
```
This will indicate that we want to use bash as the interpreter for the command line. When the resources are allocated, our interactive session will reserve a jobID. We can use that later to cancel the session (`scancel jobID`) or get performance statistics. By default, the `srun` command asks for 1 cpu core, in a single node, and with 2GB of memory. We can change all of those defaults using the flags `-c` for number of cpu cores, `-n` for number of nodes, and `--mem` for RAM. For example (note that you need to cancel the previous session before running this):

```sh
srun -c 2 --mem=4G --pty bash -i 
```
will request an interactive session with 2 cpu cores and 4 GB of RAM. These resources will be enough for the alignments we will conduct.

## Protein-coding sequence alignment with MAFFT and PAL2NAL
We are almost ready to replicate our analysis in the command line! The last bit of set up involves making sure that we have access to the software that we need: MAFFT and PAL2NAL. We will learn later in class how to build and install command live versions of phylogenetic software. But for today, we will use a pre-existing installation. In order to use it, we have to tell the command line where these programs are. One way that facilitates accesing them is by having them on our `${PATH}` variable. The `${PATH}` variable is sort of like a speed dial for directory paths. You can access files and programs that are in that variable without having to type their full path. So, let's add the installations of MAFFT and PAL2NAL to our `${PATH}` variable:

```sh
# Adding path to MAFFT
export PATH=/hpc/home/cjp47/mafft-7.475-with-extensions/bin:${PATH}
# Adding path to PAL2NAL
export PATH=/hpc/group/bio1/carlos/apps/pal2nal.v14:${PATH}
```
Now, we have the executables of MAFFT and PAL2NAL on speedial. So we can access them regardless of the directory where we are. Try typing:

```sh
mafft -h
pal2nal.pl -h
```
That should display the help menus of both programs. And we are ready!

Let's align the betatubulin nucleotide sequences. The basic syntax for MAFFT is `mafft OPTIONS PATH/TO/INPUT > PATH/TO/OUTPUT`. For example:

```sh
mafft --auto \
 class_activities/week3/seqs/betatub/peltigera_betatub.fasta > \
 class_activities/week3/seqs/betatub/peltigera_betatub_aln.fasta
```
We can do something similar with the rpsc amino acid sequences:

```sh
mafft --auto \
 class_activities/week3/seqs/rpsc/rpsc_aa.fasta > \
 class_activities/week3/seqs/rpsc/rpsc_aa_aln.fasta
```
And then, we can use this amino acid alignment as a guideline to align the rpsc nucleotide sequences with PAL2NAL:

```
pal2nal.pl class_activities/week3/seqs/rpsc/rpsc_aa_aln.fasta \
 class_activities/week3/seqs/rpsc/rpsc_na.fasta \
 -codontable 11 -output fasta > \
 class_activities/week3/seqs/rpsc/rpsc_na_aln.fasta
```
