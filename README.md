# SynStream
GenoStream: Synthetic SNP-array &amp; Brain-imaging Volumetric Data for Korean Chronic Diseases

A lightweight, reproducible pipeline that generates fully synthetic genomic (SNP-array) and matched brain-imaging volumetric datasets for a Korean-style chronic-disease cohort (hypertension, type 2 diabetes, dyslipidemia). Disease-associated variants are curated from the GWAS Catalog across ancestries, embedded as phenotype-conditional causal loci, expanded into linkage-disequilibrium (LD) blocks, and validated by GWAS (PLINK). A matched brain-volume layer (66 ROIs, NeuroStream format) carries realistic age/disease/imaging-genetics effects.

Because every record is synthetic, the data contain no personal information and can be shared, taught, and benchmarked without disclosure risk.

Why this exists

Integrated cohorts linking genotype + neuroimaging + clinical data are powerful but access-restricted. This resource reproduces their statistical signal/noise structure so you can:

benchmark GWAS / association pipelines, LD clumping, and fine-mapping;
develop and teach imaging-genetics and multi-omics integration methods;
test tooling end-to-end with a documented ground truth of every planted effect.
Features
SNP array (~1,000,000 SNPs, 10,000 subjects) in chromosome-split PLINK binary format.
Three-tier signal design: strong causal loci (target genome-wide significant), candidate genes (target p > 1e-5), and a null background.
Ancestry-diverse causal loci curated from the GWAS Catalog (EAS / EUR / multi-ancestry).
LD blocks per locus (realistic association peaks), with tunable size / span / r².
Matched brain volumes (66 ROIs) with ICV scaling, age atrophy, disease effects, and imaging-genetics variants (APOE ε4, ENIGMA loci TESC/MSRB3/KTN1/HMGA2).
Ground-truth annotation (*.causal.tsv, effects table) and a full codebook.
GWAS result → Excel helper for reporting.
Repository structure
.
├── README.md
├── LICENSE                     # MIT
├── requirements.txt
├── CITATION.cff
├── .gitignore
├── src/
│   ├── make_synthetic_snp.py   # synthetic SNP-array generator (+ LD blocks, GWAS-Catalog loci)
│   ├── make_brain_volume.py    # matched brain-volume generator (NeuroStream format)
│   └── glm_to_xlsx.py          # PLINK2 --glm result -> formatted .xlsx
├── docs/
│   ├── snp_pipeline.md         # SNP construction method
│   ├── brain_pipeline.md       # brain-volume construction method
│   ├── gwas_validation.md      # how to run GWAS + interpret plots
│   ├── inputs.md               # input file formats
│   ├── Codebook_SNP_Genomic.xlsx
│   └── ko/                     # Korean SOP / manual (supplementary)
├── examples/
│   ├── example_cohort.txt      # 200 synthetic subjects (format demo, safe to publish)
│   ├── example_catalog.xlsx    # minimal SNP catalog (sheet "3. SNP 카탈로그")
│   ├── run_snp.sh              # end-to-end SNP demo
│   ├── run_brain.sh            # brain-volume demo (needs NeuroStream assets)
│   ├── run_gwas.sh             # PLINK2 GWAS + merge
│   └── plot_gwas.R             # Manhattan / QQ (qqman)
├── data/                       # put your own cohort/catalog here (git-ignored)
└── figures/                    # example outputs (GWAS + brain validation)
Requirements
Python ≥ 3.9 with numpy, pandas, openpyxl (see requirements.txt)
PLINK2 for GWAS (optional, for validation)
R with qqman for plots (optional)
bash
python -m pip install -r requirements.txt
Quick start
1) Synthetic SNP array
bash
python src/make_synthetic_snp.py \
  --cohort examples/example_cohort.txt \
  --catalog examples/example_catalog.xlsx \
  --n-snps 20000 --ld-block-size 10 \
  --outdir out_snp

The examples/ cohort has only 200 subjects (a format demo), so signals will not reach genome-wide significance. Use your full 10,000-subject cohort and --n-snps 1000000 for realistic GWAS peaks. The console prints a self-test of planted-locus significance.

2) Matched brain-imaging volumes
bash
git clone https://github.com/NIHxAI/NeuroStream        # ROI reference (once)
python src/make_brain_volume.py \
  --cohort examples/example_cohort.txt \
  --sample-dir NeuroStream/assets \
  --outdir out_brain
3) GWAS validation (optional)
bash
bash examples/run_gwas.sh out_snp out_gwas       # PLINK2 --glm per phenotype
Rscript examples/plot_gwas.R out_gwas            # Manhattan / QQ
python src/glm_to_xlsx.py out_gwas/dm_all.DM.glm.logistic.hybrid \
  --causal out_snp/synthetic_kchip.causal.tsv    # results -> Excel
Inputs
Input	Description
Cohort (.txt, tab)	T_ID, T_SEX(1=M/2=F), T_AGE, T_HTN, T_DM, T_LIP (1=normal, 2=disease, 99999=missing)
SNP catalog (.xlsx)	Sheet 3. SNP 카탈로그 with rsID, Paper 수, 유전자, 연관 형질·질환, 신뢰도/출처, …
NeuroStream assets	cohort*_sample_updated.csv (ROI means/SDs) — from the NeuroStream repo

See docs/inputs.md for full column specs.

Outputs

SNP array (chromosome-split PLINK): synthetic_kchip_chrN.{bed,bim,fam}, plus shared synthetic_kchip.{pheno,covar,causal.tsv,chrom_summary.tsv}. Brain volumes: brain_volume_synthetic_neurostream.csv (73-col NeuroStream format), brain_volume_synthetic_extended.csv (ID + APOE + brain SNPs, imaging-genetics), brain_effects_truth.tsv.

Full variable definitions and coding are in docs/Codebook_SNP_Genomic.xlsx.

Validation (example, full 10,000-subject cohort)

GWAS recovers the planted loci as LD-block peaks over a well-calibrated null; the strongest dyslipidemia signal localizes to APOE (chr19). See figures/.

이미지 표시	이미지 표시
GWAS Manhattan (DM / HTN / LIP)	QQ plots (HTN / LIP)

Brain layer reproduces hippocampal atrophy in disease / APOE ε4 carriers and ventricular enlargement in hypertension (figures/brain_validation.png).

Method summary
SNP: background SNPs under Hardy-Weinberg equilibrium; causal genotypes sampled conditional on phenotype via a per-allele logistic model; LD neighbors copy the lead allele with probability √(r²) decaying with distance. Details: docs/snp_pipeline.md.
Brain: per-ROI volumes anchored to reference means/SDs, modulated multiplicatively by ICV, age, disease, and genotype. Details: docs/brain_pipeline.md.
Citation

If you use this resource, please cite (see CITATION.cff):

Cho M, Park BS, Nam HR, Jeon JP, Kim SC. A synthetic SNP-array and brain-imaging volumetric dataset for Korean chronic diseases: construction and genome-wide association validation. Genomics & Informatics (submitted).

The brain ROI layer follows NeuroStream:

Cho M, Park BS, Nam HR, Jeon JP, Kim SC. NeuroStream: an interactive platform for exploratory visualization and harmonization of multicohort brain MRI data. Bioinform Adv 2026;6:vbag117. PMID: 42109578.

License

Code and synthetic data are released under the MIT License (see LICENSE). Brain ROI definitions follow the NeuroStream resource (Cho et al., Bioinform Adv 2026; PMID 42109578).

Disclaimer

All data are synthetic. Allele frequencies, effect sizes, and coordinates are illustrative values based on public catalog loci, not empirical estimates, and must not be used for clinical interpretation.
