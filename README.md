## Genome Download and Locus Export Tutorial
### Pre-Steps: Activating Anvi'o Environment and Setting the Working Directory
The Anvi'o installation that you performed previously is stored in a Conda environment. This must be activated each time the terminal is restarted using the following command:

`conda activate anvio-6.2`

Now you must set your working directory. This will be the folder that your terminal will search for input files and export output files.

**WINDOWS 10 USERS**: Set working directory to the Desktop using the following command, replacing "blakesanders" with your user profile name:

`cd /mnt/c/Users/blakesanders/Desktop`

**MAC OSX USERS**: Set working directory to the Desktop using the following command:

`cd Desktop`

**ALL USERS**: Now create a new folder for your working directory:

`mkdir Practice_Bioinformatics`

And set this new folder as your working directory:

`cd Practice_Bioinformatics`

### Step 1: Download *S. aureus* Reference Sequence

The easiest way to download genomes from NCBI is to use the program ncbi-genome-download. This can be used to download all sequences in a given species or reference/representative sequences only. Here, we will only download representative sequences for 5 different bacteria species:

~~~
ncbi-genome-download bacteria \
	-g "Staphylococcus aureus,Bacillus anthracis,Yersinia pestis,Chlamydia trachomatis,Salmonella enterica" \
	-R representative \
	-o Genomes \
	-m Sequence_Metadata.txt
~~~

> bacteria: positional argument specifying bacteria as search organism
> 
> -g: specify genus to download
> 
> -R: refeq database to pull from (all, reference, representative)
> 
> -o: output folder
> 
> -m: creates metadata file containing information about path location, related gene call files, etc. for each genome

### Step 2: Convert Fasta Files into Anvi'o Contigs Databases

To read the downloaded files into Anvi'o, the files must be reformatted. The script anvi-script-process-genbank-metadata was created exactly for this purpose:

~~~
anvi-script-process-genbank-metadata -m Sequence_Metadata.txt \
	-o Genomes \
	--output-fasta-txt Sequence_fasta.txt
~~~
> -m: metadata file that you created in the previous step
 
> -o: output folder

> --output-fasta-txt: file containing the paths to each genome fasta file; will be named what you provide

Proceeding on, there are 3 different ways that a Contigs Database can be created in Anvi'o. These are outlined in Steps 3.A-3.C

### Step 3.A: Generate Anvi'o Contigs Databases One at a Time
First, change the working directory to the new "Genomes" folder that was created from running the previous command:

`cd Genomes`

Quickly check that you're in the correct directory and confirm that you have all of the expected files:

`ls`

Now you can generate a Contigs Database for each genome. To do this, first use the anvi-gen-contigs-database program. You can use the gene calls provided by NCBI by using the --external-gene-calls flag:

~~~
anvi-gen-contigs-database -f Yersinia_pestis_GCF_003798225_1-contigs.fa \
	--external-gene-calls Yersinia_pestis_GCF_003798225_1-external-gene-calls.txt \
	-n Yersinia_CONTIGS \
	-o Yersinia-CONTIGS.db

anvi-gen-contigs-database -f Staphylococcus_aureus_subsp__aureus_DSM_20231_GCF_001027105_1-contigs.fa \
	--external-gene-calls Staphylococcus_aureus_subsp__aureus_DSM_20231_GCF_001027105_1-external-gene-calls.txt \
	-n Staphylococcus_CONTIGS \
	-o Staphylococcus-CONTIGS.db

anvi-gen-contigs-database -f Salmonella_enterica_GCF_001558355_2-contigs.fa \
	--external-gene-calls Salmonella_enterica_GCF_001558355_2-external-gene-calls.txt \
	-n Salmonella_CONTIGS \
	-o Salmonella-CONTIGS.db
	
anvi-gen-contigs-database -f Chlamydia_trachomatis_A_HAR_13_GCF_000012125_1-contigs.fa \
	--external-gene-calls Chlamydia_trachomatis_A_HAR_13_GCF_000012125_1-external-gene-calls.txt \
	-n Chlamydia_CONTIGS \
	-o Chlamydia-CONTIGS.db
	
