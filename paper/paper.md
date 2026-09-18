---
title: 'Predicting microbial energy metabolism from genomes through knowledge graphs and machine learning'
title_short: ''
tags:
  - Semantic web
  - Ontologies
  - Microbial energy metabolism
  - Workflows
authors:
  - name: Shuichi Kawashima
    orcid: 0000-0001-7883-3756
    affiliation: 1
    role: Writing – original draft
  - name: Danil Ezhov
    orcid: 0000-0000-0000-0000
    affiliation: 2
    role: Conceptualization, Writing – review & editing
  - name: Akira R Kinjo
    orcid: 0000-0002-4006-8208
    affiliation: 3
    role: Conceptualization, Writing – review & editing
  - name: Julia Koblitz
    orcid: 0000-0002-7260-2129
    affiliation: 4
    role: Conceptualization, Writing – review & editing
  - name: Takeru Nakazato
    orcid: 0000-0002-0706-2867
    affiliation: 5
    role: Conceptualization, Writing – review & editing
  - name: Yoko Okabbepu
    orcid: 0000-0000-0000-0000
    affiliation: 6
    role: Conceptualization, Writing – review & editing
  - name: Risa Otsuka
    orcid: 0000-0000-0000-0000
    affiliation: 5
    role: Conceptualization, Writing – review & editing
affiliations:
  - name: Database Division for Life Science, BioData Science Initiative, National Institute of Genetics, Research Organization of Information and Systems, Chiba, Japan
    index: 1
  - name: ITMO University, Russia
    ror: 044rwnt51
    index: 2
  - name: Anima Machina G.K., Osaka, Japan
    ror: 044rwnt51
    index: 3
  - name: OSIRIS Solutions GmbH, Germany
    ror: 044rwnt51
    index: 4
  - name: Biological Resource Center, National Institute of Technology and Evaluation, Japan
    ror: 044rwnt51
    index: 5
  - name: OKBP Inc., Japan
    ror: 044rwnt51
    index: 6
date: 18 September 2026
cito-bibliography: paper.bib
event: BH26JP
biohackathon_name: "DBCLS BioHackathon 2026"
biohackathon_url:   "https://2026.biohackathon.org/"
biohackathon_location: "Matsuyama, Japan, 2026"
group: microbial-metabolic-traits
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-japan/BH26-microbial-metabolic-traits
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: First Author \emph{et al.}
---


# Abstract

The growing availability of microbial genomes creates opportunities to predict physiological traits and prioritize cultivation experiments. During DBCLS BioHackathon 2026, we connected genome-derived protein annotations to electron donor and acceptor information, initially focusing on autotrophic microorganisms. KofamScan annotations for 23,434 RefSeq genomes and KEGG Orthology (KO) definitions were represented in RDF and exposed through a SPARQL endpoint accessible to TogoMCP. Literature-assisted phenotype collection and an interactive viewer linked reported physiology to genomic evidence. Within a focused collection of 586 genomes, 358 labeled genomes supported evaluation of 14 metabolic categories. A KO-based classifier achieved macro area under the precision-recall curve (AUPRC) of 0.576 when entire genera were held out, compared with 0.472 for a taxonomic baseline. Combining KO and protein language model representations improved internal performance to 0.625, but the improvement did not transfer to external evaluation. On 562 genomes from unseen genera, the KO model achieved 0.511 versus 0.380 for the taxonomic baseline. Completeness scores for 71 KEGG modules provided complementary pathway annotations. Variation among traits and disagreement between phenotype compilations identified priorities for further curation. These resources establish a basis for genome-guided screening, while experimental validation remains necessary before predictions can guide cultivation.

# Introduction


Metagenomic sequencing and the reconstruction of metagenome-assembled genomes (MAGs) have expanded access to microbial diversity beyond cultured organisms (Nayfach et al., 2021). Predicting physiological traits from these genomes could improve screening and help identify microorganisms with useful biochemical capabilities. AutoFixMark exemplifies this approach by using curated KO marker combinations to infer CO2 fixation pathways (Kawashima et al., 2026). Carbon fixation, however, does not identify the electron donors and acceptors through which an organism obtains energy and reducing power.
Predicting these substrates could inform the choice of medium components and cultivation atmospheres. We initially focused on autotrophs, particularly chemolithoautotrophs, because existing carbon fixation resources offer a practical starting point and because separating carbon assimilation from energy acquisition helps formulate cultivation hypotheses. This focus does not imply that all autotrophs use exclusively inorganic electron donors.
At BioHackathon 2026, we combined data integration, literature-assisted curation, visualization, and predictive modeling around this question. We developed a knowledge graph that lets AI assistants retrieve genomic evidence and a benchmark that evaluates prediction beyond labeled training lineages. Together, these activities connect what microorganisms are reported to use with what their genomes encode.

