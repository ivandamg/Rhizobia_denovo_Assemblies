# Rhizobia_denovo_Assemblies
Pipeline for denovo assemblies from illumina sequencing



A. Clean reads with fastp


      for FILE in $(ls *_R1.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=$(echo $FILE | cut -d'_' -f1,2)_bbduk --time=01:00:00 --mem-per-cpu=64G --ntasks=2 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1,2)_bbduk.out --error=$(echo $FILE | cut -d'_' -f1,2)_bbduk.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/BBMap/38.91;cd /data/projects/p495_SinorhizobiumMeliloti/03_MasterSummerProject/01_rawSequencing2022 ;  bbduk.sh in=$FILE in2=$(echo $FILE | cut -d'_' -f1,2)_R2.fastq.gz out=$(echo $FILE | cut -d'_' -f1,2)_clean.1.fq.gz out2=$(echo $FILE | cut -d'_' -f1,2)_clean.2.fq.gz ref=adapters ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=r trimq=24"   ; sleep 1; done


B. denovo assemblies with SPAdes

    for FILE in $(ls WT1_L2_clean.1.fq.gz); do echo $FILE; sbatch --partition=pall --job-name=$(echo $FILE | cut -d'_' -f1,2)_spaDES --time=48:00:00 --mem-per-cpu=64G --ntasks=16 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1,2)_bbduk.out --error=$(echo $FILE | cut -d'_' -f1,2)_bbduk.error --mail-type=END,FAIL --wrap "/home/imateusgonzalez/00_Software/SPAdes-3.15.5-Linux/bin/spades.py -k 21,33,55,77 --careful --pe1-1 $FILE --pe1-2 $(echo $FILE | cut -d'_' -f1,2)_clean.2.fq.gz -o $(echo $FILE | cut -d'_' -f1,2)_spades"   ; sleep 1; done
