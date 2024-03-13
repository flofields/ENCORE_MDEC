```
cd /data/putnamlab/KITT/hputnam/   
mkdir 20231101_ENCORE_Mdec_TPC
mkdir scripts   
cd 20231101_ENCORE_Mdec_TPC   
```

```
#!/bin/bash
#SBATCH -t 24:00:00
#SBATCH --nodes=1 --ntasks-per-node=1
#SBATCH --export=NONE
#SBATCH --mem=100GB
#SBATCH --account=putnamlab
#SBATCH -D /data/putnamlab/KITT/hputnam/20231101_ENCORE_Mdec_TPC

module load IlluminaUtils/2.11-GCCcore-9.3.0-Python-3.8.2

bs download project -i 401745805 -o /data/putnamlab/KITT/hputnam/20231101_ENCORE_Mdec_TPC
```

# Generate checksum

mv SA*/*.gz .
rm -r SA23142*
md5sum *.gz > URI.downloaded.md5


# QC raw files


```
nano /data/putnamlab/KITT/hputnam/20231101_ENCORE_Mdec_TPC/scripts/qc.sh
```

```
#!/bin/bash
#SBATCH -t 24:00:00
#SBATCH --nodes=1 --ntasks-per-node=1
#SBATCH --export=NONE
#SBATCH --mem=100GB
#SBATCH -D /data/putnamlab/KITT/hputnam/20231101_ENCORE_Mdec_TPC

module load FastQC/0.11.9-Java-11 
module load MultiQC/1.9-intel-2020a-Python-3.8.2

#make qc output folder
mkdir raw_qc/

#run fastqc on raw data
fastqc *.fastq.gz -o raw_qc/

#Compile MultiQC report from FastQC files
multiqc ./raw_qc
mv multiqc_report.html raw_qc/raw_qc_multiqc_report.html
mv multiqc_data raw_qc/raw_multiqc_data

echo "Initial QC of Seq data complete." $(date)
```

```
sbatch /data/putnamlab/KITT/hputnam/20231101_ENCORE_Mdec_TPC/scripts/qc.sh
```