### 1) Trimmomatic

```
#!/bin/sh
#SBATCH --job-name=trimmomatic
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=72:00:00
#SBATCH --mem=8gb
#SBATCH --output=trimmomatic.%J.out
#SBATCH --error=trimmomatic.%J.err
#SBATCH --account=rrg-ben

# run by passing a directory argument like this
# sbatch ./2020_trimmomatic.sh path_to_raw_data
# NexteraPE-PE.fa must be present in the directory, from which the trimmomatic script is executed

module load StdEnv/2023
module load trimmomatic/0.39
# R1_001.fastq.gz
#v=1
#  Always use for-loop, prefix glob, check if exists file.
for file in $1/*R1_001.fastq.gz; do         # Use ./* ... NEVER bare *
  if [ -e "$file" ] ; then   # Check whether file exists.
      java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.39.jar PE ${file::-15}R1_001.fastq.gz ${file::-15}R2_001.fastq.gz ${file::-15}trim_R1.fq.gz ${file::-15}trim_single_R1.fq.gz ${file::-15}trim_R2.fq.gz ${file::-15}trim_single_R2.fq.gz ILLUMINACLIP:NexteraPE-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
  fi
done
```

```
sbatch /home/knedlo/projects/rrg-ben/knedlo/martin_scripts/2026_trimmomatic_mitochondria.sh ./
```

### 2) bwa alignment

mapped to the Mus musculus nuclear genome downloaded from:
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/635/GCF_000001635.27_GRCm39/GCF_000001635.27_GRCm39_genomic.fna.gz
```

unzip the genome:
```
gunzip GCF_000001635.27_GRCm39_genomic.fna.gz
```

make genome blastable:
```
makeblastdb -in GCF_000001635.27_GRCm39_genomic.fna -dbtype nucl -out GCF_000001635.27_GRCm39_genomic.fna_blastable
```

make an index file:
```
bwa index GCF_000001635.27_GRCm39_genomic.fna
```

```
#!/bin/sh
#SBATCH --job-name=bwa_align
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=4
#SBATCH --time=72:00:00
#SBATCH --mem=32gb
#SBATCH --output=bwa_align.%J.out
#SBATCH --error=bwa_align.%J.err
#SBATCH --account=rrg-ben

# run by passing an argument like this (in the directory with the files)
# sbatch 2020_align_paired_fq_to_ref.sh pathandname_of_ref path_to_paired_fq_filez
# sbatch 2020_align_paired_fq_to_ref.sh /home/ben/projects/rrg-ben/ben/2018_Austin_XB_genome/Austin_genome/Xbo.v1.fa.gz pathtofqfilez

module load bwa/0.7.17
# module load samtools/1.10
module load StdEnv/2023  gcc/12.3 samtools/1.20

for file in ${2}/*_trim_R2.fq.gz ; do         # Use ./* ... NEVER bare *    
    if [ -e "$file" ] ; then   # Check whether file exists.
	bwa mem ${1} ${file::-14}_trim_R1.fq.gz ${file::-14}_trim_R2.fq.gz -t 16 | samtools view -Shu - | samtools sort - -o ${file::-14}_sorted.bam
	samtools index ${file::-14}_sorted.bam
  fi
done
```

two scripts executed: Oocyty1 pool and Oocyty2 pool
```
sbatch /home/knedlo/projects/rrg-ben/knedlo/ben_scripts/2020_align_paired_fq_to_ref.sh /home/knedlo/projects/rrg-ben/knedlo/mitochondria/mus_musculus_nuclear_genome/GCF_000001635.27_GRCm39_genomic.fna .
```