anvi-gen-contigs-database -f Bacillus_anthracis_str___Ames_Ancestor__GCF_000008445_1-contigs.fa \
	--external-gene-calls Bacillus_anthracis_str___Ames_Ancestor__GCF_000008445_1-external-gene-calls.txt \
	-n Bacillus_CONTIGS \
	-o Bacillus-CONTIGS.db
~~~

Now that you have Contigs Databases for each genome, you can add in the functional annotations for each gene call from NCBI. This information includes the gene names for each annotated gene:

~~~
anvi-import-functions -c Yersinia-CONTIGS.db \
	-i Yersinia_pestis_GCF_003798225_1-external-functions.txt

anvi-import-functions -c Staphylococcus-CONTIGS.db \
	-i Staphylococcus_aureus_subsp__aureus_DSM_20231_GCF_001027105_1-external-functions.txt

anvi-import-functions -c Salmonella-CONTIGS.db \
	-i Salmonella_enterica_GCF_001558355_2-external-functions.txt

anvi-import-functions -c Chlamydia-CONTIGS.db \
	-i Chlamydia_trachomatis_A_HAR_13_GCF_000012125_1-external-functions.txt

anvi-import-functions -c Bacillus-CONTIGS.db \
	-i Bacillus_anthracis_str___Ames_Ancestor__GCF_000008445_1-external-functions.txt
~~~

As you can see, this process is tedious. The anvi-gen-contigs-database function is a major computational bottleneck, so running this by hand for every genome of interest is not ideal. To get around this, you can utilize a Bash loop to automatically run this command for all of your genomes.

### Step 3.B: Generate Anvi'o Contigs Databases Using a Bash Loop

Bash loops are great ways to run reptitive commands, allowing you to run a list of commands across several genomes while you step away from your computer. The following loop will generate the same end files as Step 3.A, but it will automate the process:

~~~
for GENOME in Yersinia_pestis_GCF_003798225_1 Staphylococcus_aureus_subsp__aureus_DSM_20231_GCF_001027105_1 Chlamydia_trachomatis_A_HAR_13_GCF_000012125_1 Salmonella_enterica_GCF_001558355_2 Bacillus_anthracis_str___Ames_Ancestor__GCF_000008445_1
do
	anvi-gen-contigs-database -f ${GENOME}-contigs.fa \
		--external-gene-calls ${GENOME}-external-gene-calls.txt \
		-n ${GENOME}_Contigs \
		-o ${GENOME}-CONTIGS.db
	anvi-import-functions -c ${GENOME}-CONTIGS.db \
		-i ${GENOME}-external-functions.txt
done
~~~

To run this loop, copy it into a new file in nano. The name must end in .sh:

`nano Contigs_Database_Loop.sh`

To save the file, press "Ctrl + X" and press "y" to overwrite the file. The nano controls can be found at the bottom of the screen. You now have a bash script saved in your working directory. To run the loop, run the following command:

`bash Contigs_Database_Loop.sh`

Bash loops can be powerful tools for autmating workflows, but Anvi'o natively includes an even more powerful pipeline tool: Snakemake.

### Step 3.C: Generate Anvi'o Contigs Databases Using a Snakemake Workflow

Snakemake is a tool that can be utilized similar to Bash loops, but, in it's implementation in Anvi'o, it relies on a .json file to specify the parameters that you want to utilize. Anvi'o uses different .json files for running different workflows. For creation of Contigs Databases, use the "contigs" workflow. To get this file, run the following command:

~~~
anvi-run-workflow -w contigs \
	--get-default-contig Contigs-Config.json
~~~

This command will create a file named "Contigs-Config.json" that includes the necessary Snakemake instructions for creating Contigs Databases. We typically end the name with "config" because this is a configuration file that gives the program info on what to run. To view this file in the terminal, use the program nano:

`nano Contigs-Config.json`

