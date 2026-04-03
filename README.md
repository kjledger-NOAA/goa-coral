# goa-coral

### MiFish Libraries 
1. removed primers from demultiplexed amplicon reads using cutadapt 
in: /Users/kimberly.ledger/Documents/goa-coral/data/20260325_GOAcoral_MiFish/
- conda activate cutadptenv
- DATA=/Users/kimberly.ledger/Documents/goa-coral/data/20260325_GOAcoral_MiFish/
- NAMELIST=$(ls ${DATA} | sed 's/e*_L001.*//' | uniq)
- echo "${NAMELIST}"
- mkdir trimmed
- for i in ${NAMELIST}; do
   cutadapt --discard-untrimmed -g GCCGGTAAAACTCGTGCCAGC -G CATAGTGGGGTATCTAATCCCAGTTTG -o trimmed/${i}_R1.fastq.gz -p trimmed/${i}_R2.fastq.gz "$DATA/${i}_L001_R1_001.fastq.gz" "$DATA/${i}_L001_R2_001.fastq.gz";
done
- pigz -d trimmed/*.gz
- mkdir trimmed/filtered
- mkdir trimmed/filtered/outputs

2. process reads using '1_MiFish_sequence_filtering.Rmd'

3. perform taxonomic assignment using MIDORI2 database 
makeblastdb -in MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST.fasta \
            -dbtype nucl \
            -out MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db/MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db

blastn -query trimmed/filtered/outputs/myasvs.fasta \
       -db /Users/kimberly.ledger/Documents/metaprobes_BeringSea2024/MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db/MIDORI2_UNIQ_NUC_GB267_srRNA_BLAST_db \
       -out trimmed/filtered/outputs/blastnresults \
       -perc_identity 96 -qcov_hsp_perc 100 \
       -outfmt "6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore sscinames staxids" \
       -num_threads 10
       
4. Run lulu to identify any asv's to merge usign "2_MiFish_lulu.Rmd"

5. filter blastn results using '3_MiFish_blastn_taxonomy.Rmd'

*** note: i moved some files/folders around to be able to upload small files/outputs to github and avoid the fastqs, so some file paths may now be inaccurate... 

6. curate and decon the mifish libraries using "4_curate_and_decon.Rmd"

7. summarize detections and compare extraction protocols using "5_MFish_summary.Rmd"


### Rockfish libraries 

1. removed primers from demultiplexed amplicon reads using cutadapt 
in: /Users/kimberly.ledger/Documents/goa-coral/fastq/20260325_GOAcoral_Rockfish/
- conda activate cutadptenv
- DATA=/Users/kimberly.ledger/Documents/goa-coral/fastq/20260325_GOAcoral_Rockfish/
- NAMELIST=$(ls ${DATA} | sed 's/e*_L001.*//' | uniq)
- echo "${NAMELIST}"
- mkdir trimmed
- for i in ${NAMELIST}; do
   cutadapt --discard-untrimmed -g ATNACCATATCTAGGNTTNAACC -G TGRRCTTGTTGGTCGGYT -o trimmed/${i}_R1.fastq.gz -p trimmed/${i}_R2.fastq.gz "$DATA/${i}_L001_R1_001.fastq.gz" "$DATA/${i}_L001_R2_001.fastq.gz";
done
- pigz -d trimmed/*.gz
- mkdir trimmed/filtered
- mkdir ../../data/rkfish_outputs

2. process reads using '6_Rkfish_sequence_filtering.Rmd'

3. perform taxonomic assignment using custom rockfish dloop database

makeblastdb -in data/rkfish_db/rockfish_reference_db_534_20250117.fasta -dbtype nucl -out data/rkfish_db/rockfish_db_534_20250117

blastn -query data/rkfish_outputs/myasvs.fasta \
       -db data/rkfish_db/rockfish_db_534_20250117 \
       -out data/rkfish_outputs/blastn_rkfish.txt \
       -perc_identity 92 \
       -qcov_hsp_perc 98 \
       -num_threads 10 \
       -outfmt '6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore sscinames staxids'