# Methods

## Genomic and physiological evidence

We used KofamScan annotations for 23,434 RefSeq reference genomes and a focused collection of 586 autotrophic microbial genomes. KofamScan assigns KEGG Orthology (KO) identifiers to proteins using profile hidden Markov models and profile-specific thresholds (Aramaki et al., 2020). Assembly accessions linked genomes, protein annotations, and phenotype records.
Protein-to-KO assignments were converted to Resource Description Framework (RDF), retaining scores, thresholds, E-values, and significance flags. KofamScan definitions were also represented in RDF using shared KO identifiers and skos:definition, enabling direct retrieval of annotated functions and their descriptions. The data were loaded into QLever and exposed at the experimental endpoint https://rdfportal.org/microbes/sparql. Its registration in TogoMCP made these annotations available to AI assistants alongside other RDF Portal resources (TogoMCP contributors, accessed 2026) (DBCLS, accessed 2026).
In parallel, workflows assisted by large language models (LLMs) extracted donor and acceptor information from articles and databases, including BacDive (Leibniz Institute DSMZ, accessed 2026). We merged complementary extraction efforts and recorded supporting text, conflicts, and evidence grades. Grades distinguished experimentally supported statements found by multiple or single extraction efforts from weaker, inferred, or disputed evidence; agreement between extraction efforts was not treated as independent experimental replication. A 16 September snapshot contained 1,817 evidence records. The prediction benchmark contained 358 labeled genomes and 14 categories, covering seven donor-related and seven acceptor-related traits; 228 genomes lacked these benchmark labels.
We developed a viewer connecting reported substrates and evidence grades to genomic markers. Reference tables linked metabolic transformations to relevant proteins and KOs. Jaccard similarity between KO sets supported comparative displays of functional profiles. These displays facilitate inspection of disagreements between phenotype records and encoded functions.

## Predictive models and pathway annotations

The KO model, B2, used binary presence or absence features from an 852-KO energy-metabolism panel; 585 KOs varied across the 358 labeled genomes in the final training set. For each trait, inner cross-validation selected L2-regularized logistic regression or an XGBoost configuration. B2XL averaged logistic-regression and XGBoost predictions on the same KO features. The command-line workflow accepted RefSeq accessions or nucleotide FASTA files, using NCBI protein annotations or Pyrodigal gene predictions, respectively. Its KOfam search used pyhmmer with profile-specific thresholds.
We compared KO features with pooled ESM-2 35M (Lin et al., 2023) and ESM-C 300M representations (ESM contributors, accessed 2026), attention-based multiple-instance learning over ESM-2 proteins, and Bacformer genome representations (Bacformer contributors, accessed 2026). The strongest internal fusion combined B2XL with Bacformer-large: ESM-C-derived protein vectors were processed in genomic coordinate order, pooled into a genome vector, and classified by logistic regression. The KO and sequence-branch probabilities were averaged with equal weights. Supervised genomic models did not use organism names, taxonomy, or literature as input.
Because the reference labels described organism-level traits, we assessed pathways separately by parsing and scoring completeness for 71 KEGG modules spanning energy metabolism, carbon fixation, methane, nitrogen, and sulfur metabolism (KEGG, accessed 2026). These deterministic annotations complement the trait classifiers; they are not predictions from a separately trained pathway model.

## Evaluation

Cross-validation held out complete genera and, separately, complete families. Comparators included curated marker rules and a taxonomic baseline that transferred label frequencies from training relatives. The primary metric was macro area under the precision-recall curve (AUPRC), giving equal weight to all 14 traits. Binary thresholds maximized Matthews correlation coefficient (MCC) within inner cross-validation. Uncertainty was estimated by resampling whole genera or families; comparisons across 28 model variants used max-T simultaneous inference.
External evaluation used 562 genomes from genera absent from supervised training, with phenotype annotations derived from Madin and colleagues (Madin et al., 2020). A separate LLM comparison supplied significant KofamScan hits for 96 genomes, either with organism names withheld or with names supplied. The named GPT condition returned complete answers for 84 genomes, and its paired comparisons used that subset. This distinguishes inference from gene evidence from answers that may incorporate known physiology.

# Results

## Integrated evidence and pathway analysis

