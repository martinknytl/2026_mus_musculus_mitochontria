*** 1) Trimmomatic

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
