# NTM WGS analysis course

## Species

The first thing we often need to do is to identify the species of our sample. There are a number of ways to do this with whole genome sequencing (WGS) data including:

* Rapid taxonomic tools that work from fastq data
* Assembly followed by ANI comparison to reference

Tools that work straight from fastq data have the advantage that they work straight from raw sequence data. They can also easily identify if you have multiple species. They can work on very low quantities of sequence data. Popular tools include kraken2, sylph and sourmash.

Assembly-based methods require you to do some data processing first (quality-control and assembly). The assembled sequence can then be compared using several metrics including average nucleotide identity (ANI). Generally, if you identify a reference sequence >= 95% ANI to your sample it can be considered the same species. 

## Drug resistance

Drug resistance in NTM can be acquired through the evolution of mutations on genes coding for drug targets. Resistance can also occur due to the presence genes coding for drug metabolising enzymes. If we know the mechanisms, we can use WGS data to identify the presence of both drug resistance mutations and genes.

## WGS data

WGS data can be produced by different platforms/instruments. Popular platforms used these days include Illumina and Oxford nanopore technologies (ONT). Both these platforms produce many (thousands or millions) of sequences that should cover the genome of interest. These sequences are often called reads. The Illumina platform produces short sequences (150-300 bp) and ONT can produce long reads (kilobases). Additionally, Illumina sequences the input fragments at both ends and so the resulting data will be split into two files (often called the forward and reverse reads). This is often termed paired-end sequencing

```
Forward           -------->
Fragment          ---------------------------
Reverse                             <--------

Gap not sequenced           |<----->|
```

## Next

The next sections will introduce you to sequence analsis using ntm-profiler and other commandline tools. Enjoy!