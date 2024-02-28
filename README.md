# Rhizobia_denovo_Assemblies
Pipeline for denovo assemblies from illumina sequencing

0. Change reads names
            mv H6_L1_R2_001_852WMH6HscGS.fastq.gz  H6_L1_R2.fastq.gz

A. Clean reads with fastp


             for FILE in $(ls *_R1.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1,2)fastp --time=0-01:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1,2)_fastp.out --error=$(echo $FILE | cut -d'_' -f1,2)_fastp.error --mail-type=END,FAIL --wrap " cd /data/projects/p495_SinorhizobiumMeliloti/07_Axelle/03_denovoNifh_v2; module load FastQC; ~/00_Software/fastp --in1 $FILE --in2 $(echo $FILE | cut -d'_' -f1,2)_R2.fastq.gz --out1 02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_1_trimmed.fastq.gz --out2 02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_2_trimmed.fastq.gz -h 02_TrimmedData/$(echo $FILE | cut -d',' -f1,2)_fastp.html --thread 4; fastqc -t 4 02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_1_trimmed.fastq.gz; fastqc -t 4 02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_2_trimmed.fastq.gz"; sleep  1; done


B. denovo assemblies with SPAdes

    sbatch --partition=pibu_el8 --job-name=H1_spaDES --time=48:00:00 --mem-per-cpu=64G --ntasks=8 --cpus-per-task=1 --output=H1_spades.out --error=H1_spades.error --mail-type=END,FAIL --wrap "cd /data/projects/p495_SinorhizobiumMeliloti/07_Axelle/03_denovoNifh_v2/02_TrimmedData; /home/imateusgonzalez/00_Software/SPAdes-3.15.5-Linux/bin/spades.py -k 21,33,55,77 --careful --pe1-1 -1 H1_L1_1_trimmed.fastq.gz --pe1-2 -1 H1_L1_2_trimmed.fastq.gz --pe1-1 -2 H1_L2_1_trimmed.fastq.gz --pe1-2 -2 H1_L2_2_trimmed.fastq.gz -o H1_spades" 


C. Annotation with prokka

                 prokka --outdir Prokka_annotation_p_ctg_oric --genus Sinorhizobium --species meliloti --strain 2011 --locustag SmelilotiFR --prefix FribourgSMeliloti_Prokka --rfam --usegenus p_ctg_oric.fasta
