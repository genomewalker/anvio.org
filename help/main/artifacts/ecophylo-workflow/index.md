---
layout: artifact
title: ecophylo-workflow
excerpt: A WORKFLOW-type anvi'o artifact. This artifact is typically generated, used, and/or exported by anvi'o (and not provided by the user)..
categories: [anvio]
comments: false
redirect_from: /m/ecophylo-workflow
image:
  featurerelative: ../../../images/header.png
  display: true
---


{% include _toc.html %}


<img src="../../images/icons/WORKFLOW.png" alt="WORKFLOW" style="width:100px; border:none" />

A WORKFLOW-type anvi'o artifact. This artifact is typically generated, used, and/or exported **by anvi'o** (and not provided by the user)..

🔙 **[To the main page](../../)** of anvi'o programs and artifacts.

## Provided by


<p style="text-align: left" markdown="1"><span class="artifact-p">[anvi-run-workflow](../../programs/anvi-run-workflow)</span></p>


## Required or used by


There are no anvi'o tools that use or require this artifact directly, which means it is most likely an end-product for the user.


## Description

The <span class="artifact-n">[ecophylo-workflow](/help/main/artifacts/ecophylo-workflow)</span> explores the ECOlogical and PHYlogenetic relationships of proteins across genomes and metagenomes. It is a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow run by the anvi'o script <span class="artifact-p">[anvi-run-workflow](/help/main/programs/anvi-run-workflow)</span>. Briefly, the workflow extracts a target protein from any number of assemblies in <span class="artifact-n">[metagenomes](/help/main/artifacts/metagenomes)</span> and/or <span class="artifact-n">[external-genomes](/help/main/artifacts/external-genomes)</span> using a user-designated [HMM](https://anvio.org/vocabulary/#hidden-markov-models-hmms) from <span class="artifact-n">[hmm-list](/help/main/artifacts/hmm-list)</span>, then clusters the sequences and selects representatives from each cluster. Next, the workflow post-processes the representative sequences using phylogenetics and metagenomic read recruitment to produce an anvi'o <span class="artifact-n">[interactive](/help/main/artifacts/interactive)</span> interface to explore their phylogenetic distances and co-occurrence across metagenomes (via simultaneous visual of a phylogenetic tree and read recruitment results). The workflow can use any protein-based [HMM](https://anvio.org/vocabulary/#hidden-markov-models-hmms) including [single-copy core genes](https://anvio.org/vocabulary/#single-copy-core-gene-scg) to taxonomically profile metagenomes or any functional protein to explore variants across samples. 

The <span class="artifact-n">[ecophylo-workflow](/help/main/artifacts/ecophylo-workflow)</span> has 2 modes which can be designated in the <span class="artifact-n">[workflow-config](/help/main/artifacts/workflow-config)</span> by changing the input files that are provided: `tree-mode` and `profile-mode`. In `tree-mode`, the sequences will be used to calculate a phylogenetic tree. In `profile-mode`, the sequences will be used to calculate a phylogenetic tree but also profiled via read recruitment across user-provided metagenomes. 

## Required input

The <span class="artifact-n">[ecophylo-workflow](/help/main/artifacts/ecophylo-workflow)</span> requires the following files:

- <span class="artifact-n">[workflow-config](/help/main/artifacts/workflow-config)</span>: This allows you to customize the workflow step by step. Here is how you can generate the default version:

<div class="codeblock" markdown="1">
anvi&#45;run&#45;workflow &#45;w ecophylo &#45;&#45;get&#45;default&#45;config config.json
</div>

{:.notice}
Here is a tutorial walking through more details regarding the EcoPhylo <span class="artifact-n">[workflow-config](/help/main/artifacts/workflow-config)</span> file: coming soon!

- <span class="artifact-n">[hmm-list](/help/main/artifacts/hmm-list)</span>: This file designates which HMM should be used to extract the target protein from your <span class="artifact-n">[contigs-db](/help/main/artifacts/contigs-db)</span>.  
- <span class="artifact-n">[metagenomes](/help/main/artifacts/metagenomes)</span> and/or <span class="artifact-n">[external-genomes](/help/main/artifacts/external-genomes)</span>: These files hold the assemblies where you are looking for the target protein. Genomes in <span class="artifact-n">[external-genomes](/help/main/artifacts/external-genomes)</span> can be reference genomes, [SAGs](https://anvio.org/vocabulary/#single-amplified-genome-sag), and/or [MAGs](https://anvio.org/vocabulary/#metagenome-assembled-genome-mag). 

## Want to explore phylogenetic relationships of proteins across assemblies? Tree-mode

This is the simplest implementation of EcoPhylo. The workflow will extract the target protein from input assemblies, cluster the sequences and pick representatives, then calculate a phylogenetic tree based on the amino acid version of the representative sequences. There are two sub-modes of `tree-mode` depending on how you pick representative sequences: `NT-mode` or `AA-mode`


### NT-mode

This is the default version of `tree-mode`. Target protein sequences are clustered based on the nucleotide sequences of the proteins. This is done to prepare for `profile-mode` where there needs to be adequate NT sequence distance between proteins to prevent [non-specific-read-recruitment](https://anvio.org/vocabulary/#non-specific-read-recruitment). 

Here is what the start of the EcoPhylo <span class="artifact-n">[workflow-config](/help/main/artifacts/workflow-config)</span> should look like if you want to run `tree-mode`:

```bash
{
    "metagenomes": "metagenomes.txt",
    "external_genomes": "external-genomes.txt",
    "hmm_list": "hmm_list.txt",
    "samples_txt": ""
}
```

### AA-mode

This is a sub-version of `tree-mode` where sequences that are used to calculate the tree are subsetted from amino acid cluster representatives rather than nucleotide clusters. If you are only interested in protein phylogenetics, this is the way to go. 

To initialize `AA-mode`, go to the rule `cluster_X_percent_sim_mmseqs` in the EcoPhylo <span class="artifact-n">[workflow-config](/help/main/artifacts/workflow-config)</span> and turn "AA_mode" to true:

```bash
{
    "cluster_X_percent_sim_mmseqs": {
        "threads": 5,
        "--min-seq-id": 0.94,
        "clustering_threshold_for_OTUs": [
            0.99
        ],
        "AA_mode": True
    }
}
```

## Want to track proteins across metagenomic samples via read recruitment? Profile-mode

`profile-mode` is an extension of default `tree-mode` where NT sequences representatives are profiled with metagenomic reads from user provided metagenomic samples. This allows for the simultaneous visualization of phylogenetic and ecological relationships of proteins across metagenomic datasets. 

Additional required files:
- <span class="artifact-n">[samples-txt](/help/main/artifacts/samples-txt)</span>

To initialize `profile-mode`, add the path to your <span class="artifact-n">[samples-txt](/help/main/artifacts/samples-txt)</span> to your EcoPhylo <span class="artifact-n">[workflow-config](/help/main/artifacts/workflow-config)</span>:

```bash
{
    "metagenomes": "metagenomes.txt",
    "external_genomes": "external-genomes.txt",
    "hmm_list": "hmm_list.txt",
    "samples_txt": "samples.txt"
}
```

{:.notice}
Edit [this file](https://github.com/merenlab/anvio/tree/master/anvio/docs/artifacts/ecophylo-workflow.md) to update this information.

