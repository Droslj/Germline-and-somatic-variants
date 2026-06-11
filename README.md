# Analysis of Germline and somatic variants<br>
<br>
**Keywords**<br>
Somatic variant calling, PDX xenograft, Malignant Transformation of Mature Cystic Teratoma (MTMCT), PARP inhibitors (PARPi)<br>
<br>
**Objective**<br>
MTMCT that undergoes malignant transformation (most commonly to squamous cell carcinoma), may be sensitive to PARP inhibitors, which works by exploiting synthetic lethality in tumors that cannot repair double-strand DNA breaks via the homologous recombination (HR) pathway.<br>
The purpose of this study was to detect any somatic mutations in tumor cells that could enable the tumor cells to survive treatment with PARP inhibitors. <br>

# Introduction
Analysis of Somatic mutations of a highly malignant tumor. Data taken from [1]<br>
<br>
This study is about Malignant Transformation of Mature Cystic Teratoma (MTMCT) using a Patient-Derived Xenograft (PDX) mouse model treated with PARP inhibitors (PARPi), followed by Whole Exome Sequencing (WES).<br>
<br>
Mature cystic teratomas (termed dermoid cysts) originate from partogenous activation of a single germ cell that has undergone faulty oogenesis. These benign tumors inherently possess massive, genome-wide blocks of homozygosities — meaning that they exhibit high baseline LOH without necessarily being deficient in DNA repair.<br>
<br>
# Experimental setup
<br>
This study has following samples:<br>
 - Control DNA (SRR32341090) -> Untreated MTMCT PDX tumor represents a baseline against all treated samples are compared<br> 
 - CDDP_DNA (SRR32341089) -> Cisplatin treatment induces bulky DNA cross-links. Tumors with HR deficiency are sensitive to both Cisplatin and PARP inhibitors, so this can serve as an excellent cytotoxic benchmark<br>
 - Olaparib_DNA (SRR32341088) -> Residual tumor under Olaparib (PARPi) selective pressure<br>
 - Niraparib_DNA (SRR32341087) -> Residual tumor under Niraparib (PARPi) selective pressure.<br>

Since background meiotic LOH is present in all samples, the focus is on treatment-induced somatic alteration. This is done by comparing all treated samples to control sample.
<br>
# WES Pipeline
<br>
The complete pipeline I used to process data from this study is shown on Figure 1.<br>
<br>


![Processing pipeline]/Images/Processing_pipeline.png
<br>
## Quality Control & Adapter Trimming
<br>
Inital preprocessing steps included QC and adapter trimming (fastp).<br>
<br>
## Mouse Stroma Deconvolution
<br>
Sequencing of a residual mouse PDX tumor means that the sample will contain varying amounts of mouse stromal cells infiltrating the human tumor mass. Before running variant or copy-number callers, raw reads need to be filtered in order to separate human from mouse reads. Separating mouse residual reads from graft (human) was done using Dual-Alignment strategy:<br>

 - Step 1: Align reads to Mouse Genome (mm39) (Minimap2)<br>
 - Step 2: Extract the unmapped reads (Samtools fastx)<br>
 - Step 3: Align the unmapped reads to the Human Genome (hg38) (Minimap2)<br>
 - Step 4: Sort reads (Samtools sort)<br>
 - Step 5: Mark duplicates (Mark duplicates).<br>
<br>
Processing reads in this way ensures that any read moving forward into variant calling pipeline is human tumor DNA.<br>
<br>
## Variant calling & Annotation
<br>
VarScan Somatic tool was run three times, comparing each treated sample to control sample. After that, only true somatic entries were extracted (Snpsift filter) and annotated (SnpEff annotate). Variant effects were predicted (Predict variant effects with VEP) and only those that carry Impact level HIGH or MODERATE were selected for final list. All the genes from all three treatments, filtered and shown with the impact are presented in Table 1.



![Table 1](Images/Gene_list_final.png)

Figure 2 shows VENN diagram with all three treatment branches and highlights intersections between branches.<br>
<br>

![Venn diagram](Images/Venn_diagram.png)
<br>
<br>

## Discussion
<br>
The filtered list contains a collection of mutated genes damaged under the drug pressure. If a mutation completely broke an essential housekeeping gene that the cell needs to stay alive, that cell would have died before the sequencing run  and its DNA would have vanished from the sample. They represent one of the following:<br>
 - Category 1: Mutations that enable the tumor to bypass cell safety mechanism and survive<br>
 - Category 2: Mutations that enable the tumor to adapt survival and defense mechanisms<br> 
 - Category 3: Mutations that enable the tumor to modify extracellular mechanisms and response to immune system.<br>
<br>
Category 1 (HIGH Impact Loss-of-Function)<br>
Tumor uses complete protein destruction to disable its own cellular safety trigger<br>
 - BNIP1 (HIGH in Cisplatin): It normally acts as a pro-apoptotic sensor that forces heavily damaged cells to undergo suicide. By knocking it out, the tumor cell survives platinum cross-links because it can no longer trigger apoptosis<br>
 - PPP1R15A (HIGH in Olaparib): It normally forces a cell experiencing massive replication stress into strict growth arrest. Truncating it allows the tumor to bypass the checkpoint and keep dividing despite trapped PARP complexes.<br>
<br>
Category 2 (MODERATE Impact - Missense)<br>
Tumor uses single amino acid tweaks to over-activate pathways or optimize cellular machinery to handle continuous stress without dying<br>
 - IRS2 (MODERATE) in Olaparib: Instead of destroying the protein, this missense modification alters its shape to continuously pump pro-survival PI3K/Akt signaling throughout the cell<br>
 - NEK11 (MODERATE) in Niraparib: It subtly adjusts the G2/M DNA damage checkpoint clock. It doesn't break the cell cycle entirely; it tunes it just enough to let the cell patch its replication forks before dividing.<br>
<br>
Category 3 (MODERATE Impact)<br>
Tumor modifies its exterior surface to protect its newly transformed epithelial components and hide from immune clearing <br>
 - MUC16 (MODERATE): Mutating the CA-125 matrix architecture helps tracking and anchoring the malignant cells during targeted selection <br>
 - HLA-DQA2 (MODERATE): Altering these major histocompatibility complexes serves to mute local microenvironmental immune surveillance while the tumor undergoes massive structural genomic shifts.<br>
<br>

**References**<br> 
[1] Project PRJNA1223657 (NCBI), High LOH MTMCT, To evaluate genomic alteration by PARP inhibitors for high LOH malignant transformation of mature cystic teratoma
