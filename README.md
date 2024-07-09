# Rhizobia_denovo_Assemblies
Pipeline for denovo assemblies from illumina sequencing

0. Change reads names
            mv H6_L1_R2_001_852WMH6HscGS.fastq.gz  H6_L1_R2.fastq.gz

A. Clean reads with fastp


             for FILE in $(ls *_R1.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1,2)fastp --time=0-01:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1,2)_fastp.out --error=$(echo $FILE | cut -d'_' -f1,2)_fastp.error --mail-type=END,FAIL --wrap " cd /data/projects/p495_SinorhizobiumMeliloti/07_Axelle/03_denovoNifh_v2; module load FastQC; ~/00_Software/fastp --in1 $FILE --in2 $(echo $FILE | cut -d'_' -f1,2)_R2.fastq.gz --out1 02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_1_trimmed.fastq.gz --out2 02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_2_trimmed.fastq.gz -h 02_TrimmedData/$(echo $FILE | cut -d',' -f1,2)_fastp.html --thread 4; fastqc -t 4 02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_1_trimmed.fastq.gz; fastqc -t 4 02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_2_trimmed.fastq.gz"; sleep  1; done


B. denovo assemblies with SPAdes

    for FILE in $(ls *_1_trimmed.fastq.gz); do echo $FILE; sbatch --partition=pibu_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_spaDES --time=7-48:00:00 --mem-per-cpu=128G --ntasks=8 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_spaDES.out --error=$(echo $FILE | cut -d'_' -f1)_spaDES.error --mail-type=END,FAIL --wrap "module load SPAdes/3.15.3-GCC-10.3.0; cd /data/projects/p774_MARSD/IVAN/02_MAR_DATA/01_Delf/01_raw/02_TrimmedData; spades.py -k 21,33,55,77 --isolate --pe1-1 $FILE --pe1-2 $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz -o ../03_$(echo $FILE | cut -d'_' -f1)_spaDES" ; done 

B2. Continue denovo assemblies with SPAdes if stopped before

    sbatch --partition=pibu_el8 --job-name=H1_spaDES --time=72:00:00 --mem-per-cpu=64G --ntasks=8 --cpus-per-task=1 --output=H1_spades.out --error=H1_spades.error --mail-type=END,FAIL --wrap "cd /data/projects/p495_SinorhizobiumMeliloti/07_Axelle/03_denovoNifh_v2/02_TrimmedData; /home/imateusgonzalez/00_Software/SPAdes-3.15.5-Linux/bin/spades.py --continue -o H1_spades" 


B3. Reduce nb. of contigs, depending on lenght and coverage. with fastagrep.pl script by Tim Booth - tbooth@ceh.ac.uk, http://nebc.nerc.ac.uk


            for file in scaffolds.fasta; do grep -F “>” $file | sed -e ‘s/_/ /g’ |sort -nrk 6 |awk ‘$6>=2.0 && $4>=1000 {print $0}’| sed -e ‘s/ /_/g’|sed -e ‘s/>//g’>$file.txt; echo sequences to keep; wc -l $file.txt ;echo running fastagrep.pl; /data/projects/p774_MARSD/IVAN/02_MAR_DATA/ZZ_tools/fastagrep.pl -f $file.txt $file > 01_filteredScaffolds/HCov.$file; echo sequences kept ;grep -c “>” 01_filteredScaffolds/HCov.$file; done


B4. Change name to include filename

            mv HCov.scaffolds.fasta B11_HCov.scaffolds.fasta
            
C. Annotation with prokka

                  

                                    sbatch --partition=pibu_el8 --job-name=B06PROKKA --time=12:00:00 --mem-per-cpu=64G --ntasks=12 --cpus-per-task=1 --output=B06prokka.out --error=B06_prokka.error --mail-type=END,FAIL --wrap "module load prokka/1.14.5-gompi-2021a; cd /data/projects/p774_MARSD/IVAN/02_MAR_DATA/01_Delf/01_raw/03_B06_spaDES/01_filteredScaffolds/; prokka --outdir ../02_Prokka --genus bla --species bla --strain bla --locustag bla --prefix B06 --rfam --force --cpus 12 --usegenus B06_HCov.scaffolds.fasta"

D. BUSCO


                        sbatch --partition=pibu_el8 --job-name=B06Busco --time=0-10:00:00 --mem-per-cpu=50G --ntasks=12 --cpus-per-task=1 --output=B06_Busco.out --error=B06_Busco.error --mail-type=END,FAIL --wrap "module load BUSCO; cd /data/projects/p774_MARSD/IVAN/02_MAR_DATA/01_Delf/01_raw/03_B06_spaDES/01_filteredScaffolds/; busco -o ../03_BUSCO -i B06_HCov.scaffolds.fasta -l bacteria_odb10 --cpu 12 -m genome -f"


E. QUAST

            sbatch --partition=pshort_el8 --job-name=H1_spaDES --time=01:00:00 --mem-per-cpu=64G --ntasks=8 --cpus-per-task=1 --output=B06_quast.out --error=B06_quast.error --mail-type=END,FAIL --wrap "cd /data/projects/p774_MARSD/IVAN/02_MAR_DATA/01_Delf/01_raw/03_B06_spaDES; module load QUAST/5.0.2-foss-2021a; quast scaffolds.fasta  -o Quast"
