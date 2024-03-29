# 35S_rRNA_for_Natalia
This is for Natalia to do the 35S-rRNA data analysis

Natalia generated the CCS reads from the pre-rRNA from three B hybridum accessions. And the goal was to verify if there are any differences in the pre-rRNA transcripts between the three B. hybridum genotypes that are characterised by differential 35S rDNA activity from both subgenomes. Her original email is below:

```
Li, the study's main goal was to verify if there are any differences in the pre-rRNA transcripts between the three B. hybridum genotypes that are characterised by differential 35S rDNA activity from both subgenomes. According to our data (based on RT-CAPS and RT-qPCR):
    • ABR113: there is a strong nucleolar dominance (ND) towards the D-genome 35S rDNA in almost all studied tissues (there is a slight expression of the S-genome rDNA in the adventitious roots);
    • 3-7-2: ND is present in leaves, but not in roots. The level of the S-subgenome 35S rRNA gene expression is higher in the adventitious roots than in the primary ones.
    • 2.2: the S-genome rDNA is dominant in primary and adventitious roots; in the case of leaves – the rDNA expression levels vary between the individuals.
We would like to take a closer look at the pre-rRNA variants in both leaves and adventitious roots. The pre-rRNA before the processing is approx. 6kb-long. It contains the external transcribed spacer (ETS) and two internal transcribed spacers (ITS1 and 2). It is possible to differentiate the transcripts from the D- and S-subgenome based on these sequences. If you can help us with the analysis of D- and S-subgenome contributions in these three genotypes and the analysis of the existing length and sequence variants, we would be very grateful.
```

She sent me the 35S rDNA sequences from the Bd21 (D subgenome) and ABR114(S subgenome).

35SrDNA = ETS + 18S + ITS1 + 5.8S + ITS2 + 28S

pre-rRNA should be the same as the rDNA except for the replacement of T by U. However, the length of the transcripts may vary since you will have the pre-rRNA at different processing stages. In plants, there are at least three pathways of pre-rRNA processing that govern either the ITS1-first or 5’ETS-first processing or plant-specific ITS2 processing as the first one.

Here are the process I have done:

### 1 download the data from the JGI PORTAL.

### 2 process

1> concatenate a fake genome of the 35S rDNA for both distachyon and stacei

```
cd /global/u2/l/llei2019/plantbox/Natalia_rRNA/reference_fake_chrs
cat ABR114_complete35SrDNA_consensus.fa Bd21_complete35SrDNA_consensus_1.fa >reference_fake_35S.fa

```

2> Index the reference:

```    
cd /global/u2/l/llei2019/plantbox/Natalia_rRNA/reference_fake_chrs/

minimap2 -d reference_fake_35S.fa.mmi reference_fake_35S.fa
```

3> Process the CCS reads by Jie Guo

(1) Taking pacbio subreads as the input to generate the polished Circular Consensus Sequence (also called CCS reads) by pbccs (version 4), here the accuracy rate we set in pbccs is 98% and minimum pass is 3. ( ccs --min-passes 3 --min-snr 4 --m
ax-length 21000 --min-length 10 --min-rq 0.98 )
(2) Using lima to classify if these CCS reads is full length or not according to if it contains 5' primer, 3' primer and polyA tail or not. Only the full length reads were kept for further classification
(3) Using 'isoseq3 refine' to remove polyA tails

4> Read mapping with the `*.flnc.fasta.gz: non chimeric full length ccs reads`

```
    #HYYNG
    minimap2 -ax splice:hq -uf --MD /global/u2/l/llei2019/plantbox/Natalia_rRNA/reference_fake_chrs/reference_fake_35S.mmi /global/u2/l/llei2019/pscratch/Natalia_related/3-7-2/Reads/HYYNG.flnc.fasta.gz >HYYNG_flnc.aligned.sam
     
     
    # Convert SAM to BAM 
    samtools view -bS HYYNG_flnc.aligned.sam > HYYNG_flnc.aligned.bam
     
    # Sort BAM 
    samtools sort HYYNG_flnc.aligned.bam -o HYYNG_flnc.aligned.sorted.bam
     
    # Index BAM 
    samtools index HYYNG_flnc.aligned.sorted.bam
     
    #count the reads mapping back to each subgenomes of the 35S
     
     samtools idxstats HYYNG_flnc.aligned.sorted.bam
   
    Bd21_complete35S_rDNA_consensus-1       8084    360014  0
    ABR114_complete35SrDNA_consensus        9215    359748  0
    *       0       0       437220

    This command outputs a tab-separated list with four columns:
    
    Reference sequence name (chromosome name)
    Reference sequence length
    Number of mapped reads
    Number of unmapped reads
    
```
The full reasults can be seen [here](https://docs.google.com/spreadsheets/d/13vjSLVGcbKkCOakw5stNA2JZCbuLx_H4n_cPA_BSGFA/edit#gid=0).


