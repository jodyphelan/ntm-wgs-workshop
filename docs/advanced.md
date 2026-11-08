# Learning by example

We will now demonstrate how to investigate NTM species and drug resistance using several example cases.

## Sample relatedness

You have been given sequence data for four _M. intracellulare_ isolates and have been taked with identifying if they are linked to an outbreak.

Isolates IDs:

* SAMN41229600
* SAMN41229617
* SAMN41229627
* WMW1085

### Basic analysis

!!! task
    Run ntm-profiler on the following using either the web interface or CLI and look at the results. Then have a go at answering the questions below

!!! question

    Try to answer the following quesitons before moving on:
    
    * Which species do you observe?
    * Is there any subspecies classification?
    * Do you see any resistance?
    * With the data you have, do you think they are linked?

### High resolution clustering

While some indicate they may be part of the [HCU outbreak](TODO), it can be useful to see how distant the isolates are to a higher resolution. SNP distance is a simple metric used to measure how similar two genomes are. The logic is that if two isolates share a common ancestor very recently, which happens when you have an outbreak, they will have had less time to acumulate mutations than if they are not linked and share an ancestor further back in time. To look at SNP distance, we first need to create a consensus genome for each isolates. NTM-Profiler has this functionality but it is only available via the CLI. To create a consensus genome, variants must be called across the whole genome and as a result the analysis will take longer to run. For this reason it is not enabled by default. To enable it simply add `--consensus` to your command. For example:

```
ntm-profiler profile -1 /data/mount/ntm-ngs-data/ntm-course/fastq/SAMN41229600_1.fastq.gz -2 /data/mount/ntm-ngs-data/ntm-course/fastq/SAMN41229600_2.fastq.gz -p SAMN41229600 -t 4 --consensus
```


Run the command with consensus genome calling enabled for all the genomes. After running the command you should notice files ending in `.consensus.fa`. This consensus uses the reference genome as a template and then inserts the sample-specific SNPs into it. As they will all be the same length, we can concatenate them all together into the same file to create an alignment. These genomes will not include any indels but since we just want to calculate SNP-distance this is fine. 

```
cat SAMN41229600.consensus.fa SAMN41229617.consensus.fa SAMN41229627.consensus.fa WMW1085.consensus.fa > alignment.fa
```

Next we can should calculate the number of SNPs between each isolate. We can use a tool called `snp-dists` to do this. But first we will need to change to a different conda environment. Change to the `analysis` environment with the following command:

```
micromamba activate analysis
```

Now we can run `snp-dists` on the alignment:

```
snp-dists alignment.fa > distance.tsv
```

This will create a matrix in TSV format. You can open this file on the terminal or on your system if you have access to excel or similar software.

!!! question
    * Do the SNP distances correspond to the subspecies assignemnts you saw?
    * Which isolates are the most closely related to each other?
    * Are any isolates part of an outbreak?

### Phylogenetic trees

The alignment can also be used to create phylogenetic trees. While this is not very informative in this case as we only have 4 samples, we will use it for demonstration purposes. You can run `iqtree` or any tree building tool using the `alignment.fa` file that you created before. We will use `iqtree`:

```
iqtree -s alignment.fa -st DNA
```

The phylogenetic tree will be output in a file called `alignment.fa.treefile`. This can be visualised in tools like [iTOL](TODO). Open the website, click on upload and select the tree file. 

!!! question
    Does the clustering correspond with the observed SNP distance values?
    
    Hint: hit `m` on your keyboard to perform midpoint rooting and will be easier to interpret the tree.

## Novel species

Three cases of pulmonary TODO. The isoaltes were cutured and sequenced with Illumina technology. You have been tasked with identifying the species.

Isolate IDs:
* WMW1108
* WMW1089
* WMW1097

!!! task
    Run ntm-profiler via the web interface or the CLI and note the species reported.

!!! question
    * Are the the same species?
    * If do you have enough data to make an accurate prediction? 

### Further analysis

NTM-Profiler is unable to report a species. There may be several reasons why this is the case including:

* Too little data to make a prediction
* If the isolate belongs to a non-mycobacterial species
* If the isolatee belongs to a novel mycobacterial species

By looking at the number of bases in the reads it should be possible to rule out insifficient data. To distinguish between the second and third scenarios we will have to some follow up analysis. To investigate further we would like to compare our isolates against all other mycobacterial species. NTM-Profiler only reports matches with ANI > 95% so we will need to use other tools to further investigate.

