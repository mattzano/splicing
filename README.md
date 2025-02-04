# splicing
Splicing done with MAJIQ tool
The purpose of this pipeline is to be able to run MAJIQ using Snakemake. The aim is to make MAJIQ easier to run for non-bioinformaticians and it produces additional parsing and annotation to the MAJIQ output.

# Needed files
1. Aligned, sorted, and indexed BAM files of RNA-seq for all your samples.
 - .bam 
 - .bam.bai files 
2. GFF3 and GTF of your species of interest - already provided with the config file
3. A formatted sample sheet, see examples and explanation below
# Get started
## Necessary R packages
#If you're just going to run the build + psi workflows you will need
#data.table
#tidyverse
#optparse
#glue
No worries about this because there is an environment provided with the necessary packages

You will need to do 3 things to get this pipeline going:
1. Set up a sample sheet
2. Edit the config/comparisons.yaml
3. Edit the config/config.yaml


## 1.Making a sample sheet - the same used for alignment should work

Remember, the following columns are mandatory:
 - sample_name,
 - group, 
 - exclude_sample_downstream_analysis # this should be present, if you want to exclude a sample it should be a 1, otherwise you can leave it blank
 - After these 3 critical columns, you can include as many additional columns as you like
*Please* use syntactic names for `sample_name` and `group` (no spaces, don't start with a number, use underscores and not hyphens) that would lead to errors!!!

Here is an example sample sheet where we have a het, hom, and wt of a mutant

| sample_name | group | exclude_sample_downstream_analysis | litter |
|-------------|-------|------------------------------------|--------|
| M323K_HET_1 | het   |                                    | one    |
| M323K_HET_2 | het   |                                    | two    |
| M323K_HET_3 | het   |                                    | three  |
| M323K_HET_4 | het   |                                    | four   |
| M323K_HOM_1 | hom   |                                    | one    |
| M323K_HOM_2 | hom   |                                    | two    |
| M323K_HOM_3 | hom   |                                    | three  |
| M323K_HOM_4 | hom   |                                    | four   |
| M323K_HOM_5 | hom   |                                    | five   |
| M323K_WT_1  | wt    |                                    | one    |
| M323K_WT_2  | wt    |                                    | two    |
| M323K_WT_3  | wt    |                                    | three  |

My bams are named like this:
`M323K_HET_1.Aligned.sorted.out.bam`


## Setting up your comparisons

To compare groups, we need to go int the config/comparisons.yaml and edit it. You can either compare pairwise (first and second examples) or by pooling some of the conditions together (third example)

Here's an example from the sample sheet above:

knockdownexperiment:
  column_name:
    - group
  wt:
    - wt
  hom:
    - hom
controlVersusHets:
  column_name:
    - group
  wt:
    - wt
  het:
    - het
litterComparison:
  column_name:
    - litter
  firstLitters:
    - one
    - two
  secondLitters:
    - three
    - four
    - five


## Making the config
In the config/config.yaml file you only have to change/check lines:
- 2, species
- 3, read_len
- 4, genome_refcode
- 5, endtype (pe or se)
- 6, sampleCSVpath!!!!
- 10, project_top_level
- 15, run_name
- 18, bam_dir!!!!
- 28 bam_suffix


# Final outputs
Underneath the folder in

`majiq_top_level: /SAN/vyplab/alb_projects/data/linked_bams_f210i_brain/majiq/`
```
majiq
├── builder
│   ├── wt_sample1.majiq
│   ├── wt_sample1.sj
│   ├── wt_sample2.majiq
│   ├── wt_sample2.sj
│   ├── mut_sample1.majiq
│   ├── mut_sample1.sj
│   ├── mut_sample2.majiq
│   ├── mut_sample2.sj
│   ├── majiq.log
│   └── splicegraph.sql
├── delta_psi
│   ├── wt_mut.deltapsi.tsv
│   ├── wt_mut.deltapsi.voila
│   └── deltapsi_majiq.log
├── delta_psi_voila_tsv
│   ├── wt_mut.junctions.bed
│   ├── wt_mut.csv
│   ├── wt_mut.gff3
│   ├── wt_mut_parsed_psi.tsv
│   └── wt_mut.psi.tsv
├── run_name_majiqConfig.tsv
├── psi_single
│   ├── wt_sample1.tsv
│   ├── wt_sample1.voila
│   ├── wt_sample2.tsv
│   ├── wt_sample2.voila
│   ├── mut_sample1.tsv
│   ├── mut_sample1.voila
│   ├── mut_sample2.tsv
│   ├── mut_sample2.voila
├── psi_voila_tsv_singlehis
│   ├── wt_sample1.tsv
│   ├── wt_sample1.voila
│   ├── wt_sample2.tsv
│   ├── wt_sample2.voila
│   ├── mut_sample1.tsv
│   ├── mut_sample1.voila
│   ├── mut_sample2.tsv
│   ├── mut_sample2.voila
└── psi
    ├── wt.psi.tsv
    ├── wt.psi.voila
    ├── mut.psi.tsv
    ├── mut.psi.voila
    └── psi_majiq.log
```


## Submitting on SGE
1. Build step
`source submit.sh build run_name`
2. PSI step
`source submit.sh psi run_name`
with whatever run name you'd like

## Submitting on Slurm
1. Build step
`source submit_slurm.sh build run_name`
2. PSI step
`source submit_slurm.sh psi run_name`
with whatever run name you'd like


## Running without a cluster
If you don't have a cluster, you can run straight with snakemake
`snakemake -s workflows/build.smk`
`snakemake -s workflows/psi.smk`


## Annotation of splicing events
Annotation is done with a function grabbed directly from source code here:
https://github.com/dzhang32/dasper/

Please cite Dasper, Snakemake, and of course MAJIQ if you use this pipeline.
