# snpSniffer

Tool for checking genotype concordance between multiple assays

This tools was developed by Venkata "Teja" Yellapantulla as part of his PhD thesis work in the Keats Lab at Translational Genomics Research Institute.

This tool is maintained by Salvatore Facista (TGen - Lab of Dr. William Hendricks) in 2020. The tool will now function with custom .ini files, generated with any number of SNPs. Please submit bug reports via Github.


## Usage
- Move the configuration (.txt "reference" file, .ini "flat" file, and "geno" file) into the same directory as snpSniffer.jar. Some aspects of snpSniffer are underdeveloped and it will not appropriately handle paths. Please submit a bug report on Github if this causes you issues.

- To generate a custom .ini file
Use a text editor to format a blank file as below (or see included .ini files for examples). Ensure files have Unix style line endings (run dos2unix to convert DOS-style line endings to Unix style):

Sample
<br>&lt;chromosome&gt;:&lt;position index&gt;</br>

- To generate a custom "reference" file users will need the same information as the .ini file. The format for the file is slightly different. Ensure files have Unix style line endings (run dos2unix to convert DOS-style line endings to Unix style):
<br>&lt;chromosome&gt;	&lt;position index&gt;</br>
Note: The tab character separates the chromosome and position index. See example .txt files in the package for more information.

- To edit "geno" file so that it functions with the custom reference file you created
Edit the file name in "geno" with your favourite text editor. Ensure it resides within the same directory as the .jar file.

- To generate genotypes from a bam:

java -jar snpSniffer.jar -genotype <fullFilePath/reference> <fullFilePath/BAM>
        Alternately, genotypes can be generated using:
	
	Note: To clarfiy, snpSniffer.jar -genotype simply initiates a BASH script that is located in the "geno" file.
	Users may find it easier to run these commands without the script utilizing the SamTools utility.

		java -jar snpSniffer.jar -genotype <fullFilePath/reference> <fullFilePath/BAM>
	
  Alternately, genotypes can be generated using:

		geno <fullFilePath/reference> <fullFilePath/BAM>

 *Users should make sure bam is indexed

- To add genotypes from a vcf:

`java -jar snpSniffer.jar -add <fullFilePath/VCF fileName> <fullFilePath/database.ini>`

- To view all samples:

`java -jar snpSniffer.jar -check Samples <fullFilePath/database.ini>`

- To check concordance of genotypes for a sample:

`java -jar snpSniffer.jar -check <sampleName> <fullFilePath/database.ini>`

- For help:

`java -jar snpSniffer.jar -help`



## Example usage
1) Generate the custom .ini flat file, or use one of the provided files. snpSniffer does not do this for you. The flat file is just a modified text file. Ensure you have Unix-style line endings by using dos2unix if you are generating the .ini file on a non-Unix system.


2) Generate the genotypes in a vcf format at specific genomic loci:
    `java -jar ~/local/bin/snpSniffer.jar -genotype /lustre/vyellapa/reference.fa /lustre/vyellapa/sample1.bam`

3) Adding the genotypes generated to a flat file "database.ini," provided
    Step 1, will generate a vcf having the same name as the bam in the same directory, this will be added to database.ini with same name:
    `java -jar ~/local/bin/snpSniffer.jar -add /lustre/vyellapa/sample1.vcf /lustre/vyellapa/database.ini`

4) Compare the genotypes for samples of interest(after 2 or more vcf's are added), examine the snpSniffer output and infer if any mixups occurred:
    `java -jar ~/local/bin/snpSniffer.jar -check sample1 /lustre/vyellapa/database.ini`

Note: Users need to reference the correct directory when attempting to run snpSniffer.jar . Your version of snpSniffer.jar may reside in a different directory than the example above. 

## Example output

Step (4) above should generate lines of output, depending on number of samples, similar to the one below. It shows that between sample1 and sample2, 171 genotypes were obtained with good quality out of which 169 positions match. The "ratio" field is the ratio of match to count and ratio>0.8 signifies that the two samples match.

In the output given below, sample1 and sample2 have a ratio of ~0.98 suggesting both sequences come from the same individual. However, sample1 and sample3 have have a ratio of ~0.32 suggesting that the sequences do not come from the same individual.

```
sample1 & sample2 count=171.0 match=169.0 ratio=0.9883040935672515

sample1 & sample3 count=325.0 match=107.0 ratio=0.3292307692307692
```

### SnpSniffer Wrapper Usage Notes

This package contains a wrapper script that leverages the power of the SnpSniffer JAVA tool to identify sample mix-ups and cross-contaminations

It can be launched from any directory as the first step is a find for any VCF files in the directory structure below the starting directory

Dependancies:
SnpSniffer.jar (Included)
Run_SnpSniffer.sh (Included)
SnpSniffer_Graph.R (Included)
R
GGPLOT2 Library
Samtools (includes BCFtools - do not use outdated stand-alone BCFtools).

### USAGE:

1) Update the absolute paths for the VARIABLES in Run_SnpSniffer.sh
```
	EMPTY_DATABASE=/data/tools/snpSniffer.v5/database.ini
	SNP_SNIFFER_JAR=/data/tools/snpSniffer.v5/snpSniffer.jar
	SnpSniffer_Graph_R=/data/jkeats/scripts/SnpSniffer_Graph.R
```

2) Execute the wrapper <./Run_SnpSniffer.sh>

	This will produce a number of tables and graphs.
	The table to focus on the most is "SnpSniffer_NotExpected_Results_SharedCalls20plus.txt"
		Comparisons of the same patient with ratios less than 0.8 are included
			These are likely mixups, as even tumor-normal comparisons typically have higher values
		Comparisons of different patients with ratios above 0.8 are included
			This is unexpected and one of the two samples is likely a mixup
	Graphs are created in triplicate to filter for number of possible matches
		100 and 50 should be considered high confidence
		20 are often still valuable but can have false positives
		
### UPDATE NOTES:

20200214 - Changed main class name to exclude version number. Added more verbose main method argument error message. Added -version option. Changed the Genotype class to recognize the format "0/0" as a homozygous reference. Previously, "0" was expected for homozygous reference. Renamed internal classes to exclude version number.

20190930 - Fixed improper parameter reference in expectedV6.java. Cleaned up the expected/ notExpectedV6.java files so that they look more similar. Added more verbose error messages in countLines.java. Repackaged .jar. Overwrote old .java & .class files with the updated ones (as a whole). End notes 20190630

20190611 - Substituted ad-hoc array dimension generation for previously, hard-coded values. You can view the original source for the .jar in the src directory on GitHub. Please note that the source was decompiled using Procyon decompiler, as original .java files were not immediately available. New source files have the internal V6-modified file name suffix; however, due to not wanting to tool with re-writing the wrapper, I have left the .jar as V5. Much thanks to the Procyon team. Special thanks to Dr. Nieves Perdigones for her assistance in helping to build out a tool .ini for the canine model. Thank you also to Natalia Briones for her assistance in configuration and troubleshooting. End update notes 20190611

20190625 - Fixed "off by one" indexing error in checkV5.java/.class when running without specific sample names. I have uploaded the new files to the src directory. Removed redundant copies of the .jar file and put it in the root directory for the project. I added a lot to documentation. 
KNOWN ISSUE: snpSniffer geno command may not work if the geno files or subsequent reference files are not in the same directory as the .jar file. This is an artifact of poor scripting. I have no intention to remedy the issue at this time. Users should move the .jar to the same directory as the dependencies, manually process samples through the "geno" script, or move the dependent files to the same directory as the .jar. End update notes 20190625