#### Generate an assembly

To generate an assebmly we will use a fast tool called `megahit`. We chose this tool because it is fast, however `spades` or `shovill` are more commonly used for bacterial isolate assembly. Run this command to generate an assembly:

```
megahit -o WMW1108_asm -1 /data/mount/ntm-ngs-data/ntm-course/fastq/WMW1108_1.fastq.gz -2 /data/mount/ntm-ngs-data/ntm-course/fastq/WMW1108_2.fastq.gz
```

This will create an output folder called `WMW1108_asm` which will contain a file called `final.contigs.fa`. Also note the statement printed when the tool is finished. It will report a few statistics including:
* number of contigs - fewer means more contiguous assembly
* total assembly size - for a pure isolate this should equate to the genome size
* N50 - similar to a median value for contig size which can be used to judge how fragmented an assembly is (higher is better) 

!!! task
    Repeat this for the other two samples and try to answer the following questions
    * How many contigs in each assebmly?
    * Are the genome sizes similar?
    * Are the genome sizes what you would expect for Mycobacteria?


When you are done, create a folder and copy the assemblies into the folder:

```
mkdir assemblies
cp WMW1108_asm/final.contigs.fa assemblies/WMW1108.fa
cp WMW1089_asm/final.contigs.fa assemblies/WMW1089.fa
cp WMW1097_asm/final.contigs.fa assemblies/WMW1097.fa
```

#### Calculate ANI

Now we can answer the question of if they are the same species or not. We are going to use a tool called skani to estimate the ANI between our sequences:

```
skani triangle  assemblies/*
```

This will calculate pairwise ANI and present the lower triangle of the matrix. 

!!! tip
    We used a wildcard `*` here to select all files in the directory

!!! question
    * Are these isolates the same species?
    * Remember, two sequences with > 95% ANI are considered to be the same species.

Now we know they are the same species, let's compare against all other mycobacterial species. To do this, unzip the `ntm-representatives.tgz` file which contains a representative genome sequence for each species in GTDB. Unzip with the following command:

```
tar -xvf /data/mount/ntm-ngs-data/ntm-course/ntm-representatives.tgz 
```

The files will unzip to a folder called `representative_asm`. Have a look at the contents using the following command:

```
ls representative_asm
```

There should be one sequence for each species. 

#### Generate distance tree

We can use a diferent distance metric called mash distance which can be calculated across more distantly related sequences. Mash distances can then be converted into a distance-based tree and used to visualise how our sequences compare both to each other and to all other mycobacteial sequences. This analysis workflow is implemented in a software called `mashtree`. You can run it like this:

```
mashtree --numcpus 4 --outtree mashtree.nwk assemblies/* representative_asm/*
```

This will take a couple of minutes as it is doing a lot of comparisons (853*853 to be exact!). When it is finished you should have the tree in the `mashtree.nwk` file. This is a newick format file which can be visualised with tools such as [iTOL](TODO). Open the website and upload the newick file. By default it will use the name of the file as the names of the leaves in the tree. To make this more informative - drag and drop onto the tree the file named `species_text_labels.txt`  in the `metadata` folder. This will annotate the species names. 

!!! question
    * Where do our isolates cluster? 
    * Which is the most closely related species?
    * What is the ANI with the closest relative? (hint: use `skani` like before)


## Mixed species

Have a look at sample **WMW1130**. Run ntm-profiler on it and note the species we get. 


In this case we seem to get a mixture between two species. This can sometimes happen when:
* There is a legitimate mixture of two different species in your sample
* A novel species is present which has large segments of sequence similar to other species



One way to investigate this is to perform a genome assembly. If multiple species are present you could assemble both species and the resulting total assebmly size will be much larger than any single genome could be.

!!! task
    Assemble the genome using `megahit` as you did before.

!!! question
    * What is the total genome size? 
    * Is this consistant with one or two species?

This is a critical step when consider whether you have mixtures or contamination. If you jump straight to `mashtree` here it might look like you have a novel species

!!! task
    Run `mashtree` like before.

!!! question
    * Comment on the clustering pattern. Which species do you think it belongs to?
    * Calculate the ANI to closely related straings (tip: use `skani`)

The mashtree analysis assumes only one strain present and will because of this may cluster your assembly in between different species as it is pulled in different directions. This can be misleading so it is always important to investigate using multiple angles.

!!! task
    Perform a similar analysis for **WMW1101**