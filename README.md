# Analysis of Germline and somatic variants of MTMCT<br>
<br>

**Keywords**<br>
Somatic variant calling, PDX xenograft, Malignant Transformation of Mature Cystic Teratoma (MTMCT), PARP inhibitors (PARPi)<br>
<br>

**Objective**<br>
MTMCT that undergoes malignant transformation (most commonly to squamous cell carcinoma), may be sensitive to PARP inhibitors, which work by exploiting lethality in tumor cells that cannot repair double-strand DNA breaks via the homologous recombination (HR) pathway.<br>
The purpose of this study was to detect any somatic mutations in tumor cells that could enable the tumor cells to survive treatment with PARP inhibitors or Cisplatin. <br>

# Introduction
Analysis of Somatic mutations of a highly malignant tumor. Data taken from [1]<br>
<br>
This Case study is about Malignant Transformation of Mature Cystic Teratoma (MTMCT) using a Patient-Derived Xenograft (PDX) mouse model treated with PARP inhibitors (PARPi), followed by Whole Exome Sequencing (WES).<br>
<br>
Mature cystic teratomas (termed dermoid cysts) originate from partogenous activation of a single germ cell that has undergone faulty oogenesis. <br>
These benign tumors inherently possess massive, genome-wide blocks of homozygosities — meaning that they exhibit high baseline LOH without necessarily being deficient in DNA repair.<br>
<br>
# Experimental setup
<br>
This study has following samples:<br>
 - Control DNA (SRR32341090) -> Untreated MTMCT PDX tumor represents a baseline against all treated samples are compared<br> 
 - CDDP_DNA (SRR32341089) -> Cisplatin treatment induces bulky DNA cross-links. Tumors with HR deficiency are sensitive to both Cisplatin and PARP inhibitors<br>
 - Olaparib_DNA (SRR32341088) -> Residual tumor under Olaparib (PARPi) selective pressure<br>
 - Niraparib_DNA (SRR32341087) -> Residual tumor under Niraparib (PARPi) selective pressure.<br>
<br>
Since background meiotic LOH is present in all samples, the focus is on treatment-induced somatic alteration. This is done by comparing all treated samples to control sample, which subtracts the background meiotic LOH.
<br>

# WES Pipeline
<br>
The complete pipeline I used to process data from this study is shown on Figure 1.<br>
<br>

![Processing pipeline](Images/Complete_flow.png)
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
VarScan Somatic tool was run three times, comparing each treated sample to control sample. After that, only true somatic entries were extracted (Snpsift filter) and annotated (SnpEff annotate). 
Variant effects were predicted (Predict variant effects with VEP) and only those that carry Impact level HIGH or MODERATE were selected.


<br>
<br>
List of Somatic mutations for CDDP sample<br>

![List CDDP](Data/Somatic_mutations_CDDP.tabular)<br>
<br>
<br>
List of Somatic mutations for Olaparib sample<br>

![List OP](Data/Somatic_mutations_OP.tabular)<br>
<br>
<br>
List of Somatic mutations for Niraparib sample<br>

![List NP](Data/Somatic_mutations_NP.tabular)<br>
<br>
<br>
All the genes from all three treatments, filtered to contain a single entry for each gene and shown only with the modification impact are presented in Table 1.

![Table 1](Images/Gene_list_final.png)

Same list is used to create Venn diagram (Figure 2) with all three treatment branches and intersections between branches highlighted.<br>
<br>

![Venn diagram](Images/Venn_diagram.png)
<br>

**Venn diagram comments**
Sector 1: The Exclusive PARPi Core (Olaparib & Niraparib)
These 4 genes are mutated exclusively under PARP inhibitor pressure (OP and NP), but remain completely untouched by classic chemotherapy (CDDP).
 - MUC16 (MODERATE): Mutated in both OP and NP. This confirms tracking of the malignant epithelial component during targeted therapy.
 - HLA-DQA2 (MODERATE): Mutated in both OP and NP. Confirms targeted selective sweeps focused on immune evasion markers.
 - DUX4 (MODERATE): Mutated in both OP and NP. A major pioneer transcription factor involved in early development and facioscapulohumeral muscular dystrophy pathways, showing clear germ-cell line dynamics.
 - TBCD (HIGH vs MODERATE conflict): This is an exceptional finding. TBCD shows a HIGH impact mutation in Niraparib, but it also appears in your Cisplatin list as HIGH. This means TBCD actually shifts sectors!

Sector 2: The Universal Stress Core (OP & NP & CDDP) — 2 Genes Total
These are the 2 genes sitting dead-center in your Venn diagram, capturing general structural fragility.
MUC4 (MODERATE): Universally mutated across all three arms.
GOLGA6L1 (MODERATE): Universally mutated across all three arms.

Sector 3: The Niraparib-Specific Wing (NP Only) — 8 Genes Total
These are mutations unique to the Niraparib microenvironment. Notably, almost all of them are finely tuned missense modifications (MODERATE) rather than absolute structural blowouts (HIGH).
NEK11 (MODERATE): Critical G2/M DNA damage checkpoint tuning.
HLA-DRB5 (MODERATE): Additional antigen-presentation pathway adjustment.
USP17L29, ZAN, MUC3A, SPDYE2, GOLGA8N, ARMCX4, ENSG00000286185, NBPF19 (MODERATE).