You should now see the full file in your terminal window. You can move the cursor with your arrow keys and edit text by deleting it with the "delete" or "backspace" key on your keyboard and replacing it with the new text. To use this file, we need to specify what genomes Anvi'o should be using for creating the Contigs Databases. This information is stored in the "Sequence_fasta.txt" file that we created in Step 2.

The top line of the Contigs-Config.json file should read "fasta.txt": "fasta.txt". You must replace the second instance of "fasta.txt" with "Sequence_fasta.txt". All other parameters may stay the same if you wish to create a Contigs Database and add new annotations from predefined databases (e.g. COGs, tRNA scan, etc.). To turn off a function, change the "run": true line to "run": false.

A final important setting can be found at the very bottom of the file. "max_ threads" specifies how many processing cores the programs can run on simultaneously. You may increase this number according to the processing power of your computer, but you must be very careful to not exceed your processing capabilities. The number of threads can also be changed individually for programs that allow multithreading, but the "max_threads" option at the very end is a fail safe that will override any thread parameter that is higher than it. By default, this option has no value and will therefore adhere to the thread values of each individual program.

Once you have your settings adjusted, you may run the following command to begin the workflow:

~~~
anvi-run-workflow -w contigs \
	-c Contigs-Config.json
~~~

Once the Contigs Databases are created, we can export genes or loci of interest in 2 different ways, described in Steps 4.A and 4.B

### Step 4.A: Export Gene or Locus of Interest

To export a gene or locus of interest, you can use the anvi-export-locus program:

~~~
anvi-export-locus -c Yersinia-CONTIGS.db \
	--num-genes 0,0 \
	--search-term "SecA" \
	-O Yersinia_SecA
	
anvi-export-locus -c Staphylococcus-CONTIGS.db \
	--num-genes 0,0 \
	--search-term "SecA" \
	-O Staphylococcus_SecA

anvi-export-locus -c Salmonella-CONTIGS.db \
	--num-genes 0,0 \
	--search-term "SecA" \
	-O Salmonella_SecA

anvi-export-locus -c Chlamydia-CONTIGS.db \
	--num-genes 0,0 \
	--search-term "SecA" \
	-O Chlamydia_SecA

anvi-export-locus -c Bacillus-CONTIGS.db \
	--num-genes 0,0 \
	--search-term "SecA" \
	-O Bacillus_SecA
~~~
> -c: Contigs database
> 
> --num-genes: number of genes to include upstream and downstream of the search term gene
> 
> --search-term: your gene of interest. Must match with the functional annotations that you added to the Contigs Database.
> 
> -O: output file name

### Step 4.B: Export Gene or Locus of Interest Using a Bash Loop

To export genes from multiple Contigs Databases, you can use a Bash loop similar to the one that we used before. Here, we can use a more complex set of commands to automatically grab file names instead of listing them manually:

~~~
for GENOME in *-CONTIGS.db
do
	FNAME=$(basename "${GENOME}" -CONTIGS.db)
	anvi-export-locus -c ${GENOME} \
		--num-genes 0,0 \
		--search-term "SecA" \
		-O ${FNAME}_SecA
done
~~~

To run this loop, copy it into a new file in nano:

`nano Export_SecA_Loop.sh`

To save the file, press "Ctrl + X" and press "y" to overwrite the file. The nano controls can be found at the bottom of the screen. You now have a bash script saved in your working directory. To run the loop, run the following command:

`bash Export_SecA_Loop.sh`

### Step 5: Align Genes Using MUSCLE

MUSCLE is a commonly used alignment software installed by default with Anvi'o. This software can be used directly from the command line, but it requires a fasta file containing all sequences to be aligned. Such a file can be easily created with a single Bash command:

'awk 1 *SecA*.fa > SecA_Sequences.fa'

Once you have a multifasta file, you can run MUSCLE:

'muscle -in SecA_Sequences.fa -out SecA_Sequence_Alignment.afa'

This alignment can then be opened in any number of programs for viewing.
