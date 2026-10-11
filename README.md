# Divergent Chromatin Accessibility Landscapes at Cell-Cycle and Developmental Signaling Loci in Alzheimer's Disease and Medulloblastoma

## Project overview

This repository documents a student bioinformatics research project comparing publicly available ATAC-seq datasets associated with Alzheimer's disease (AD) and medulloblastoma (MB). The project investigates similarities and differences in chromatin accessibility, with particular attention to genomic regions associated with cell-cycle regulation and developmental signaling.

ATAC-seq (Assay for Transposase-Accessible Chromatin using sequencing) identifies regions of open chromatin that may contain regulatory elements. By comparing accessible regions across the selected datasets, this project aims to characterize shared chromatin regions and condition-associated accessibility differences, and to explore the genomic annotations, sequence motifs, and functional terms associated with these regions.

**Important:** The title describes the research focus. Claims about specific cell-cycle or developmental signaling findings should be supported by the final analysis outputs and validated gene/pathway lists; this README does not assume that any particular pathway was significantly enriched.

## Research objectives

- Assess the quality of the selected public ATAC-seq datasets.
- Align genomic paired-end reads to the human GRCh38/hg38 reference genome.
- Identify accessible chromatin regions in the AD and MB datasets.
- Characterize genomic features represented in each condition's peak set.
- Identify overlapping accessible intervals between AD and MB peak sets.
- Annotate accessible regions by genomic location.
- Explore enriched DNA sequence motifs in shared and condition-associated regions.
- Compare chromatin accessibility between the two conditions using DiffBind.
- Perform functional enrichment analysis of genes associated with the analyzed regions, where appropriate.

## Data sources

Sequencing data were obtained from the NCBI Sequence Read Archive (SRA).

| Condition | SRA accessions |
|---|---|
| Alzheimer's disease (AD) | `SRR26205319`, `SRR26205320`, `SRR26205321`, `SRR26205322` |
| Medulloblastoma (MB) | `SRR18324548`, `SRR18324549`, `SRR18324550`, `SRR18324551` |

Four SRA accessions were included for each condition. The reference genome assembly used for alignment and genomic analysis was human GRCh38/hg38.

- NCBI SRA: https://www.ncbi.nlm.nih.gov/sra
- NCBI SRA Toolkit: https://github.com/ncbi/sra-tools

### Read organization

The downloaded sequencing data included barcode, genomic, and UMI read types. In the primary pseudo-bulk workflow, `_2` files were treated as forward genomic reads and `_4` files as reverse genomic reads. Barcode (`_1`) and UMI (`_3`) reads were not used as the primary genomic alignment input. Confirm the read layout against each accession's metadata before reproducing the analysis.

## Methods overview

The analysis workflow is summarized below.

1. **Data retrieval:** Download the selected SRA accessions and prepare genomic paired-end reads.
2. **Quality control:** Use FastQC to assess per-base sequence quality, GC content, read length, duplication levels, and overrepresented sequences.
3. **Read alignment:** Align paired-end genomic reads to GRCh38/hg38 using BWA-MEM.
4. **BAM processing:** Convert, sort, and index alignment files using SAMtools. Assess duplicate reads and document the handling applied before downstream analyses.
5. **Peak calling:** Use MACS2 with paired-end BAM input to identify accessible genomic intervals for the AD and MB datasets.
6. **Shared-region analysis:** Use BEDTools `intersect` to identify overlapping intervals between the AD and MB peak sets.
7. **Genomic annotation:** Use HOMER to annotate accessible intervals as promoter, exon, intron, intergenic, or other regions.
8. **Motif enrichment:** Use HOMER to identify sequence motifs enriched in the selected shared or condition-associated regions.
9. **Differential accessibility:** Use DiffBind to compare accessibility across the four AD and four MB samples, using the DESeq2 statistical framework.
10. **Functional enrichment:** Use g:Profiler for Gene Ontology enrichment and use pathway resources such as KEGG or Reactome where these analyses were performed and their input requirements were met.
11. **Visualization:** Use IGV to inspect selected genomic loci in the context of coverage tracks and gene annotations. Use notebooks to explore and plot analysis outputs.

## Software and resources

The recorded computational environment and tools include:

| Tool or resource | Version / purpose |
|---|---|
| SRA Toolkit | 3.2.1 |
| FastQC | 0.12.1 |
| BWA-MEM | 0.7.18-r1243 |
| SAMtools | 1.21 |
| MACS2 | 2.2.9.1 |
| BEDTools | 2.31.1 |
| HOMER | Genomic annotation and motif enrichment |
| DiffBind | 3.20.0 |
| DESeq2 | 1.50.2 |
| g:Profiler | Functional enrichment |
| IGV | 2.17.4 |
| Reference genome | GRCh38/hg38 |
| Cell Ranger ATAC | Separate 10x Genomics pipeline/reference bundle; install only if required by the processing workflow |