The shared resource linked assembly identifiers to proteins, KO assignments, and functional definitions. Through TogoMCP, an assistant could incorporate these annotations into answers about electron donors and acceptors. The viewer presented genomic markers alongside literature-derived traits, supporting review of both concordant and conflicting evidence. This established an evidence-retrieval workflow; deployment of the trained classifier as an MCP prediction tool remained a separate step.
Module scoring identified a median of six complete modules per genome. Mean completeness was higher in genomes labeled positive than negative for corresponding traits: 0.81 versus 0.15 for dissimilatory sulfate reduction, 0.60 versus 0.09 for SOX thiosulfate oxidation, and 0.46 versus 0.11 for denitrification. Module annotations also distinguished transformations grouped within broader phenotype labels, including denitrification and dissimilatory nitrate reduction to ammonium. Completeness describes encoded potential rather than demonstrated pathway activity.


## Prediction beyond training lineages

B2 achieved macro AUPRC of 0.576 under genus-held-out evaluation and 0.560 under family-held-out evaluation, exceeding the taxonomic baseline by 0.104 and 0.180, respectively (Table 1). Bacformer-large alone performed similarly to B2 at genus level. The highest internal point estimates came from fusion models, reaching 0.625 and 0.585 for B2XL plus Bacformer-large. Two fusion configurations retained an advantage over B2 after correction across model variants, but their advantage over the KO-only ensemble B2XL was not statistically resolved.


```markdown
Table: Selected model comparisons. Values are macro AUPRC under taxonomically grouped cross-validation. The attention model shown here is the completed rerun without a KO-based protein prefilter.

| Model | Held-out genus | Held-out family |
| -------- | -------- | -------- |
| Curated marker rules | 0.317 | 0.317 |
| Taxonomic baseline | 0.472 | 0.380 |
| Pooled ESM-2 | 0.544 | 0.497 |
| Attention MIL without KO prefilter | 0.548 | 0.495 |
| Bacformer-large | 0.577 | 0.531 |
| B2 KO classifier | 0.576 | 0.560 |
| B2XL KO ensemble | 0.602 | 0.573 |
| B2XL plus ESM-2 | 0.616 | 0.584 |
| B2XL plus Bacformer-large | 0.625 | 0.585 |
```

The completed attention-model rerun achieved 0.548 compared with 0.544 for mean pooling, a difference of 0.004 with a 95% confidence interval of −0.024 to 0.045. It therefore provided no clear evidence that attention improved on pooling in this setting.
On the external genomes, B2 achieved 0.511 compared with 0.380 for the taxonomic baseline, a paired difference of 0.131 (95% confidence interval 0.100–0.167). Completed external evaluation of the two strongest internal fusion configurations did not support transfer of their internal gains. B2 was consequently retained as the deployment model. A random-split analysis produced 0.647 compared with 0.576 under genus-held-out evaluation, illustrating the importance of separating related genomes when estimating generalization.
Performance varied among traits. Sulfate reduction, sulfur oxidation, H2 oxidation, methanogenesis/acetogenesis, aerobic respiration, and ammonia oxidation had B2 AUPRC values of 0.817–0.887. Nitrate reduction reached 0.678, whereas Fe(III) reduction and the residual donor and acceptor categories scored 0.214, 0.095, and 0.136. Nitrite oxidation scored 0.337 with only 12 positives, emphasizing that absolute AUPRC must be interpreted alongside prevalence. Outputs for the 228 genomes without benchmark labels included scores and reliability flags, with binary calls restricted to traits showing predictive skill beyond training lineages.

## LLM comparisons and phenotype disagreement

On the 96-genome subset, anonymized gene-evidence prompts yielded AUPRC of 0.578 for Claude and 0.588 for GPT-6 Astra, compared with 0.666 for B2 and 0.697 for the strongest fusion model (Table 2). Supplying organism names raised Claude's score to 0.777. Named GPT reached 0.731 on 84 genomes; its paired difference from the strongest fusion model was 0.035, with a confidence interval including zero. Named-organism performance can benefit from previously published physiology and therefore does not establish equivalent performance for uncharacterized genomes.

```markdown
Table: LLM comparison on a 96-genome subset. The named GPT row used 84 complete responses; its paired comparisons were calculated on those same genomes. These values should not be compared directly with the full-cohort scores in Table 1.

| Configuration | Genomes | Macro AUPRC |
| -------- | -------- | -------- |
| Taxonomic baseline | 96 | 0.485  |
| Claude with anonymized gene evidence | 96 | 0.578 |
| GPT-6 Astra with anonymized gene evidence | 96 | 0.588 |
| B2 | 96 | 0.666 |
| B2XL | 96 | 0.687 |
| B2XL plus Bacformer-large | 96 | 0.697 |
| Claude with organism name | 96 | 0.777 |
| GPT-6 Astra with organism name | 84 | 0.731 |
```

