## Mapping Exercise

MIP 280A4
---

## First: login to thoth01 and activate the bio_tools conda environment 

Login to the thoth01 server:
```
ssh your_eid@thoth01.cvmbs.colostate.edu
```

To activate the bio_tools conda environment, you will need to run:
```
conda activate bio_tools
```

**You will need to activate this environment every time you want to use these tools.**



## In this exercise, we will learn how to create an index from a reference sequence, then map reads to that reference sequence

### [Last time] Downloading an SRA dataset

Last week, you downloaded one of the NGS datasets reported in [this paper](http://journals.plos.org/plospathogens/article?id=10.1371/journal.ppat.1004900)

This dataset was generated by performing shotgun sequencing of total RNA from the liver of a boa constrictor that was diagnosed with [inclusion body disease](https://en.wikipedia.org/wiki/Inclusion_body_disease). 

![Python with IBD](IBD.png)

You also used cutadapt to trim off low quality and adapter-derived parts of reads.


### Downloading the boa constrictor (mitochondrial) genome.

The NGS data we are working with came from boa constrictor liver RNA.  We will map the reads in this dataset to the boa constrictor mitochondrial genome sequence to demonstrate read mapping.

First, we need to *find* the boa constrictor mitochondrial genome sequence.  As usual, there are few ways we could go about this:

1. navigate through the NCBI [Taxonomy database](https://www.ncbi.nlm.nih.gov/taxonomy/)
2. navigate through the NCBI [Genome database](https://www.ncbi.nlm.nih.gov/genome/)
3. navigate through another genome database, like [Ensembl](http://www.ensembl.org/index.html) or [UCSC](https://genome.ucsc.edu/)
4. google 'boa constrictor genome sequence'  (not a terrible way to do it)

- Let's choose option 1, and go through the NCBI Taxonomy database.  Navigate to https://www.ncbi.nlm.nih.gov/taxonomy/
   - Search for `boa constrictor`.
   - Click on Boa constrictor link, then click the Boa constrictor link again
   - You should see a table in the upper right corner showing linked records in various NCBI (Entrez) databases.
   - Click on the `Nucleotide` link in that table to go to boa constrictor sequences in the NCBI Nucleotide database

   - There are ~600 boa constrictor nucleotide sequences in this database.  We want the mitochondrial genome, which happens to be the only nucleotide sequence in the NCBI RefSeq database.
   - Click on "RefSeq" filter on the left hand side of the page.
   - You should see a link to the boa constrictor mitochondrial genome sequence.
   - Click on this 'NC_007398.1' RefSeq link

Now we need to download the sequence.  We'll do this through the browser.  In the upper right hand corner of the page, note the 'Send' drop down

- Click Send->Complete Record->File->Format->FASTA->Create File

You should have downloaded a fasta file of ~19 kb, named sequence.fasta, or something like that.

Now download the sequence in GenBank format too.  Note that this file is larger (~42 kb), because it contains annotation as well as the actual sequence.

Note that the downloaded files have unhelpful names: `sequence.fasta` and `sequence.gb` or similar.  You need to do two things with these downloaded files:

1. Rename these files to something more descriptive than sequence.fasta and sequence.gb.  Rename these files to:

- boa_mtDNA.fasta
- boa_mtDNA.gb

2. We want these files in Geneious too.  Drag them into Geneious:
 - Open Geneious
 - Create a new folder in Geneious
 - Drag and drop these files into Geneious

3. Transfer these files to the thoth01 server so we can work with them there.  To do this use sftp:

```
# change to the directory on your laptop where these downloaded files are
cd directory_where_your_files_are

# open an sftp connection to thoth01
sftp your_eid@thoth01.cvmbs.colostate.edu

# once connected via sftp, use the put command to transfer the files
put boa_mtDNA*
```

Now open an ssh connection to thoth01:
```
ssh your_eid@thoth01.cvmbs.colostate.edu
```

Once connected to thoth01, use the `mv` command to move these files to the directory where your trimmed fastq files are.


### Once you have these files on the thoth01 server

Change to the directory where your trimmed fastq files are and make sure these new boa_mtDNA files are in the same directory.

Let's inspect the contents of these files.  You can use the cat (or less) commands to output the contents of these files:
```
# cat outputs the contents of a file all at once
cat boa_mtDNA.fasta
cat boa_mtDNA.gb

# less allows you to page through files
less boa_mtDNA.fasta
less boa_mtDNA.gb
```

Hint: press `space` to advance a page in less and press `q` to exit

What is the difference between the genbank and FASTA format versions of this sequence?


### Create a bowtie index from the boa constrictor mitochondrial genome sequence

Read mapping tools map reads very quickly because they use pre-built indexes of the reference sequence(s).  We'll use the [Bowtie2](http://www.nature.com/nmeth/journal/v9/n4/full/nmeth.1923.html) mapper. Bowtie2 has a nice [manual](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml) that explains how to use this software. 

There are a variety of other good read mapping tools, such as [BWA](https://github.com/lh3/bwa) and [minimap2](https://github.com/lh3/minimap2), which works well for long read data or long sequences.

The first step will be to create an index of our reference sequence (the boa constrictor mitochondrial genome).

Make sure you are in the right directory (our working directory):
```
pwd
```

Now confirm that the boa constrictor mtDNA sequence file is there and in FASTA format: 
```
# should see: boa_mtDNA.fasta and 2 trimmed fastq (SRR1984309_1_trimmed.fastq SRR1984309_2_trimmed.fastq)
ls -lh    
```

Now, we'll use the bowtie2-build indexing program to create the index.  This command takes 2 arguments: 
(1) the name of the fasta file containing the sequence(s) you will index
(2) the name of the index (can be whatever you want)

```
# create a bowtie2 index.  Name it boa_mtDNA_bt_index
bowtie2-build boa_mtDNA.fasta boa_mtDNA_bt_index 
```

Confirm that you built the index.  You should see six files with names ending in bt2, like boa_mtDNA_bt_index.3.bt2
```
ls -lh
```

Note that this index building went very fast for a small genome like the boa mtDNA, but can take much longer (hours) for Gb-sized genomes.


### Mapping reads in the SRA dataset to the boa constrictor mitochondrial genome 

Now that we've created the index, we can map reads to the boa mtDNA.  We'll map our cutadapt-trimmed paired reads to this sequence, as follows:

```
bowtie2 -x boa_mtDNA_bt_index \
   -1 SRR1984309_1_trimmed.fastq \
   -2 SRR1984309_2_trimmed.fastq \
   --no-unal \
   --threads 8 \
   -S SRR1984309_mapped_to_boa_mtDNA.sam
```

Let's deconstruct this command line:

| Part | Purpose |
| ---- | ------- | 
| bowtie2 | name of command |
| -x boa_mtDNA_bt_index | -x: name of index you created with bowtie2-build |
| -1 SRR1984309_1_trimmed.fastq | name of the paired-read FASTQ file 1 |
| -2 SRR1984309_2_trimmed.fastq | name of the paired-read FASTQ file 2 |
| --no-unal | don't output unmapped reads to the SAM output file (will make it _much_ smaller |
| --threads 8 | since the server has multiple processers, run on 8 processors to go faster |
| -S SRR1984309_mapped_to_boa_mtDNA.sam   | name of output file in [SAM format](https://en.wikipedia.org/wiki/SAM_(file_format)) |


---
:question: **Questions:**
- What percentage of reads mapped to the boa mitochondrial genome?
- Does this make sense biologically?  Remember that this is total RNA from snake liver tissue.  Explain.
---

The output file `SRR1984309_mapped_to_boa_mtDNA.sam` is in [SAM format](https://en.wikipedia.org/wiki/SAM_(file_format)).  This is a plain text format, so you can look at the first 20 lines by running this command:


```
head -20 SRR1984309_mapped_to_boa_mtDNA.sam      
```

You can see that there are several header lines beginning with `@`, and then one line for each mapped read.  See [here](http://genome.sph.umich.edu/wiki/SAM) or [here](https://samtools.github.io/hts-specs/SAMv1.pdf) for more information about interpreting SAM files.

---
:question: **Answer the following questions about the first mapped read:**
- What position in the mtDNA sequence did it map to?
- What is the mapping quality for this read's mapping?
- What does this mapping quality score indicate?  
---


### Visualizing aligned (mapped) reads in Geneious

Geneious provides a nice graphical interface for visualizing the aligned reads described in your SAM file.   Other tools for visualizing this kind of data include [IGV](http://software.broadinstitute.org/software/igv/) and [Tablet](https://ics.hutton.ac.uk/tablet/)

First, you need to transfer the SRR1984309_mapped_to_boa_mtDNA.sam file from thoth01 to your computer.  Use sftp or cyberduck to do this.

Second, you need to have your reference sequence in Geneious, preferably with annotations.  You can do this 2 ways:

1. Drag and drop the boa_mtDNA.gb file into a folder in Geneious
2. Download the file directly into Genious, using the NCBI->Nucleotide interface (search for NC_007398.1).  Once downloaded, drag from the NCBI download folder into another folder in Geneious.  

Once you have the boa constrictor mitochondrial genome in a folder in Geneious, you can drag and drop the SAM file that bowtie2 output into the same folder.  Geneious will tell you that it 'can't find the sequence it needs in the selected file'.  It is telling you it is trying to find the reference sequence to which you aligned reads.  Answer: 'Find a sequence with the same name in this Geneious folder' or 'Use one of the selected sequences' (after selecting the boa mtDNA sequence).

- A few Geneious tips:
  - Enlarge the Geneious window so that it fills the screen
  - Click View->Expand Document View to enlarge the alignment
  - Try playing with the visualization settings in the panels on the right of the alignment

---
:question: **Questions to consider when viewing the alignment:**
- What is the average coverage depth across the mitochondrial genome?
- Is the coverage even across the mitochondrial genome?
- Would you expect coverage to be even across the genome?  (Recall that this data is derived from total RNA from liver tissue). 
- Are the mitochondrial genes expressed evenly?
- Are there any variants between this snake's mitochondrial genome sequence and the boa constrictor reference sequence?
- Is it expected that there are variants between these reads and this reference sequence?  Explain your answer.
- Can you distinguish true variants from sequencing errors?
- In general, how can you distinguish true variants from sequencing errors?
- Is it possible that reads that derive from the boa constrictor nuclear genome are mapping to this sequence?
- How would you prevent nuclear reads from mapping to the mitochondrial genome?
- Can you identify mapped read pairs?
---