Sector 4: The Olaparib-Specific Wing (OP Only) — 27 Genes Total
Olaparib shows a distinct evolutionary strategy. It relies on a heavy combination of protein modifications (MODERATE) paired with abrupt functional knockouts (HIGH). You should explicitly label these HIGH genes in bold on your report:
AGRN (HIGH): Agrin disruption, affecting extracellular matrix structuring.
DIS3L2 (HIGH): Exoribonuclease disruption, driving mitotic abnormalities and cellular overgrowth syndromes.
CPE (HIGH): Carboxypeptidase E knockout.
LCOR (HIGH): Ligand-dependent corepressor knockout, heavily involved in altering chromatin structure and transcriptional silencing.
FBN1 (HIGH): Fibrillin-1 structural frame truncation.
VPS35 (HIGH): Vacuolar protein sorting 35 knockout, destroying endosomal retrograde transport to the Golgi network.
PPP1R15A (HIGH): Major pathway hit. This is GADD34, a critical regulator of the cellular stress response, growth arrest, and DNA damage recovery cascades. Knocking this out shifts how the cell enters apoptosis vs. senescence.
Key Metabolic Modulators: IRS2 (MODERATE), WWP2 (MODERATE), USP42 (MODERATE).

Sector 5: The Cisplatin Genotoxic Wing (CDDP Only) — 29 Genes Total
Cisplatin tracks standard chemotherapy defense mechanics, focusing heavily on base excision/DNA repair modification and cell-death controllers (HIGH):
BNIP1 (HIGH): BCL2/Adenovirus E1B 19kDa Interacting Protein 1. This is a direct pro-apoptotic engine knockout, showing how cisplatin-treated cells explicitly delete death-signaling networks to survive.
WASHC1 (HIGH vs MODERATE cluster): Shows multi-allelic pressure with both a severe structural truncation and a secondary missense mutation running concurrently.
CCDC30, SPOUT1, PPP1R13L, C20orf96 (HIGH).Key DNA Repair Engine: 
APEX2 (MODERATE).

The Shared Core: MUC16, MUC4, HLA-DQA2, GOLGA6L1. (Label these as: Universal structural/environmental adaptations to PARP inhibition).
The Olaparib Special: IRS2, WWP2, USP42, OVGP1. (Label these as: Targeted metabolic signaling and DNA-damage checkpoint modifications).

# Discussion
<br>
The filtered list contains a collection of mutated genes damaged under the drug pressure. If a mutation completely broke an essential housekeeping gene that the cell needs to stay alive, that cell would have died before the sequencing run and its DNA would have vanished from the sample. So, in the final list I focused on genes whose mutation would have been beneficial to the survival of the tumor cell:<br>
 - Category 1: Mutations that enable the tumor to bypass cell safety mechanism and survive<br>
 - Category 2: Mutations that enable the tumor to adapt survival and defense mechanisms<br> 
 - Category 3: Mutations that enable the tumor to modify extracellular mechanisms and response to immune system.<br>
<br>

**Category 1** (HIGH Impact Loss-of-Function)<br>
Tumor benefits from non-functional protein to disable its own cellular safety triggers for cell senescence/apoptosis. These are possible options:<br>
 - BNIP1 (HIGH in Cisplatin)<br>
   - It normally acts as a pro-apoptotic sensor that forces heavily damaged cells to undergo suicide<br>
   - By knocking it out, the tumor cell survives platinum cross-links because it can no longer trigger apoptosis<br>
 - PPP1R15A (HIGH in Olaparib)<br>
   - It normally forces a cell experiencing massive replication stress into strict growth arrest<br>
   - Truncating it allows the tumor to bypass the checkpoint and keep dividing despite trapped PARP complexes<br>
<br>

**Category 2** (MODERATE Impact - Missense)<br>
Tumor uses single amino acid tweaks to over-activate pathways or optimize cellular machinery to handle continuous stress without dying:<br>
 - IRS2 (MODERATE in Olaparib)<br>
   - Instead of destroying the protein, this missense modification alters its shape to continuously pump pro-survival PI3K/Akt signaling throughout the cell<br>
 - NEK11 (MODERATE) in Niraparib)<br>
   - It subtly adjusts the G2/M DNA damage checkpoint clock. It doesn't break the cell cycle entirely; it tunes it just enough to let the cell patch its replication forks before dividing.<br>
<br>

**Category 3** (MODERATE Impact)<br>
Tumor modifies its exterior surface to protect its newly transformed epithelial components and hide from immune clearing <br>
 - MUC16 (MODERATE in Olaparib, Niraparib)<br>
   - Mutating the CA-125 matrix architecture helps tracking and anchoring the malignant cells during targeted selection <br>
 - HLA-DQA2 (MODERATE in Olaparib, Niraparib)<br>
   - Altering these major histocompatibility complexes serves to mute local microenvironmental immune surveillance while the tumor undergoes massive structural genomic shifts.<br>
<br>

**References**<br> 
[1] Project PRJNA1223657 (NCBI), High LOH MTMCT, To evaluate genomic alteration by PARP inhibitors for high LOH malignant transformation of mature cystic teratoma