Across 179 shared species, agreement between phenotype compilations was Cohen's kappa 0.61, compared with 0.52 between the model and the external compilation. Excluding H2 oxidation, for which the compilations labeled 125 versus 33 species positive, increased agreement between compilations to 0.68. These discrepancies identify label definitions and evidence coverage as priorities for curation, but do not establish a numerical ceiling on AUPRC.

# Discussion and conclusions

The hackathon connected genomic annotations, literature-derived phenotypes, comparative visualization, and predictive evaluation within a common workflow. RDF and TogoMCP make supporting evidence accessible, while supervised evaluation tests whether genomic features generalize beyond labeled lineages. Pathway completeness adds mechanistic context to broad donor and acceptor categories.
The results favor a compact KO-based deployment model. Although fusion improved internal scores, its advantage did not transfer to the external annotation set, and the internal contribution of sequence representations could not be separated from learner ensembling. The completed attention comparison likewise provided no clear improvement over mean pooling. These findings support investment in clearer phenotype definitions and targeted genomic evidence alongside further model development.
Several limitations constrain interpretation. Unreported substrates are not necessarily experimentally tested negatives, species-level records may differ from strain physiology, and different compilations may share underlying publications. Genus and family separation does not exclude overlap in foundation-model pretraining. The limited labeled cohort also makes rare traits difficult to evaluate, and performance on incomplete MAGs remains to be established. Neither KO presence nor module completeness alone demonstrates an active, energy-conserving pathway.
For cultivation, compatible donor, acceptor, and carbon fixation predictions could prioritize substrate combinations for testing. Actual medium design additionally depends on growth conditions, nutrients, regulation, and substrate concentrations. We did not experimentally validate cultivation predictions during this work. The immediate outcome is a shared computational foundation for screening and evidence review; harmonized labels, reproducible model releases, and cultivation experiments are the next steps toward practical application.

## Acknowledgements and declarations

We thank the BioHackathon 2026 organizers and participants and the maintainers of the resources used here. [TO CONFIRM: author contributions, funding, and competing interests.] LLMs assisted evidence extraction, trait prediction experiments, and manuscript drafting; authors must review the final text and underlying evidence.

## Figures

A figure is added with:

```markdown
![Caption for BioHackrXiv logo figure](./biohackrxiv.png)
```

This gives:

![Caption for BioHackrXiv logo figure \label{figureCode}](./biohackrxiv.png)

Figures can be scaled by adding the width or height to the Markdown like this:

```markdown
![Caption for BioHackrXiv logo figure](./biohackrxiv.png){ width=50px }
```

You can add cross references to figures by adding a LaTeX `\label{figureCode}` to
the label of the Markdown figure and then use `\ref{figureCode}` to cite it:

```markdown
![Caption for BioHackrXiv logo figure \label{figureCode}](./biohackrxiv.png){ width=50px }
```

This way, we can cite Figure \ref{figureCode}.


# Citation Typing Ontology annotation

You can use [CiTO](http://purl.org/spar/cito/2018-02-12) annotations, as explained in [this BioHackathon Europe 2021 write up](https://raw.githubusercontent.com/biohackrxiv/bhxiv-metadata/main/doc/elixir_biohackathon2021/paper.md) and [this CiTO Pilot](https://www.biomedcentral.com/collections/cito).
Using this template, you can cite an article and indicate _why_ you cite that article, for instance DisGeNET-RDF [@citesAsAuthority:Queralt2016].

The syntax in Markdown is as follows: a single intention annotation looks like
`[@usesMethodIn:Krewinkel2017]`; two or more intentions are separated
with colons, like `[@extends:discusses:Nielsen2017Scholia]`. When you cite two
different articles, you use this syntax: `[@citesAsDataSource:Ammar2022ETL; @citesAsDataSource:Arend2022BioHackEU22]`.

Possible CiTO typing annotation include:

* citesAsDataSource: when you point the reader to a source of data which may explain a claim
* usesDataFrom: when you reuse somehow (and elaborate on) the data in the cited entity
* usesMethodIn
* citesAsAuthority
* citesAsEvidence
* citesAsPotentialSolution
* citesAsRecommendedReading
* citesAsRelated
* citesAsSourceDocument
* citesForInformation
* confirms
* documents
* providesDataFor
* obtainsSupportFrom
* discusses
* extends
* agreesWith
* disagreesWith
* updates

There is a general `cites` intention, but this is already implied and should be left out.

...

# References

```{=latex}
\AtEndDocument{%
```

# Appendices

If you want the Appendix (-ces) to show up after the references, wrap them in 
after the header, like done in this Markdown file. Look at the [source](paper.md)
to see the exact structure.

```{=latex}
}
```
