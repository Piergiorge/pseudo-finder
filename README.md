# Pseudo finder

**Warning: This repository is still a work in progress!**

Pseudo finder is a Python3 script that detects pseudogene candidates [https://en.wikipedia.org/wiki/Pseudogene] from annotated genbank files of bacterial and archaeal genomes.

It was tested mostly on genbank (.gbf/.gbk) files annotated by Prokka [https://github.com/tseemann/prokka] with the --compliant flag (i.e. including both /gene and /CDS annotations). We recommend several rounds of Pilon [https://github.com/broadinstitute/pilon/wiki] polishing with Illumina reads if you're interested to find pseudogenes in minION/PacBio assemblies. However, Pseudo-finder can be alternatively also used to find sequencing errors (indels) breaking genes in MinION/PacBio-only assemblies.

There are alternative programs for pseudogene finding and annotation (e.g. the NCBI Prokaryotic Genome Annotation Pipeline [https://www.ncbi.nlm.nih.gov/genome/annotation_prok/]), but to the best of our knowledge, none of them is open source and allows fine-tuning of parameters.


## Authors
Mitch Syberg-Olsen & Filip Husnik

University of British Columbia, Vancouver, Canada

## Versions and changes

Read the ChangeLog.txt [https://github.com/filip-husnik/pseudo-finder/blob/master/ChangeLog.txt] for major changes or look at Github commits for everything else [https://github.com/filip-husnik/pseudo-finder/commits/master].

## Citation

Pseudo finder is still a work in progress, so there is no official publication. If it was useful for your work, you can cite it as: M. Syberg-Olsen & F. Husnik 2018: Pseudo finder, GitHub repository: https://github.com/filip-husnik/pseudo-finder/.

Please also cite various dependencies used by Pseudo finder (listed below).

## Getting Started

These instructions will (hopefully) get you the pipeline up and running on your local machine.

### Prerequisites

Installation requirements: python3, pip3 (or any other way how to install python libraries, e.g. conda or easy_install), biopython, ncbi-blast+

Databases: NCBI-NR (non-redundant) protein database (or similar such as SwissProt) formatted for blastP/blastX searches.

Input files: A genome sequence with gene annotations in the genbank (.gbf/.gbk) format.


### Installing

A step by step series of commands to install all system dependencies:

Installation of python3, pip3, git (optional), and ncbi-blast+ on Ubuntu (as an administrator):
```
sudo apt-get update

sudo apt-get install python3
sudo apt-get install python3-pip
sudo apt-get install ncbi-blast+
sudo apt-get install git
```

Installation of 3rd party python3 libraries on Ubuntu (as an administrator):
```
sudo pip3 install biopython
```

*Alternative installation with conda (no root access required) is also possible:*

```
#install Python3 miniconda https://conda.io/miniconda.html
conda install -c conda-forge biopython
conda install -c bioconda blast
```

Clone the up to date pseudo_finder.py code from github (no root access required):

```
git clone https://github.com/filip-husnik/pseudo-finder.git
```
Or download a stable release:

```
# No stable version released yet.
```


## Preparing your genome file

We recommend genbank (.gbf/.gbk) files generated by Prokka [https://github.com/tseemann/prokka] with the --compliant and --rfam flags. Annotating rRNAs, tRNAs, and other ncRNAs in Prokka is recommended to eliminate any false positive 'pseudogene' candidates. ORFs overlapping with non-coding RNAs such as rRNA can be sometimes misannotated in databases as 'hypothetical proteins'.

```
prokka --compliant --rfam contigs.fa
```
## How does pseudo finder detect pseudogene candidates?

This flowchart shows all steps of the pseudofinder pipeline.

![alt text](https://github.com/filip-husnik/pseudo-finder/blob/master/pseudofinder_flowchart.png)

## Running pseudo finder

As with any other python script, there are two ways how to run it.

```
# Call it directly with python3.
python3 pseudo_finder.py

# Or make the file executable and then rely on its shebang line [#!/usr/bin/env python3].
chmod u+x
./pseudo_finder.py
```

Providing input files.
```
# Run the full pipeline on 16 processors (for BlastX/BlastP searches).
# Unless you have a $BLASTDB environmental variable set on your system, you have to provide a full path to the NR database.
python3 pseudo_finder.py --genome GENOME.GBF --output PREFIX --database /PATH/TO/NR/nr --threads 16 

# Run only pseudogene detection (e.g. when blast output files are already available from a previous run).
python3 pseudo_finder.py --genome GENOME.GBF --output PREFIX --blastp BLASTPFILE.TSV --blastx BLASTX.TSV
```

```
# All command line arguments.
optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit

Required arguments:
  -g GENOME, --genome GENOME
                        Please provide your genome file in the genbank format.
  -op OUTPREFIX, --outprefix OUTPREFIX
                        Specify an output prefix.

Required arguments if BLAST files are not provided:
  -db DATABASE, --database DATABASE
                        Please provide name (if $BLASTB is set on your system) or absolute path of your blast database.

Required arguments if providing BLAST files:
  -p BLASTP, --blastp BLASTP
                        Specify an input blastp file.
  -x BLASTX, --blastx BLASTX
                        Specify an input blastx file.

Adjustable parameters:
  -of OUTFORMAT, --outformat OUTFORMAT
                        Specifies which style of output to write. Default is 1.
                        See below for explanation.
  -t THREADS, --threads THREADS
                        Please provide total number of threads to use for blast, default is 4.
  -i INTERGENIC_LENGTH, --intergenic_length INTERGENIC_LENGTH
                        Please provide length of intergenic regions to check, default is 30 bp.
  -l LENGTH_PSEUDO, --length_pseudo LENGTH_PSEUDO
                        Please provide percentage of length for pseudo candidates, default is 0.60 (60%). 
                        Example: "-l 0.50" will consider genes that are less than 50% of the average length of similar genes.
  -s SHARED_HITS, --shared_hits SHARED_HITS
                        Percentage of blast hits that must be shared in order to join two nearby regions, default is 0.30 (30%). 
                        Example: "-s 0.50" will merge nearby regions if they shared 50% of their blast hits.
  -e EVALUE, --evalue EVALUE
                        Please provide e-value for blast searches. Default is 1e-4.
  -d DISTANCE, --distance DISTANCE
                        Maximum distance between two regions to consider joining them. Default is 1000.
  -hc HITCAP, --hitcap HITCAP
                        Maximum number of allowed hits for BLAST. Default is 15.
                        
  -ce, --contig_ends    Forces the program to include intergenic regions at contig ends. If not specified,
                        the program will ignore any sequence after the last ORF on a contig.
  -it INTERGENIC_THRESHOLD, --intergenic_threshold INTERGENIC_THRESHOLD
                        Number of BlastX hits needed to annotate an intergenic region as a pseudogene.
                        Calculated as a percentage of maximum number of allowed hits (--hitcap).
                        Default is 0.3.

```
## Output of pseudo finder
Every run should result in two or more output files: a summary log.txt file and .gff or .faa files.

log.txt: The log file includes basic statistics about pseudogene candidates detected.

Explanation of output choices
'--outformat 1': GFF file containing only ORFs flagged as pseudogenes.
	The .gff file can be used to overlay the original annotation (we use the Artemis genome browser [http://www.sanger.ac.uk/science/tools/artemis]) with predicted pseudogene candidates.

'--outformat 2': GFF file containing only ORFs not flagged as pseudogenes. 
	*Currently only contains nucleotide positions, without qualifier information.*

'--outformat 3': FASTA file containing only ORFs flagged as pseudogenes. 
	*Not yet implemented*

'--outformat 4': FAA file containing only ORFs not flagged as pseudogenes.
	This file can be used for any downstream analysis of functional genes. 

*Note: Outputs can be combined. If '-of 12' is specified, files '1' and '2' will be written.

## Contributing

We appreciate any critical comments or suggestions for improvements. Please raise issues or submit pull requests.

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* This code was inspired mostly by work on bacterial symbionts in early stages of becoming intracellular and strictly host-associated. This ecological shift releases selection pressure ('use it or loose it') on many genes considered essential for free-living bacteria, so relatively recent symbionts can have over 50% of their genes pseudogenized.

## References

**Basic information about bacterial pseudogenes:**

Recognizing the pseudogenes in bacterial genomes: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1142405/

Taking the pseudo out of pseudogenes: https://www.ncbi.nlm.nih.gov/pubmed/25461580

**Several examples from the Sodalis clade showing how important is pseudogene annotation for bacteria in a nascent stage of symbiosis:**

Mobile genetic element proliferation and gene inactivation impact over the genome structure and metabolic capabilities of Sodalis glossinidius, the secondary endosymbiont of tsetse flies: https://www.ncbi.nlm.nih.gov/pubmed/20649993

A novel human-infection-derived bacterium provides insights into the evolutionary origins of mutualistic insect–bacterial symbioses: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3499248/

Genome degeneration and adaptation in a nascent stage of symbiosis: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3914690/

Repeated replacement of an intrabacterial symbiont in the tripartite nested mealybug symbiosis: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5027413/

Large scale and significant expression from pseudogenes in Sodalis glossinidius - a facultative bacterial endosymbiont: https://www.biorxiv.org/content/early/2017/07/23/124388

## Wish list
There are several additional features we'll try to include in the script in the near future:

1. Check if the ORFs called as pseudogenes do not represent individual protein domains that can exists and evolve independently of the rest of the original multi-domain protein chain.
2. Include an optional analysis of cryptic pseudogenes based on dN/dS ratios (PAML?...) when there are closely related genomes available.
3. Include an optional analysis when there are RNA-Seq data available.
4. Fine tune pseudogene finding for mobile elements such as transposases.
5. Visualize results by a scatter plot of all genes/pseudogenes (dN/dS, GC content, expression, length ratio, ...).
6. Sometimes ORFs are predicted by mistake on the opposite strand (e.g. in GC-rich genomes), check regions with ORFS with no blastP hits by blastX.

Please suggest any additional features here: [https://github.com/filip-husnik/pseudo-finder/issues].
