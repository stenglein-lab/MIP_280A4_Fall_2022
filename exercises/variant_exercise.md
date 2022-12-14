## Variant calling exercise

MIP 280A4 
---

### In this exercise, we will go through a variant calling pipeline. 

The data we'll use was generated from a pool of _Anopheles gambiae_ mosquitoes collected in Liberia.  The dataset contains reads mapping to a virus (Anopheles flavivirus).  We'll map the reads to the viral genome and use a variant caller to identify variants and determine their frequencies.

An _Anopheles gambiae_ mosquito
![Anopheles gambiae mosquito](./anopheles_gambiae.jpg)
image credit: Jim Gathany, USCDCP, CC-O

We will:

- copy the files we need to a new working directory
- make a bowtie index from the viral genome (reference) sequence
- map reads to the reference sequence using bowtie2
- use lofreq to call variants
- visualize and inspect variants in vcf output file and in Geneious

### Download files needed for exercise from github

First, we need to download some files.

Open the terminal and change directory to a directory of your choosing (make a new directory if you'd like).

The files for this exercise are in the directory `/home/data_for_classes/2021_MIP_280A4/`.  You need to copy this file into your current directory.

```
cp /home/data_for_classes/2022_MIP_280A4/variant_exercise_files.tar.gz .
```

The file extension `.tar.gz` is similar to .zip.  It means this is a compressed set of files.  

You'll often run across `.tar.gz` files when you're downloading data or software.  You can uncompress and extract this file using the tar command, as follows:

```
tar xvf variant_exercise_files.tar.gz
```

You should now see 4 new file (run the `ls` command):
```
Pool_filtered_R1.fastq   # paired read files containing already trimmed reads in FASTQ format
Pool_filtered_R2.fastq
viral_genome.fasta       # viral genome in FASTA format
viral_genome.gb          # viral genome in GenBank (annotated) format
```

### Create a bowtie index from the viral genome sequence

We're going to use bowtie2 to map reads in the dataset to the viral genome.  First, we need to create a bowtie2 index.  Remember how to do that?

```
# don't forget to activate the bio_tools conda environment
conda activate bio_tools

# build the index
bowtie2-build viral_genome.fasta viral_genome_bt_index 
```


### Mapping reads in the dataset to the viral genome sequence 

Now that we've created the index, we can map reads to it.  We'll use [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml) to do this, as follows

```
bowtie2 -x viral_genome_bt_index \
   -1 Pool_filtered_R1.fastq \
   -2 Pool_filtered_R2.fastq \
   --no-unal \
   --threads 18 \
   -S Pool_reads_aligned_to_viral_genome.sam
```

This command will map the paired reads in the 2 fastq files to the bowtie index you just created.  The output will be in the sam format file.

### Using lofreq to call variants

Now we have mapped reads.  There are a variety of variant callers.  We'll use the [lofreq](http://csb5.github.io/lofreq/) variant caller, because it is pretty easy to use and has performed reasonably well in benchmarking comparisons.

Most variant callers take as input sorted .bam format files, which are a compressed, sorted version of sam files.  We'll use the samtools program to convert our .sam output file from bowtie to .bam format:

```
# convert SAM -> BAM
samtools view -b Pool_reads_aligned_to_viral_genome.sam > Pool_reads_aligned_to_viral_genome.bam

# sort BAM 
samtools sort -T tmp -O 'bam' Pool_reads_aligned_to_viral_genome.bam  > Pool_reads_aligned_to_viral_genome.sorted.bam
```

Now, we'll run lofreq.  To learn more about how you could run lofreq, run:
```
lofreq                          # show general usage info
lofreq call                     # show general usage info for the call command in lofreq (actually does variant calling)
```

To call variants, you tell lofreq the name of the reference sequence (-f option), the name of the sorted bam file, and the name of the output file that will be created (-o option).  The output is in [VCF format](https://samtools.github.io/hts-specs/VCFv4.3.pdf):
```
lofreq call -f viral_genome.fasta Pool_reads_aligned_to_viral_genome.sorted.bam -o Pool_reads_aligned_to_viral_genome.vcf
```

Use less or cat to inspect the contents of your vcf file.  
```
less Pool_reads_aligned_to_viral_genome.vcf
```

#### Questions about this variant calling output:
- How many variants were identified (how many variants are described in the vcf lofreq output file)? What bash command did you run to determine this?  
- What is the allele frequency of the first described variant? 
- Is linkage between variants described in the vcf file? 
- Are the variants SNPs, or InDels?  Would you expect to see InDel variants here? 


### Visualize and inspect mapped data supporting called variants in Geneious

First, you need to get your reference sequence into Geneious, preferably with annotations.  Then you need to bring in the mapped reads (can bring in .sam or .bam format)

1. Transfer the viral_genome.gb and Pool_reads_aligned_to_viral_genome.sorted.bam files to your laptop.
2. Drag and drop the viral_genome.gb file into a folder in Geneious
3. Drag and drop the Pool_reads_aligned_to_viral_genome.sorted.bam file into the same folder in Geneious.  


#### Questions to consider about mapped reads in geneious:
- Can you identify individual variants called in your VCF file in Geneious?
- Are variant frequencies generally similar between Geneious and the VCF file?  Report the variant frequency as reported by Geneious and in the VCF file for one variant (include its position in the genome too).  
- Can you visually identify linked variants by looking at reads in Geneious?  How far apart do you think you can identify linked variants using this strategy?  
- Would long read sequencing be a good alternative approach to identify linked variants? Why or why not?  
- Are any of the variants non-synonymous (do they change the encoded amino acid)?  Provide an example of one such variant.
- Are these intrahost or interhost variants?  Explain your answer.
- Can you identify any InDel variants that _should_ have been called by lofreq that weren't called? (Hint: look near the beginning of the reference sequence). 
- We ran bowtie2 in end-to-end mode, which doesn't permit soft-trimming of read ends.  It forces the entire read to align.  Can you identify any ends that should have been trimmed but weren't?  (I.e. that contain low quality basecalls at their ends?) 

