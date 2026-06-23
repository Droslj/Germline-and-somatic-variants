# Analysis of Germline and somatic variants of MTMCT<br>
<br>

**Keywords**<br>
Somatic variant calling, PDX xenograft, Malignant Transformation of Mature Cystic Teratoma (MTMCT), PARP inhibitors (PARPi)<br>
<br>

**Objective**<br>
Analysis of Somatic mutations of a highly malignant tumor. Data taken from [1]<br>

Mature Cystic Teratoma (MCT) that undergoes malignant transformation (most commonly to squamous cell carcinoma), may be sensitive to PARP inhibitors, which work by exploiting lethality in tumor cells that cannot repair double-strand DNA breaks via the homologous recombination (HR) pathway.<br>
The purpose of this study was to detect any somatic mutations in tumor cells that could enable the tumor cells to survive treatment with PARP inhibitors or Cisplatin. <br>
<br>
# Experimental setup
<br>
Malignant Transformation of Mature Cystic Teratoma (MTMCT) was studied using a Patient-Derived Xenograft (PDX) mouse model treated with PARP inhibitors (PARPi), followed by Whole Exome Sequencing (WES).<br>
<br>
This study has following samples:<br>
 - Control DNA (SRR32341090) -> Untreated MTMCT PDX tumor represents a baseline against which all treated samples are compared<br> 
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
VarScan Somatic tool was run three times, comparing each treated sample to control sample. After that, Variants were annotated (SnpEff anotate) and Variant effects were predicted (Predict variant effects with VEP and Ensemble VEP).  

## Downstream filtering
<br>
DS Variant filtering selected only mutations with (Impact = {HIGH, Moderate}).<br>
<br>
After that, all Variants were tested against Ensemble score, which was created to account for pathogenicity (SIFT description = {deleterious, deleterious_low_confidence}) as well as Protein structure/evolution (PolyPhen description = {probably_damaging/possibly_damaging}). 
This filtering provided high impact, highly pathogenic (SIFT contribution) mutations that take into account both Protein structure as well as evolution (Polyphen contribution).  
<br>

## Population and biotype cleanup
<br>
After DS filtering step, another round of filtering consisting of Global alele frequency selection (AF <= 0.001), Biotype selection (BIOTYPE = protein_coding) and Clinical signinficance selection (CLIN_SIG = {pathogenic, likely_pathogenic, risk_factor}) was used to ensure getting High somatic confidence mutants. All the genes from all three treatments are presented in Table 1.


![Table 1](Images/Gene_list_final.png)

**Table 1: Gene list (final)
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


Same list is used to create Venn diagram (Figure 2) with all three treatment branches and intersections between branches highlighted.<br>
<br>

![Venn diagram](Images/Venn_diagram.png)
<br>

**Venn diagram comments**<br>
<br>
Venn diagram show a very well balanced entries for all three categories with ~fifty entries (53 (CDDP) vs 52 (OP) vs 51 (NP)) for each branch. The fact that the numbers are almost identical proves that lines shared similar genomic  backgrounds with just 1 or 2 specific variations to guide branching under distinct selection pressures of Cisplatin versus the PARP inhibitors. This also confirms that these specific, high-impact mutations are drivers of the tumor's malignant phenotype.<br>
<br>
## Categorization of entries
<br>
Filtering for gene variants with HIGH/MODERATE impact that are deleterious and damaging gives a list of protein coding genes that are detrimental to the normal function of those specific proteins and thus possibly beneficial for the tumor survival. These mutations found fall into one of the following categories:

