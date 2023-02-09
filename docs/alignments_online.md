# Part 1: Aligments with MAFFT online and Mesquite
___

## Contents

* **[Protein coding sequences with introns (fungal beta-tub)](#btub)**
* **[Coding DNA sequences only (bacterial rpsC)](#resources)**
* **[Concatenting sequence and morphological data in Mesquite](#alignment)**

<a name="btub"></a>
## Protein coding sequences with introns (fungal beta-tub)

### Aligning sequences using the MAFFT online server

For this example we will use sequences from the beta-tubuling gene in lichen-forming fungi. In your favorite text editor, open the file `week3/seqs/betatub/peltigera_betatub.fasta`:

![betatubfasta](https://raw.githubusercontent.com/cjpardodelahoz/BIO556L/main/docs/images/alignments/betatub_seqs.png)

The sequences are in *fasta* format, which is specified by a sequence header starting with `>`followed by the label, a new line ccharacter, and then the string of sequence letters.

Copy all the sequences and navigate to the [MAFFT online server](https://mafft.cbrc.jp/alignment/server/). Then paste the sequences in the input box:

![mafftinput](https://raw.githubusercontent.com/cjpardodelahoz/BIO556L/main/docs/images/alignments/mafft_input.png)

This online interface allows you to play with alignment algorithm settings. We will go over some these settings during class. For now, let's use the defaults. Hit the submit buttom. After the alingment algorithm is done, you will see the output alignment displayed in CLUSTAL interleaved format:

![mafftout](https://raw.githubusercontent.com/cjpardodelahoz/BIO556L/main/docs/images/alignments/mafft_output.png)

You can see that gaps have been added, to the alignment and the asterisks at the end of the character columns indicate sites for which all taxa have the same nucleotide, whereas the dots indicate sites where a single taxon has a difference. Click on the "Fasta format" link at the top left to download the resulting alignment in fasta format. The file will be named with an alphanumeric label. Rename it to somehting understandable (e.g. `peltigera_betatub_aln.fasta`) and move it to the `week3/seqs/betatub` directory.

### Visualizing the beta-tub alignment with Mesquite

Open Mesquite, and got to **File>Open File** and select the file where you saved the alignment output from MAFFT online. Every time you open a file with mesquite that is not in NEXUS format, mesquite will first want to convert it and save in NEXSUS format. Therefore, the first thing it will ask is for a format to interpret the file:

![mesquite_format](https://raw.githubusercontent.com/cjpardodelahoz/BIO556L/main/docs/images/alignments/mesquite_format.png)

Choose FASTA (DNA/RNA) and clik *OK*. Then it will ask you how you want to name the resulting NEXUS file. Choose a name and give it the `.nex` extension (e.g. `peltigera_betatub_aln.nex` to remind you that it is the NEXUS format.

The next thing we want to do is annotate our alignment with the sites that are protein-coding and indicate to which codon position they correspond. This will be allow us to use the codon positions and the amino acid translation as a guide to edit protein-coding alignments.

These sequences are deposited in the GenBank database from the National Center for Biotechnology Information (NCBI). The first field of the taxon names in this dataset correspond to the GenBank accession numbers of the sequences, which are unique identifiers that allow us to retrieve the record from the database, along with all the associated metadata. The metadata includes sequence features, such as the regions of the sequence that are coding, and the codon positions.

 Let's take the accession number of the first sequence (`MH770992`) and serach for it in *nucleotide* database of [GenBank](https://www.ncbi.nlm.nih.gov/genbank/). The search will take us directly to the record for this sequence. Scroll down to the FEATURES section, which includes the Coding DNA Sequence (CDS) annotation:

![genbank](https://raw.githubusercontent.com/cjpardodelahoz/BIO556L/main/docs/images/alignments/genbank_features.png)

This tells us that in that sequence, the first coding region starts with site 73 and goes all the way to site 96. Then there is a non-coding sequence region, and the coding part starts again at site 147, etc. It also tells us that the first site that is coding in this sequence corresponds to codon position 1 (/codon_start = 1). Recall that introns may be located in between any of the three codon positions, so the first coding nucleotide in a partial sequence won't always be codon position 1. We can use this information to annotate all of our sequences in Mesquite, by finding the character in the alignment that corresponds to site 73 in this particular sequence that we looked up in GenBank. 

To do this, you can hover your cursor over the matrix cells and look at the information displayed on the lower panel of Mesquite:

![mesquite_locate](https://raw.githubusercontent.com/cjpardodelahoz/BIO556L/main/docs/images/alignments/mesquite_locate.png)

In this example, if your cursor is over the G shown with the black circle, the lower panel will indicate that you are in character 92 of the alignment matrix, and that the cell corresponds to site 73 of the first sequence. 

Let's now find the end of the first coding region, which spans from site 73 to site 96. Once you find the character that has site 96 (character 115) for the first sequence, click on the character name that has that site (the character names are the numbers in the gray boxes above the matrix). Then, hold shift and click on the character name that has site 73 (character 92):

![charselect](https://raw.githubusercontent.com/cjpardodelahoz/BIO556L/main/docs/images/alignments/char_select.png)

This will highlight the columns of the matrix in that span, indicaing that these characters are now *selected* for Mesquite. We can no indicate that those selected characters are coding.

Go to **Characters > List of Characters**. Mesquite will open a new tab with the list of characters in the matrix. If you scroll down, you will see that characters 92–115 are selected. To set this as coding, click on **Codon position > Set codon position > 123123...**

![secodon](https://raw.githubusercontent.com/cjpardodelahoz/BIO556L/main/docs/images/alignments/set_codon.png)

In this case, we chose 123123... because we know that the fir coding site corresponds to the first codon position, but this might change for different coding regions and sequences. After you click that option, you will see that the codon position value for the selected characters will be updated.

If we return to the Character Matrix tab, you will see that Mesquite has now highlighted the seelcted characters with three different colors: blue for 1st codon position, green for second, and red for third.

With this annotation we can ask Mesquite to change how the coding characters are displayed, which may be useful for editing the alignment. My favorite is to Color the matrix cells according to the amino acids. You can set it **Display > Color Matrix Cells > Color AAs to Check** and you should get something like this:

![coloraas](https://raw.githubusercontent.com/cjpardodelahoz/BIO556L/main/docs/images/alignments/aascolored.png)