Versions should be checked in the environment used to reproduce the analysis. Some tools may require separate installation and configuration.

## Installation guide (Ubuntu/Linux)

The commands below are a practical starting point for setting up the analysis environment. Run them in a terminal. Software download pages and available builds can change, so check the official documentation if a download URL or package name has changed. Do not install every tool into the system Python; use a dedicated Conda environment where possible.

### 1. Update Ubuntu and install basic utilities

```bash
sudo apt update
sudo apt install -y wget curl git unzip tar gzip bzip2 make gcc g++ \
  openjdk-17-jre-headless
```

Check that the basic tools are available:

```bash
git --version
wget --version
java -version
```

### 2. Install Conda or Mamba

Install Miniforge (which provides Conda/Mamba) from the official project instructions:

https://github.com/conda-forge/miniforge

After installation, open a new terminal and check:

```bash
conda --version
```

Create an environment for the command-line ATAC-seq tools:

```bash
conda create -n atac_tools -c conda-forge -c bioconda \
  fastqc bwa samtools macs2 bedtools homer sra-tools multiqc -y
conda activate atac_tools
```

Check the installed versions:

```bash
fastqc --version
bwa 2>&1 | head
samtools --version | head -n 1
macs2 --version
bedtools --version
fastq-dump --version
```

Conda repositories may not provide the exact historical version used in this project on every platform. If exact reproducibility is required, pin package versions explicitly and record the environment with:

```bash
conda list
conda env export --no-builds > environment.yml
```

### 3. Install HOMER

HOMER may be available through Bioconda in the environment above. Check that its commands are available:

```bash
annotatePeaks.pl 2>&1 | head
findMotifsGenome.pl 2>&1 | head
```

If HOMER was not installed by Conda, follow the official HOMER installation instructions:

http://homer.ucsd.edu/homer/

HOMER also needs the appropriate genome annotation/reference for the commands being used. Follow HOMER's documented genome setup process if you intend to use HOMER's genome-specific annotation functions.

### 4. Install R, DiffBind, and DESeq2

Create a separate R environment so the Bioconductor packages do not conflict with the command-line tools:

```bash
conda create -n atac_r -c conda-forge -c bioconda \
  r-base r-essentials bioconductor-diffbind bioconductor-deseq2 -y
conda activate atac_r
```

Open R:

```bash
R
```

Then check the package versions:

```r
R.version.string
packageVersion("DiffBind")
packageVersion("DESeq2")
```

If the required historical versions are not available for your platform through Conda, use the official Bioconductor installation instructions and select package versions compatible with your R version:

- DiffBind: https://bioconductor.org/packages/DiffBind/
- DESeq2: https://bioconductor.org/packages/DESeq2/

Exit R with `q()` when finished.

### 5. Install g:Profiler access for Python

The project used g:Profiler for functional enrichment. If a notebook uses the Python client, install it in the Python environment used by that notebook:

```bash
conda create -n atac_python -c conda-forge python=3.11 jupyterlab \
  pandas numpy matplotlib seaborn gprofiler-official -y
conda activate atac_python
```

Start JupyterLab with:

```bash
jupyter lab
```

If your notebook uses `gseapy` rather than `gprofiler-official`, install it in the same environment:

```bash
python -m pip install gseapy
```

For online g:Profiler use, an internet connection may be required. Keep the environment consistent with the package imports in each notebook.

### 6. Install IGV

IGV can be installed using the official download and setup instructions:

https://igv.org/doc/desktop/

The Ubuntu package/installer can vary by release. After installation, open IGV and select the human GRCh38/hg38 genome, or load the matching reference and annotation files. Make sure the BAM files and their index files (`.bai`) are in the same accessible location.

### 7. Install Cell Ranger ATAC and obtain the reference files

**Important distinction:** The project workflow described in this repository uses BWA-MEM, SAMtools, and MACS2 for alignment and peak calling. Cell Ranger ATAC is a separate 10x Genomics pipeline. Its reference bundle is not automatically required for a BWA-MEM/MACS2 workflow unless your specific processing steps use that bundle or the corresponding reference files. Do not mix reference builds; use GRCh38/hg38 consistently.

If Cell Ranger ATAC was used for processing or you need its GRCh38 reference bundle:

1. Read the official 10x Genomics installation instructions for Cell Ranger ATAC and check the supported Linux requirements: https://www.10xgenomics.com/support/software/cell-ranger-atac
2. Download the appropriate Linux release from the official 10x Genomics download page. You may need to sign in or accept the software terms.
3. In a terminal, move to the directory where the downloaded archive is located. The exact archive filename depends on the version downloaded. For an archive named `cellranger-atac-<version>.tar.gz`, extract it with:

   ```bash
   tar -xzvf cellranger-atac-<version>.tar.gz
   ```

