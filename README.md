# Germline-and-somatic-variants

Calling somatic and germline variants. Data taken from [1]

Processing pipeline - Galaxy WF

WF takes trimmed read as inputs (Initial QC and read trimming not part of the WF) and performs QC of trimmed reads. 
Downstream processing steps include (only main steps)
(1) Mapping (BWA-MEM)
(2) Filtering of reads
(3) Removing duplicates (RmDup)
(4) Calling variants (VarScan somatic)
(5) Annotating variants (SnpEff)
(6) Predicting variant effects (VEP)

References 
[1] Project PRJNA1223657 (NCBI), High LOH MTMCT, To evaluate genomic alteration by PARP inhibitors for high LOH malignant transformation of mature cystic teratoma