**Energy Efficiency & Metabolic Rewiring**<br>
Tumors operating under continuous drug duress alter nutrient influx and mitochondrial logistics to sustain the intense ATP demands of DNA repair and continuous division. Following genes fall into this category:<br>
SLC25A48 (mitochondrial transporter), SLC6A18 (amino acid transporter), LPIN3 (lipid metabolic regulator), GK3 (glycerol kinase 3 — specific to OP), AADACL2 (esterase activity).<br>
<br>
**Evasion of Senescence & Apoptosis**<br>
The critical survival mechanism where clones aggressively silence or bypass internal cellular apoptosis triggered by Cisplatin-induced DNA cross-links or PARP failure. Following genes fall into this category:<br>
RHBDD1 (suppresses apoptotic signals), PTPRN (tyrosine phosphatase receptor-type N), FLII (flightless I actin remodeling regulator, blocks caspase activation cascades).<br>
<br>
These mutations could allow the tumor to systematically ignore internal damage checkpoints, converting what should be lethal drug stress into a survivable cellular state.<br>
<br>
**Loss of Adherence & Tissue Invasion (EMT)** <br>
To transform from a benign mature teratoma into an aggressive, infiltrating malignancy, cells must dismantle their rigid tissue anchorage. Following genes fall into this category:<br>
(CLCA2 (epithelial tight-junction regulator), PRTG (protocogenin adhesion molecule), ITGBL1 (integrin subunit beta like 1), PCDH7 (protococadherin 7)).<br>
<br>
Alterations in these structural proteins weaken epithelial anchoring, enabling the cell to undergo epithelial-to-mesenchymal transition (EMT) and migrate freely.<br>
<br>
**Structural Barriers (The Mucin Shield)** <br>
A heavily selected physical defense mechanism where tumors overproduce thick, highly glycosylated extracellular mucus matrices to physically impede small-molecule drug diffusion.<br>
MUC2, MUC3A, MUC4, MUC6.
<br>
Finding this complete mucin ensemble across all three arms may suggest that the tumor constructed a dense, physical microenvironmental coat to shield internal vulnerable cell-surface receptors from drug uptake.<br>
<br>
**Active Drug Efflux & Transport/Ion Clearance**<br>
Shifting electrochemical cell potential and altering internal pH proactively interferes with passive drug retention. Following genes fall into this category:<br>
KCNJ16 (inward-rectifier potassium channel), SLC9B1 (sodium-hydrogen exchanger — specific to CDDP).<br>
<br>
The presence of SLC9B1 exclusively in the Cisplatin arm points to a specialized chemical escape loop, as altering intracellular pH gradients is a classic tumor mechanism to neutralize heavy-metal accumulation.<br>
<br>
**Stress Tolerance & Autophagic Recycling**<br>
When PARP inhibitors shatter genomic structures, the tumor digests its damaged parts to generate raw recycling materials. Following genes fall into this category:<br>
ATG9B (core autophagosome structural lipid carrier), DCPS (scavenger decapping enzyme for damaged mRNA clearance).<br>
<br>
ATG9B acts as the primary waste-management engine, enabling the cell line to turn cytotoxic debris into usable metabolic building blocks rather than triggering cell death.<br>
<br>
**Epigenetic & Chromatin Reprogramming**<br>
Instead of mutating every gene independently, the tumor alters master chromatin remodelers to sweepingly change which survival programs are open or closed for transcription. Following genes fall into this category:<br>
KMT2C (essential lysine methyltransferase), TCERG1L (transcription elongation regulator), PAGR1 (PA1-dependent chromatin regulator).<br>
<br>
**Immune Evasion & Signaling Blinding**<br>
Suppressing or altering cell-surface receptor expressions to mask the tumor from host local surveillance and silence immune signaling cascades. Following genes fall into this category:<br>
TNFRSF17 (BCMA receptor), IFNA7 (Interferon Alpha 7), LGALS9B (Galectin 9B — specific to NP).<br>
<br>
Modulating tumor necrosis factors and interferon pathways blinds the local microenvironment, suppressing local inflammatory responses that would otherwise target the malignant graft.<br>
<br>
**Chromosomal Instability & Mitotic Shuffling** <br>
Structural strategy which allows the clone to introduce subtle, manageable defects into its cell-division machinery to rapidly shuffle its chromosome variations and accelerate drug-resistance evolution. Following genes fall into this category:<br>
CEP350 (centrosomal microtubule anchor), CFAP20DC, CCDC32, CCT8L2, UBE2W (ubiquitin-conjugating enzyme vital for DNA damage bypass).<br>
<br>
Finding CEP350 presence across all lineages proves that a permanent structural shakeup in mitotic spindle mechanics acts as a foundational engine for genomic variation in this tumor model.
<br>
**Phenotypic Plasticity & Stemness (Progenitor Maintenance)** <br>
Because a teratoma originates from germ cells, blocking differentiation pathways keeps the tumor cells locked in an immortal, highly plastic, embryonic-like progenitor state. Following genes fall into this category:<br>
LGR6 (definitive stem-cell marker), AGAP6, AKAP13.<br>
<br>
LGR6 is highly significant. It proves the tumor maintains an undifferentiated, highly flexible stem-like state, ensuring infinite self-renewal capability regardless of the treatment wing.
<br>
<br>

**References**<br> 
[1] Project PRJNA1223657 (NCBI), High LOH MTMCT, To evaluate genomic alteration by PARP inhibitors for high LOH malignant transformation of mature cystic teratoma