4. Add the extracted directory's `cellranger-atac` executable to your `PATH`, or run it using its full path. For example, if extracted under your home directory:

   ```bash
   echo 'export PATH="$HOME/cellranger-atac-<version>:$PATH"' >> ~/.bashrc
   source ~/.bashrc
   cellranger-atac --version
   ```

   Replace `<version>` with the actual directory name. Do not paste the angle-bracket placeholder literally.

5. Download the **GRCh38 reference package specifically intended for the Cell Ranger ATAC release you installed** from the official 10x Genomics reference-data/download page. Reference bundle names and compatible versions can change; select the human GRCh38 bundle listed for Cell Ranger ATAC, not a mouse reference or a different genome build.
6. Extract the downloaded reference archive. For a `.tar.gz` file, use:

   ```bash
   tar -xzvf refdata-cellranger-atac-GRCh38-<version>.tar.gz
   ```

   Replace the placeholder with the actual downloaded filename. Keep the extracted reference directory intact and note its full path.

7. Confirm the extracted reference directory exists and inspect its contents:

   ```bash
   ls -lah
   find refdata-cellranger-atac-GRCh38-* -maxdepth 2 -type f | head
   ```

   Use the actual extracted directory name in the command. Do not rename individual files inside the reference bundle.

**Reference compatibility note:** Cell Ranger ATAC reference packages are built for the Cell Ranger ATAC pipeline and a particular genome annotation. They are not interchangeable with every generic GRCh38 FASTA/index. For BWA-MEM, use a compatible FASTA and BWA index and record the source and build. If you use a Cell Ranger ATAC reference bundle only to obtain genome files, document that explicitly; do not state that Cell Ranger ATAC performed alignment unless you actually ran that pipeline.

### 8. Download the GRCh38 reference for BWA-MEM

For the BWA-MEM steps, you need a GRCh38 FASTA file and a BWA index. Prefer the exact reference FASTA used in the original analysis, and document its source/version. If using a FASTA from a Cell Ranger ATAC reference bundle, verify that it is the intended human GRCh38 sequence and use it consistently.

Example commands, after you have placed the chosen FASTA at `references/GRCh38.fa`:

```bash
mkdir -p references
# Place the verified GRCh38 FASTA at references/GRCh38.fa before continuing.
bwa index references/GRCh38.fa
samtools faidx references/GRCh38.fa
```

The `bwa index` command creates BWA index files; `samtools faidx` creates a FASTA index. These commands do not download the genome. Download reference files only from an authoritative source and comply with its terms.

### 9. Verify the main tools

Activate the environment in which each tool was installed, then run the relevant version commands:

```bash
fastqc --version
bwa 2>&1 | head
samtools --version | head -n 1
macs2 --version
bedtools --version
cellranger-atac --version
```

The `cellranger-atac` check applies only if you installed Cell Ranger ATAC and configured its executable path. HOMER, R packages, and IGV should be checked using their own commands or application interface.

### 10. Record your environment for collaborators

After checking the versions actually used, save environment information in the repository. For example:

```bash
conda env export --no-builds > environment.yml
conda list > software_versions.txt
```

Review these files before committing them. They should reflect the environments you actually used, not a newly created environment that has not been tested with the notebooks.

## Repository contents

The project notebooks include:

- `Quality_assessment.ipynb` — quality-control summaries and plots.
- `Genomic_distribution_region_plot_analysis.ipynb` — genomic distribution summaries and visualizations.
- `Differential Accessibility_plot_analysis.ipynb` — exploration and plotting of differential accessibility results.
- `Differential_accessibility_region_and motif_enrichment.ipynb` — analysis notes and workflow steps related to differential/shared regions and motif analysis.
- `Motif_enrichment_and_Functional_enrichment_plot_analysis.ipynb` — motif and functional enrichment result exploration and plots.

These filenames reflect the notebooks currently prepared for the project. For a more portable public repository, consider renaming them with numbered prefixes and underscores, for example:

```text
notebooks/
├── 01_quality_assessment.ipynb
├── 02_genomic_distribution.ipynb
├── 03_differential_accessibility.ipynb
├── 04_shared_regions_and_motifs.ipynb
└── 05_functional_enrichment.ipynb
```

A suggested overall structure is:

```text
.
├── README.md
├── CITATION.cff
├── .gitignore
├── notebooks/
├── scripts/
├── results/
│   ├── figures/
│   └── tables/
└── docs/
    └── methods.md
```

This is a recommended structure, not a claim that all these folders already exist. Add only files that are actually included in the repository.

## Reproducing the analysis

