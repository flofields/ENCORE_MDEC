---
LAYOUT: Post
TITLE: QC of RNAseq Mdec files from the ENCORE 2022 TPC project 
DATE: '2024-03-08'
CATEGORIES: QC
TAGS: [Coral, Quailty Control, Madracis decactis, RNAseq ]
PROJECTS: ENCORE
---

## Quality Control of RNAseq files for Madracis decactis (Mdec) ENCORE 2022 TPC project  

**About**: This post details the QC of Mdec from the ENCORE 2022 Thermal performance curve (TPC) project RNAseq files. See [here](https://github.com/flofields/MDEC_Reference_Transcriptome/blob/aba1fd09f33816aff2abfebccfd25423cd0ec537/metadata/Project-Summary-ENCORE-MDEC-RNA-DNA-Extractions.md) for the project summary for the Mdec DNA and RNA extractions and [my notebook post](https://flofields.github.io/Florence_Putnam_Lab_Notebook/Madracis-decactis-denovo-transcriptome/) for the denovo transcriptome which this QC was used for.

### 1) Write and run script with raw data for checking quality with FastQC on Andromeda (untrimmed and unfiltered)

```
nano /data/putnamlab/flofields/denovo_transcriptome/scripts/fastqc.sh
```
```
#!/bin/bash
#SBATCH -t 24:00:00
#SBATCH --nodes=1 --ntasks-per-node=1
#SBATCH --export=NONE
#SBATCH --mem=100GB
#SBATCH --mail-type=BEGIN,END,FAIL #email you when job starts, stops and/or fails
#SBATCH --mail-user=ffields@uri.edu #your email to send notifications
#SBATCH --account=putnamlab
#SBATCH -D /data/putnamlab/flofields/denovo_transcriptome/data/raw
#SBATCH --error="script_error" #if your job fails, the error report will be put in this file
#SBATCH --output="output_script" #once your job is completed, any final job report comments will be put in this file

module load FastQC/0.11.9-Java-11

for file in /data/putnamlab/flofields/denovo_transcriptome/data/raw/MDEC*
do
fastqc $file --outdir /data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results/
done
```

#### Run the script.

```
sbatch /data/putnamlab/flofields/denovo_transcriptome/scripts/fastqc.sh
```

#### Submitted batch job 289531 on Nov 28 2023
#### Finished Nov 28 2023
---

#### Combined QC output into 1 file with MultiQC, a script is not needed due to fast computational time
```
#load module 
module load MultiQC/1.9-intel-2020a-Python-3.8.2

#Combined files 
multiqc /data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results/*fastqc.zip -o /data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results/multiqc/
```
---

#### Copied MultiQC and FastQC report to my computer : Run this in the computer's terminal not the server
```
scp -r ffields@ssh3.hac.uri.edu:/data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results/multiqc/multiqc_report.html /Users/flo_f/Putnam-lab/bioinformatics/MDEC_transcriptome/original_fastqc
scp -r ffields@ssh3.hac.uri.edu:/data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results/*.html /Users/flo_f/Putnam-lab/bioinformatics/MDEC_transcriptome/original_fastqc
```
---

### The raw sequence [MultiQC Report can be found here on GitHub](https://github.com/flofields/MDEC_Reference_Transcriptome/tree/c02b9093942749497552c2e5e5018301b1b9f231/FastQC_Reports/Original)
---

### Understanding a [MultiQC Report](https://nf-co.re/eager/2.5.0/docs/output#multiqc-report) and [Fastp](https://github.com/OpenGene/fastp#adapters)
### [Secondary Fastp source](https://open.bioqueue.org/home/knowledge/showKnowledge/sig/fastp)
---
These Mdec samples were pooled and had RNA concentrations of [Qbit 67.20ng/ul and Nanodrop 95.60ng/ul](https://github.com/flofields/MDEC_Reference_Transcriptome/blob/main/metadata/Sample%20QC%20report%20of_30-818136646_230814121610.pdf)

 | Sample Name | % Dups| % GC | M Seqs |
 |-------------|-------|------|--------|
 | MDEC_R1_001 | 69.6% | 43%  | 223.6  |
 | MDEC_R2_001 | 65.1% | 43%  | 223.6  |
  
- Adapter content present in sequences. Adapters have not been removed yet via trimming.

![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/543dd3185cc809ab2b43abd8dc93e0a2ad9f564f/images/Mdec_RNAseq/raw_multiqc/fastqc_adapter_content_plot.png)

- Warnings were attached to the GC content. This could be a result of poly-G tails from Illumina NextSeq.

![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/543dd3185cc809ab2b43abd8dc93e0a2ad9f564f/images/Mdec_RNAseq/raw_multiqc/fastqc_per_sequence_gc_content_plot.png)

- Sequence counts shows that their is a high number of over-represented sequences. This can occur when they are highly expressed genes.

![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/543dd3185cc809ab2b43abd8dc93e0a2ad9f564f/images/Mdec_RNAseq/raw_multiqc/fastqc_sequence_counts_plot.png)
![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/543dd3185cc809ab2b43abd8dc93e0a2ad9f564f/images/Mdec_RNAseq/raw_multiqc/fastqc_sequence_duplication_levels_plot.png)

- Quality scores are good

![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/52a6e9c629da8f19efa400ac211a4a6a6764c362/images/Mdec_RNAseq/raw_multiqc/fastqc_per_sequence_quality_scores_plot.png)

- Low base N content

![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/52a6e9c629da8f19efa400ac211a4a6a6764c362/images/Mdec_RNAseq/raw_multiqc/fastqc_per_base_n_content_plot.png)

### 2) Trimming 
Trimming steps below were taken then another QC report was generated to decide if other trimming decisions needed to be made.

#### A new folder and script was created for the trimmed data files
```
mkdir /data/putnamlab/flofields/denovo_transcriptome/data/trimmed
```
```
nano /data/putnamlab/flofields/denovo_transcriptome/scripts/trim.sh
```
The trimming settings in [fastp](https://github.com/OpenGene/fastp#adapters) 
 
- --detect_adapter_for_pe 
Enables adapter sequence auto-dection. This trim is a result of the presence of Adapter content in the multiQC report

- --trim_poly_g 
Enables trimming of the polyG tails that occurs from signal degradation. This trim is a result of the sequence GC-content warning

- --trim_tail1 15 
Trimmed 15 base pairs from the end of the foward sequence 3'-5'.

- --trim_tail2 15
Trimmed 15 base pairs from the end of the reverse sequence 5'-3'.
Trims 15bp from the 3'R1 and 5'R2 end of reads

```
#!/bin/bash
#SBATCH -t 24:00:00
#SBATCH --nodes=1 --ntasks-per-node=1
#SBATCH --export=NONE
#SBATCH --mem=100GB
#SBATCH --mail-type=BEGIN,END,FAIL #email you when job starts, stops and/or fails
#SBATCH --mail-user=ffields@uri.edu #your email to send notifications
#SBATCH --account=putnamlab
#SBATCH -D /data/putnamlab/flofields/denovo_transcriptome/data/raw
#SBATCH --error="script_error" #if your job fails, the error report will be put in this file
#SBATCH --output="output_script" #once your job is completed, any final job report comments will be put in this file

cd /data/putnamlab/flofields/denovo_transcriptome/data/raw
module load fastp/0.19.7-foss-2018b

fastp --in1 MDEC_R1_001.fastq --int2 MDEC_R2_001.fastq --detect_adapter_for_pe --trim_poly_g --trim_tail1 15 --trim_tail2 15 --out1 /data/putnamlab/flofields/denovo_transcriptome/data/trimmed/MDEC_001_trim_R1.fastq --out2 /data/putnamlab/flofields/denovo_transcriptome/data/trimmed/MDEC_001_trim_R2.fastq
```
```
sbatch /data/putnamlab/flofields/denovo_transcriptome/scripts/trim.sh
```
#### Submitted batch job 290641 on Dec 11 2023
#### Finished Dec 11 2023
---
Check the quality of the trimmed files by confirming the number of files that were trimed and to look at the raw reads
- orginal files
```
#get sequence name
less MDEC_001_trim2_R2.fastqq
```
```
zgrep -c "@A01587"MDEC* > seq_counts
```
- Trimmed files
```
zgrep -c "@A01587"MDEC* > trimmed_seq_counts
```
### 3i) Fastqc and MultiQC on trim2 sequences
#### Run fastqc on trimmed data
```
mkdir fastqc_results_trimmed
```
```
nano /data/putnamlab/flofields/denovo_transcriptome/scripts/fastqc_trimmed.sh
```

```
#!/bin/bash
#SBATCH -t 24:00:00
#SBATCH --nodes=1 --ntasks-per-node=1
#SBATCH --export=NONE
#SBATCH --mem=100GB
#SBATCH --mail-type=BEGIN,END,FAIL #email you when job starts, stops and/or fails
#SBATCH --mail-user=ffields@uri.edu #your email to send notifications
#SBATCH --account=putnamlab
#SBATCH -D /data/putnamlab/flofields/denovo_transcriptome/data/trimmed
#SBATCH --error="script_error" #if your job fails, the error report will be put in this file
#SBATCH --output="output_script" #once your job is completed, any final job report comments will be put in this file

module load FastQC/0.11.9-Java-11

for file in /data/putnamlab/flofields/denovo_transcriptome/data/trimmed/MDEC*
do
fastqc $file --outdir /data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results_trimmed/
done
```

```
sbatch /data/putnamlab/flofields/denovo_transcriptome/scripts/fastqc_trimmed.sh
```

#### Submitted Batch Job 293773 Jan 29 2024
#### Finished Jan 29 2024

#### Combined QC output into 1 file with MultiQC and copied to my destop to look at the trimming information

```
#load module 
module load MultiQC/1.9-intel-2020a-Python-3.8.2

multiqc /data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results_trimmed/*fastqc.zip -o /data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results_trimmed/trimmed_multiqc
```

```
scp -r ffields@ssh3.hac.uri.edu://data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results_trimmed/trimmed_multiqc/multiqc_report.html /Users/flo_f/OneDrive/Desktop/Putnam-lab/bioinformatics/MDEC_transcriptome/trimmed_fastqc
scp -r ffields@ssh3.hac.uri.edu://data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results_trimmed/*.html /Users/flo_f/OneDrive/Desktop/Putnam-lab/bioinformatics/MDEC_transcriptome/trimmed_fastqc
```
---
#### The raw sequence [MultiQC report can be found here in Github](https://github.com/flofields/MDEC_Reference_Transcriptome/blob/main/FastQC_Reports/trim1/multiqc_report.html)

The MultiQC report showed that the number of unque reads had been trimmed from 68,038,679 to 66,625,214, sequence quality, per sequence quality scores, overrepresented sequences, per base n content was still good and the adapter content was now good however sequence length distribution changed from normal to slightly abnormal as well as the foward stand's per tile sequence quality.

See the [status check heat map](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/master/images/Mdec_RNAseq/trim1_multiqc/fastqc-status-check-heatmap.png) for the general overview of the multiQC report of the trim 1


| Sample Name | % Dups| % GC | M Seqs |
 |-------------|-------|------|--------|
 | MDEC_R1_001 | 69.6% | 43%  | 219.0  |
 | MDEC_R2_001 | 65.1% | 43%  | 219.0  |


The adapter was removed which meant it was not necessary to remove base pairs from the end of the foward and reverse stands so I trimmed the raw data again removing only the adapter and poly g tail

---

#### A new folder and script was created for the trim 2 data files

```
mkdir /data/putnamlab/flofields/denovo_transcriptome/data/trim2
```

```
nano /data/putnamlab/flofields/denovo_transcriptome/scripts/trim2.sh
```

```
#!/bin/bash
#SBATCH -t 24:00:00
#SBATCH --nodes=1 --ntasks-per-node=1
#SBATCH --export=NONE
#SBATCH --mem=100GB
#SBATCH --mail-type=BEGIN,END,FAIL #email you when job starts, stops and/or fails
#SBATCH --mail-user=ffields@uri.edu #your email to send notifications
#SBATCH --account=putnamlab
#SBATCH -D /data/putnamlab/flofields/denovo_transcriptome/data/raw
#SBATCH --error="script_error" #if your job fails, the error report will be put in this file
#SBATCH --output="output_script" #once your job is completed, any final job report comments will be put in this file

cd /data/putnamlab/flofields/denovo_transcriptome/data/raw
module load fastp/0.19.7-foss-2018b

fastp --in1 MDEC_R1_001.fastq --int2 MDEC_R2_001.fastq --detect_adapter_for_pe -D --trim_poly_g --out1 /data/putnamlab/flofields/denovo_transcriptome/data/trim2/MDEC_001_trim2_R1.fastq --out2 /data/putnamlab/flofields/denovo_transcriptome/data/trim2/MDEC_001_trim2_R2.fastq
```
```
sbatch /data/putnamlab/flofields/denovo_transcriptome/scripts/trim2.sh
```
#### Submitted batch job 293922 on Jan 31 2024
#### Finished on Jan 31 2024

I then downloaded the fastp.html report to look at the trimmin information
```
scp -r ffields@ssh3.hac.uri.edu://data/putnamlab/flofields/denovo_transcriptome/data/raw/fastp.html /Users/flo_f/OneDrive/Desktop/Putnam-Lab/mdec-rnaseq
```
This file can be found [here on Github](https://github.com/flofields/MDEC_Reference_Transcriptome/blob/main/data/rna_seq/QC/fastp.html)

Here are the results 

### General statistics 

| fastp version:                | 0.19.7 (https://github.com/OpenGene/fastp) |
|-------------------------------|--------------------------------------------|
| sequencing:                   | paired end (150 cycles + 150 cycles)       |
| mean length before filtering: | 150bp, 150bp                               |
| mean length after filtering:  | 147bp, 147bp                               |
| duplication rate:             | 23.583228%                                 |
| Insert size peak:             | 176                                        |
| Detected read1 adapter:       | AGATCGGAAGAGCACACGTCTGAACTCCAGTCA          |
| Detected read2 adapter:       | AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT          |

### Before filtering 

| total reads: | 447.181454 M            |
|--------------|-------------------------|
| total bases: | 67.077218 G             |
| Q20 bases:   | 64.582642 G (96.281038%)|
| Q30 bases:   | 60.960502 G (90.881082%)|
| GC content:  | 43.819885%              |

### After filtering 

| total reads: | 437.584742 M            |
|--------------|-------------------------|
| total bases: | 64.576101 G             |
| Q20 bases:   | 62.624206 G (96.977374%)|
| Q30 bases:   | 59.253674 G (91.757900%)|
| GC content:  | 43.430054%              |

### Filtering results 

| reads passed filters:   | 437.584742 M (97.853956%)|
|-------------------------|--------------------------|
| reads with low quality: | 9.263188 M (2.071461%)   |
| reads with too many N:  | 39.784000 K (0.008897%)  |
| reads too short:        | 293.740000 K (0.065687%) |

These results show that filtering improved quality of reads and removed about 2% of reads due to length and qaulity. Average QC30 bases improved from 90% to 91%. I will be running another round of fastqc and multiqc to see how this changed our qc results.

---

### 3ii) Fastqc and MultiQC on trim2 sequences
#### Run fastqc on trim 2 data

```
mkdir fastqc_results_trim2
nano /data/putnamlab/flofields/denovo_transcriptome/scripts/fastqc_trim2.sh
```

```
#!/bin/bash
#SBATCH -t 24:00:00
#SBATCH --nodes=1 --ntasks-per-node=1
#SBATCH --export=NONE
#SBATCH --mem=100GB
#SBATCH --mail-type=BEGIN,END,FAIL #email you when job starts, stops and/or fails
#SBATCH --mail-user=ffields@uri.edu #your email to send notifications
#SBATCH --account=putnamlab
#SBATCH -D /data/putnamlab/flofields/denovo_transcriptome/data/trimmed
#SBATCH --error="script_error" #if your job fails, the error report will be put in this file
#SBATCH --output="output_script" #once your job is completed, any final job report comments will be put in this file

module load FastQC/0.11.9-Java-11

for file in /data/putnamlab/flofields/denovo_transcriptome/data/trimmed/MDEC*
do
fastqc $file --outdir /data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results_trim2/
done
```

```
sbatch /data/putnamlab/flofields/denovo_transcriptome/scripts/fastqc_trim2.sh
```

#### Submitted batch job 294032 on Jan 31 2024
#### Finished on Jan 31 2024

#### Combined QC output into 1 file with MultiQC and fastp and copied to my destop to look at the trim 2 information

```
#load module 
module load MultiQC/1.9-intel-2020a-Python-3.8.2

#Copy the multiqc html file to my computer
multiqc /data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results_trim2/*fastqc.zip -o /data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results_trim2/trim2_multiqc
```

```
scp -r ffields@ssh3.hac.uri.edu://data/putnamlab/flofields/denovo_transcriptome/data/fastqc_results_trimmed/trimmed_multiqc/multiqc_report.html /Users/flo_f/OneDrive/Desktop/Putnam-Lab/bioinformatics/MDEC_transcriptome/trimmed_fastqc
```

#### The raw sequence [MultiQC report can be found here in Github](https://github.com/flofields/MDEC_Reference_Transcriptome/blob/main/data/rna_seq/QC/trim2/multiqc_report.html)

The MultiQC report results

| Sample Name |%Duplication| GC content| %PF|%Adapter| % Dups| % GC | M Seqs |
|-------------|------------|-----------|----|--------|-------|------|--------|
| Fastp       |  23.58     |  43.4     |97.9|
| MDEC_R1_001 |            |           |    |        | 69.6% | 43%  | 218.0  |
| MDEC_R2_001 |            |           |    |        | 66.1% | 43%  | 218.0  |

- Fastp filtering: most reads filtered were due to low quality

- Sequence counts shows that 30.4% of reads in R1 is unique and 33.9% in R2 is unique however dulication levels/over represented sequences are high This can occur when they are highly expressed genes. It is possible to have good libraries with small peaks at high duplication levels.
![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/dc0cd33982f0f8de074d0662a69ac8942640439d/images/Mdec_RNAseq/trim2_multiqc/fastqc_sequence_counts_plot.png)
![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/dc0cd33982f0f8de074d0662a69ac8942640439d/images/Mdec_RNAseq/trim2_multiqc/fastqc_sequence_duplication_levels_plot.png)

- Sequence Quality is good
![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/dc0cd33982f0f8de074d0662a69ac8942640439d/images/Mdec_RNAseq/trim2_multiqc/fastqc_per_sequence_quality_scores_plot.png)
![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/dc0cd33982f0f8de074d0662a69ac8942640439d/images/Mdec_RNAseq/trim2_multiqc/fastqc_per_base_sequence_quality_plot.png)

- Per Sequence GC Content came with warmings this could mean that tey are alot of PCR duplicates
![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/3f977efdc7e2c7897b84dfa67f79a1b3f566d489/images/Mdec_RNAseq/trim2_multiqc/fastqc_per_sequence_gc_content_plot.png)

- Low base n content
![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/3f977efdc7e2c7897b84dfa67f79a1b3f566d489/images/Mdec_RNAseq/trim2_multiqc/fastqc_per_base_n_content_plot.png)

- The status check below shows the overall status for each FastQC section where gree is normal, orange is slightly abnormal and red being very unsual.
![](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/dc0cd33982f0f8de074d0662a69ac8942640439d/images/Mdec_RNAseq/trim2_multiqc/fastqc-status-check-heatmap.png)

Next I will be using the trim2 data to run in trinity. 
This QC was for the purpose of assembling a denov transciptome. The entire process can be found on Github [Here.](https://github.com/flofields/Florence_Putnam_Lab_Notebook/blob/master/_posts/2023-01-31-Madracis-decactis-denovo-transcriptome.md)