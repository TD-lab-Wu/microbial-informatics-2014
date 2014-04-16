Ortholog Detection with OrthoMCL
================================

This tutorial covers ortholog detection using [OrthoMCL](http://orthomcl.org/common/downloads/software/v2.0/).  Instead of running all the software for OrthoMCL manually, we will take advantage of the [OrthoMCL Pipeline](https://github.com/apetkau/orthomcl-pipeline) to automate this process.

Introduction
============

Tutorial
========

The following steps show how to run OrthoMCL on an example set of data and examine some of the results in a Venn Diagram.

Step 1: Obtaining input Data
----------------------------

The input data for OrthoMCL consists of a set of genes.  This can be downloaded from:

	$ wget http://url-to-data/annotations-cholera.tar.gz
	$ tar -xvvzf annotations-cholera.tar.gz

This will extract the annotated genomes into a directory __annotations/__.  This directory looks as follows:

	$ ls annotations/
	2010EL-1749.faa  2010EL-1786.ffn  2010EL-1798.faa  2011EL-2317.ffn  3554-08.faa  C6706.ffn  VC-14.faa  VC-15.ffn  VC-19.faa  VC-1.ffn   VC-26.faa  VC-6.ffn
	2010EL-1749.ffn  2010EL-1796.faa  2010EL-1798.ffn  2012V-1001.faa   3554-08.ffn  VC-10.faa  VC-14.ffn  VC-18.faa  VC-19.ffn  VC-25.faa  VC-26.ffn
	2010EL-1786.faa  2010EL-1796.ffn  2011EL-2317.faa  2012V-1001.ffn   C6706.faa    VC-10.ffn  VC-15.faa  VC-18.ffn  VC-1.faa   VC-25.ffn  VC-6.faa

The files __*.faa__ contains the genes as amino acid sequences.  The files __*.ffn__ contain the genes as nucleotide sequences.  For example:

	$ head annotations/2010EL-1749.faa
	>2010EL-1749_00001 Stalked cell differentiation-controlling protein
	MDARLFDNTQTLRASVLCGLSFFWALIAFLMALINFWSTRLVELASLELVCAFYSLYIYS
	LAKRRIHTKQQVYLYLFILTGTTLFATYMKPLMMGVYIWSCFVPILFYIFTSARFAFVTS
	...

	$ head annotations/2010EL-1749.ffn
	>2010EL-1749_00001 Stalked cell differentiation-controlling protein
	ATGGATGCTAGGTTATTTGACAATACACAAACGCTTCGAGCTTCAGTGCTATGCGGCCTA
	AGTTTCTTTTGGGCTTTGATCGCTTTCTTGATGGCGCTGATCAATTTCTGGTCAACACGG
	...

Step 2: Setup OrthoMCL Database Configuration
---------------------------------------------

OrthoMCL requires the use of a database, such as [MySQL](http://www.mysql.com/), to do some of the analysis.  This requires a bit of manual setup, but a lot of this has been done for you.  A configuration file with information on how to connect to the database still needs to be generated.  This can be done with the following command:

	$ /opt/orthomcl-pipeline/scripts/setup_database.pl --user orthomcl --password orthomcl --host localhost --database orthomcl > orthomcl.conf
	Connecting to database orthomcl on host localhost with user orthomcl ...OK

This generates a file, __orthomcl.conf__ which contains the neccessary database connection information and some default settings for OrthoMCL.  This file looks as follows.

	coOrthologTable=CoOrtholog
	dbConnectString=dbi:mysql:orthomcl:localhost:mysql_local_infile=1
	dbLogin=orthomcl
	dbPassword=orthomcl
	dbVendor=mysql 
	evalueExponentCutoff=-5
	inParalogTable=InParalog
	interTaxonMatchView=InterTaxonMatch
	oracleIndexTblSpc=NONE
	orthologTable=Ortholog
	percentMatchCutoff=50
	similarSequencesTable=SimilarSequences

For a more detailed description of this file please see the [OrthoMCL](http://orthomcl.org/common/downloads/software/v2.0/UserGuide.txt) documentation.

For more details on how to setup and install the OrthoMCL Pipeline please see the [install](https://github.com/apetkau/orthomcl-pipeline/blob/master/INSTALL.md) documentation.

Step 3: Renaming input files
----------------------------

The OrthoMCL Pipeline takes as input the __*.faa__ amino acid sequence files, but assumes its input files end with the __*.fasta__ extension.  To properly rename these files please use the following command.

	$ prename 's/\.faa/\.fasta/' annotations/*.faa

This rename all the __*.faa__ files within the __annotations/** directory.  These will look like:

	$ ls annotations
	2010EL-1749.fasta  2010EL-1786.ffn    2010EL-1798.fasta  2011EL-2317.ffn   3554-08.fasta  C6706.ffn    VC-14.fasta  VC-15.ffn    VC-19.fasta  VC-1.ffn     VC-26.fasta  VC-6.ffn
	2010EL-1749.ffn    2010EL-1796.fasta  2010EL-1798.ffn    2012V-1001.fasta  3554-08.ffn    VC-10.fasta  VC-14.ffn    VC-18.fasta  VC-19.ffn    VC-25.fasta  VC-26.ffn
	2010EL-1786.fasta  2010EL-1796.ffn    2011EL-2317.fasta  2012V-1001.ffn    C6706.fasta    VC-10.ffn    VC-15.fasta  VC-18.ffn    VC-1.fasta   VC-25.ffn    VC-6.fasta

Step 4: Running OrthoMCL
------------------------

In order to run OrthoMCL please use the following command.

	$ orthomcl-pipeline -i annotations/ -o orthomcl-output -m orthomcl.conf --nocompliant
	Starting OrthoMCL pipeline on: Fri Apr 11 15:39:19 2014
	Git commit: a3999872a160994b9e8eacb95163c24b4dd78bd8
	
	
	=Stage 1: Validate Files =
	...

If you get the message:

	Warning: some tables exist already in database dbi:mysql:orthomcl:localhost:mysql_local_infile, user=orthomcl, database_name=orthomcl. Do you want to remove (y/n)?

Please answer __yes__.  This will simply delete any old results stored in the database so that OrthoMCL can run properly.

When the pipeline is finished you should see the following output.

	Orthomcl Pipeline ended on Fri Apr 11 16:12:02 2014
	Took 19.23 minutes to complete
	Parameters used can be viewed in orthomcl.conf and /home/aaron/Projects/MicrobialInformatics2014/orthomcl-tutorial/orthomcl-output/log/run.properties
	Groups file can be found in /home/aaron/Projects/MicrobialInformatics2014/orthomcl-tutorial/orthomcl-output/groups/groups.txt

Step 5: Looking at the Results
------------------------------

The main output from OrthoMCL is a file __orthomcl-output/groups/groups.txt__ which contains a list of potential orthologs among the entire input genome set, one set of orthologs per line.  This file looks as follows:

	$ head orthomcl-output/groups/groups.txt
	group_1: 2010EL-1749|2010EL-1749_00119 2010EL-1749|2010EL-1749_00120 ...
	group_2: 2010EL-1786|Vch1786_I1650 2010EL-1786|Vch1786_I2818 2010EL-1786|Vch1786_II0608 ...
	group_3: 2010EL-1786|Vch1786_I0915 2010EL-1786|Vch1786_II0814 2010EL-1786|Vch1786_II0916 ...
	...

Each gene within an ortholog group is separated by spaces.  So, for example, group_1 contains the gene `2010EL-1749_00119` from genome __2010EL-1749__ and the gene `2010EL-1749_00120` from genome __2010EL-1749__.

These correspond to the genes:

	$ grep -A1 '2010EL-1749_00119' annotations/2010EL-1749.fasta
	>2010EL-1749_00119 Flagellin D
	MAVNVNTNVAAMTAQRYLTGATNAQQTSMERLSSGFKINSAKDDAAGLQISNRLNVQSRG

	$ grep -A1 '2010EL-1749_00120' annotations/2010EL-1749.fasta
	>2010EL-1749_00120 Flagellin B
	MAINVNTNVSAMTAQRYLNGAADGMQKSMERLSSGYKINSARDDAAGLQISNRLTSQSRG

Notice how both these genes come from the same genome __2010EL-1749__.  This simply means that these two genes passed the cutoff criteria to be considered paralogs.