1. Clone or download the repository.
2. Install the command-line tools and Python/R packages required by the notebooks.
3. Download the public SRA datasets and obtain the matching GRCh38/hg38 reference files.
4. Run the read-processing, alignment, and peak-calling steps to generate the required intermediate files.
5. Update file paths in the notebooks for your own system.
6. Run the notebooks in workflow order and verify that each expected input file exists.

### Reproducibility notes

- Some notebooks may depend on intermediate files generated outside the notebooks.
- Local paths such as `/home/ankita/chromatin_results/` are specific to the original working environment and should be replaced with relative paths or configurable project paths.
- Raw sequencing files, large BAM files, and reference-genome files should generally not be committed directly to GitHub. Link to the public accessions and document how required files can be obtained or generated.
- Before claiming that the entire workflow is reproducible, test it from a clean environment and document all dependencies and commands.

## Interpreting the results

ATAC-seq measures chromatin accessibility rather than gene expression directly. Differential regions should therefore be described as **more accessible** or **less accessible**, not as upregulated or downregulated genes.

Motif enrichment identifies sequence patterns that may be recognized by transcription factors, but a motif alone does not prove that a transcription factor is expressed, bound, or active. Functional enrichment identifies over-represented annotations among the genes supplied to the analysis; it does not directly demonstrate pathway activation.

The comparison is based on the selected public datasets. Differences may reflect study design, sample composition, library preparation, sequencing depth, or other technical and biological factors. Interpret any cell-cycle or developmental-signaling association in light of the underlying data, statistical results, and limitations.

## Collaboration and contributions

This repository documents a bioinformatics research project comparing chromatin accessibility in Alzheimer's disease (AD) and medulloblastoma (MB).

- **Research Guide:** [Vaibhavi Jain](https://github.com/VaibuJain) — Research guidance, methodological supervision, and scientific interpretation.
- **Project Researcher:** Ankita Bhosale — Project implementation, data analysis, and repository maintenance.
- **Project Researcher;** [Abhishek Nagargoje](https://github.com/abhisheknagargoje-web) - Drafting, Review

Contributions and research roles are acknowledged according to the work performed by each team member.

## References

- Andrews, S. (2010). *FastQC: A quality control tool for high throughput sequence data.*
- Ashburner, M., et al. (2000). Gene Ontology: Tool for the unification of biology. *Nature Genetics, 25*, 25–29.
- Feng, J., Liu, T., Qin, B., Zhang, Y., & Liu, X. S. (2012). Identifying ChIP-seq enrichment using MACS. *Nature Protocols, 7*, 1728–1740.
- Heinz, S., et al. (2010). Simple combinations of lineage-determining transcription factors prime cis-regulatory elements required for macrophage and B cell identities. *Molecular Cell, 38*(4), 576–589.
- Jassal, B., et al. (2020). The Reactome pathway knowledgebase. *Nucleic Acids Research, 48*(D1), D498–D503.
- Kanehisa, M., & Goto, S. (2000). KEGG: Kyoto Encyclopedia of Genes and Genomes. *Nucleic Acids Research, 28*(1), 27–30.
- Leinonen, R., Sugawara, H., & Shumway, M. (2011). The Sequence Read Archive. *Nucleic Acids Research, 39*(Database issue), D19–D21.
- Li, H., & Durbin, R. (2009). Fast and accurate short read alignment with Burrows–Wheeler transform. *Bioinformatics, 25*(14), 1754–1760.
- Li, H., et al. (2009). The Sequence Alignment/Map format and SAMtools. *Bioinformatics, 25*(16), 2078–2079.
- Love, M. I., Huber, W., & Anders, S. (2014). Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2. *Genome Biology, 15*, 550.
- Quinlan, A. R., & Hall, I. M. (2010). BEDTools: A flexible suite of utilities for comparing genomic features. *Bioinformatics, 26*(6), 841–842.
- Raudvere, U., et al. (2019). g:Profiler: A web server for functional enrichment analysis and conversions of gene lists (2019 update). *Nucleic Acids Research, 47*(W1), W191–W198.
- Robinson, J. T., et al. (2011). Integrative genomics viewer. *Nature Biotechnology, 29*, 24–26.
- Schneider, V. A., et al. (2017). Evaluation of GRCh38 and de novo haploid genome assemblies demonstrates the enduring quality of the reference assembly. *Genome Research, 27*(5), 849–864.
- Stark, R., & Brown, G. (2011). *DiffBind: Differential binding analysis of ChIP-Seq peak data.*
- Zhang, Y., et al. (2008). Model-based analysis of ChIP-Seq (MACS). *Genome Biology, 9*, R137.

Please verify complete reference details and add DOIs or URLs as required by your target journal.

## License

Choose a license before inviting external contributions or encouraging reuse. An open-source license such as MIT may suit code if you have the right to license it, but it does not automatically license third-party datasets, figures, or manuscript text. Confirm the terms for all included material before adding a license.
