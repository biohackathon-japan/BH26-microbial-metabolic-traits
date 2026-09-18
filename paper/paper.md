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

The increasing availability of microbial genomes creates opportunities to predict physiological traits and prioritize cultivation experiments. During DBCLS BioHackathon 2026, we connected genome-derived protein annotations to electron donor and acceptor information, initially focusing on autotrophic microorganisms. KofamScan annotations for 23,434 RefSeq genomes and KEGG Orthology (KO) definitions were represented in RDF and made accessible through TogoMCP. Literature-assisted phenotype collection and an interactive viewer linked reported physiology to genomic evidence. Within a focused collection of 586 genomes, 358 labeled genomes supported evaluation of 14 metabolic categories. A KO-based classifier achieved macro area under the precision-recall curve (AUPRC) of 0.576 when entire genera were held out, compared with 0.472 for a taxonomic baseline. Combining KO and protein language model representations improved internal performance to 0.625, although the contribution beyond ensembling KO-based learners was not statistically resolved. This improvement did not transfer to external evaluation. On 562 genomes from unseen genera, the KO model achieved 0.511 versus 0.380 for the taxonomic baseline. Completeness scores for 71 KEGG modules provided complementary pathway annotations. Performance varied substantially among traits, and disagreement between phenotype compilations identified priorities for further curation. These hackathon outcomes establish a shared basis for evidence retrieval, comparative visualization, and genome-guided screening, while experimental validation remains necessary before predictions can guide cultivation.


# Introduction


Metagenomic sequencing and genome reconstruction have greatly expanded access to microbial diversity, including lineages without cultured representatives. Large collections of metagenome-assembled genomes (MAGs) provide opportunities to investigate functions that would be difficult to discover through cultivation alone [@citesAsAuthority:Nayfach2021]. However, the increasing availability of sequences has not been matched by physiological characterization. Methods that connect genomes to experimentally relevant traits could help prioritize organisms for screening and identify candidates with useful biochemical capabilities.
Carbon fixation provides one example of this approach. AutoFixMark uses curated combinations of KEGG Orthology (KO) markers to infer CO2 fixation pathways from genome annotations [@citesAsAuthority:Kawashima2026]. Such resources make biological knowledge usable for interpretable screening. The ability to assimilate inorganic carbon, however, does not identify how an organism acquires the energy and reducing power needed for growth. Carbon assimilation and energy acquisition therefore require complementary descriptions.
Electron donors and acceptors are central to this second question. Their utilization describes the redox transformations through which microorganisms can conserve energy and can inform the selection of candidate medium components and cultivation atmospheres. We initially focused on autotrophic microorganisms, particularly chemolithoautotrophs, because existing carbon fixation resources offer a practical starting point and because considering carbon supply separately from electron donation helps formulate explicit cultivation hypotheses. This focus does not imply that all autotrophs use exclusively inorganic electron donors or that carbon fixation genes alone establish autotrophic growth. Facultative lifestyles and condition-dependent substrate use remain relevant.
At BioHackathon 2026, we combined data integration, literature-assisted curation, visualization, and predictive modeling around this question. The project addressed two related needs: enabling an AI assistant to retrieve and explain evidence about a named organism, and evaluating whether genome-derived features predict traits beyond labeled training lineages. The former can draw on published physiology, whereas the latter must be assessed under controlled conditions relevant to uncharacterized organisms. We report the resources developed during the hackathon, the comparison of KO and protein sequence representations, and the implications of the results for genome-guided screening.

# Methods

## Genomic and physiological evidence

The project used a broad resource of KofamScan annotations for 23,434 RefSeq reference genomes and a focused collection of 586 autotrophic microbial genomes. Assembly accessions provided the connection between genomic annotations and physiological records. The focused benchmark contained 358 genomes with electron donor and acceptor labels; the remaining 228 genomes had genomic annotations but lacked the benchmark phenotype labels.
KofamScan assigns KO identifiers to proteins using profile hidden Markov models and profile-specific score thresholds [@citesAsAuthority:Aramaki2020]. We retained protein-to-KO assignments together with scores, thresholds, E-values, and significance flags. These records preserve the supporting protein evidence behind a genome-level KO profile. They also permit analyses to select significant assignments without losing the underlying scoring information.
The command-line prediction workflow accepted RefSeq assembly accessions or nucleotide FASTA files. It used NCBI protein annotations for accessions and Pyrodigal gene predictions for unannotated nucleotide input. Its KOfam search implementation used pyhmmer and the corresponding profile-specific thresholds. This route supported application of the KO classifier beyond the genomes already present in the shared annotation resource.

## RDF representation and access through TogoMCP

We converted genome-protein-KO associations to Resource Description Framework (RDF), representing assemblies, proteins, and KOs with identifiers.org URIs. The graph retains individual protein assignments as well as assignment-level attributes. This allows retrieval of either a compact KO set for an assembly or the proteins and scores supporting a specific annotated function.
The KofamScan definition file was also converted to RDF. KO descriptions were represented with skos:definition, while profile metadata included score thresholds and score types. Because definitions and assignments use shared KO identifiers, queries can join the observed genomic evidence to a description of its function. An assistant can consequently request relevant subsets of annotations and their definitions instead of receiving an entire proteome annotation table in its prompt.
The RDF data were loaded into QLever and made queryable through SPARQL. Registering the dataset with TogoMCP enabled AI assistants to retrieve these annotations alongside other resources available through RDF Portal [@citesAsAuthority:Kinjo2026],[@citesAsAuthority:Kawashima2018]. The implemented integration supplies evidence for questions about the donors and acceptors used by a specified organism. It exposes the annotation resource; deployment of the trained trait classifier as a dedicated MCP prediction tool is a separate step.

## Literature-assisted phenotype collection

Workflows assisted by large language models (LLMs) extracted electron donor and acceptor information from research articles and microbial databases, including BacDive [@citesAsAuthority:Schober2025]. Complementary searches and extraction efforts were merged, retaining supporting text, evidence grades, and conflicting claims. Database fields describing nutrition, metabolite utilization, and oxygen tolerance supplied additional context. These fields require biological interpretation: oxygen tolerance, for example, is distinct from evidence of respiratory oxygen use.
The merged collection used four evidence grades. Grade A identified statements describing experimental evidence corroborated by at least two extraction efforts, and grade B identified such statements found by a single effort. Grade C included genomic inference, hedged or review-only statements, weak evidence, and conflicting claims. Grade D covered unsupported, inadequately documented, or disputed records. Agreement between extraction efforts does not establish independent experimental replication, and these grades do not imply that every record received expert review.
A 16 September snapshot contained 1,817 evidence records: 702 grade A, 921 grade B, 136 grade C, and 58 grade D. These are statements, not distinct organisms or independent training examples. The benchmark labels comprised 14 categories, covering seven donor-related and seven acceptor-related traits. Some categories combined related physiological descriptions, including methanogenesis and acetogenesis, while residual categories grouped other donors or acceptors. The evidence collection and benchmark label matrix therefore represent different levels of aggregation.

## Marker references and comparative visualization

We assembled reference tables linking donor and acceptor transformations to relevant genes, enzymes, and KO identifiers. A matching tool compared these markers with genome annotations, and a web viewer displayed genomic markers alongside reported substrates and evidence grades. The viewer was developed for the broad RefSeq resource and the focused autotroph collection.
Additional comparative displays used Jaccard similarity between KO sets to organize genomes by their functional profiles. Such a representation supports exploration of shared annotated capabilities; it should not be interpreted as a sequence-based phylogeny. Together, the interfaces help identify organisms for which literature-derived physiology and expected genomic markers agree, and cases requiring closer examination. Some phenotype entries and associated descriptive annotations remained preliminary at the hackathon cutoff.

## Predictive models and pathway annotations

The reference KO classifier, B2, used binary presence or absence features from an 852-KO energy-metabolism panel. Of these, 585 KOs varied across the 358 labeled genomes in the final training set. For each trait, inner cross-validation selected L2-regularized logistic regression or an XGBoost configuration. 

B2XL averaged logistic-regression and XGBoost probabilities on the same KO features, providing a control for gains attributable to combining learners without adding sequence representations. The prediction workflow reported raw and Platt-calibrated scores and trait-specific binary calls.
Sequence-based comparisons included pooled ESM-2 35M [@citesAsAuthority:Lin2023] and ESM-C 300M representations (ESM contributors, accessed 2026), attention-based multiple-instance learning over ESM-2 protein vectors, and Bacformer-base and Bacformer-large genome representations (Bacformer contributors, accessed 2026). These approaches use protein sequence information in addition to, or instead of, explicit KO assignments. A completed attention-model rerun removed the KO-based protein prefilter used in the earlier configuration, enabling a cleaner comparison with mean pooling.
The strongest internal fusion combined B2XL with Bacformer-large. ESM-C-derived protein vectors were processed in genomic coordinate order by the genome model, pooled into a genome vector, and classified by logistic regression. The KO-branch and sequence-branch probabilities were then averaged with equal weights. This fusion was evaluated using precomputed representations of RefSeq genomes; the raw-FASTA route described above applied to the KO predictor. Taxonomy, organism names, and literature were not inputs to the supervised genomic models.
Because the reference labels described organism-level traits, pathway analysis was performed separately. We parsed and scored completeness for 71 KEGG modules spanning energy metabolism, carbon fixation, methane, nitrogen, and sulfur metabolism (KEGG, accessed 2026). These deterministic annotations describe the support for module components in the KO profile. They complement the trait classifiers rather than constituting a pathway model trained on pathway-level phenotype labels.

## Evaluation

Cross-validation held out entire genera and, separately, entire families, so test groups were absent from supervised training. Comparators included curated marker rules, a prevalence-based predictor, and a taxonomic baseline that transferred label frequencies from training relatives. Grouped evaluation tests generalization beyond close training relatives more directly than a random genome split, although it does not establish absence from foundation-model pretraining data.
The primary metric was macro area under the precision-recall curve (AUPRC), giving equal weight to all 14 traits. Per-trait AUPRC was interpreted alongside the frequency of positive labels. Matthews correlation coefficient (MCC) summarized binary predictions using thresholds selected to maximize MCC within inner cross-validation. Confidence intervals resampled whole genera or families rather than individual genomes. Comparisons across 28 model variants used max-T simultaneous inference; the analysis also used Benjamini-Hochberg adjustment for per-trait tests.
External evaluation used 562 genomes from genera absent from supervised training, with phenotype annotations derived from the compilation of Madin and colleagues [@citesAsAuthority:Madin2020]. This tests transfer to an additional annotation source as well as new genera. It does not necessarily provide independent biological experiments, because phenotype compilations can draw on overlapping publications.
A separate LLM comparison supplied significant KofamScan hits from the shared RDF resource for 96 genomes. An anonymized condition withheld organism names; a named condition supplied names alongside the gene evidence. The named GPT condition returned complete answers for 84 genomes, and its paired comparisons used that same subset. This experiment distinguishes interpretation of supplied genomic annotations from answers that can additionally exploit organism identity and known physiology.

# Results

## A shared resource for genomic and physiological evidence

The hackathon produced a searchable genome-protein-KO resource, an assembly-linked phenotype collection, and interfaces connecting the two. Through TogoMCP, an assistant could retrieve an organism's annotated functions, inspect their definitions, and incorporate them into answers about electron donors and acceptors. The viewer made these genomic markers inspectable alongside reported physiology and evidence grades. The resulting workflow connects what an organism has been reported to use with what its genome encodes.
This evidence-retrieval capability and the prediction benchmark serve distinct purposes. A named-organism answer can incorporate published physiological observations and therefore cannot alone demonstrate prediction accuracy for uncharacterized genomes. The supervised benchmark addressed the latter question by withholding complete taxonomic groups during evaluation.

## KO features provided a useful reference model

B2 achieved macro AUPRC of 0.576 under genus-held-out evaluation and 0.560 under family-held-out evaluation (Table 1), exceeding the taxonomic baseline by 0.104 and 0.180, respectively. Curated marker rules alone achieved 0.317 under both evaluations. The trained KO model therefore captured more of the benchmark label structure than the tested marker rules or transfer from training relatives.
Bacformer-large alone reached 0.577 at genus level and 0.531 at family level. Its genus-level point estimate was nearly identical to B2, while the family-level score was lower. Fusion models improved internal point estimates, reaching 0.625 and 0.585 with B2XL plus Bacformer-large. This configuration and B2XL plus ESM-2 retained an advantage over B2 after correction across model variants. However, their advantage over B2XL, the KO-only ensemble, was not statistically resolved. The internal results therefore do not isolate a benefit specific to sequence representations beyond learner ensembling.

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


The completed attention-model rerun achieved genus-held-out AUPRC of 0.548 compared with 0.544 for mean pooling. The difference was 0.004, with a 95% confidence interval of −0.024 to 0.045. It provided no clear evidence that attention improved on pooling in this setting, resolving the earlier comparison's dependence on annotation-based protein preselection.

## External evaluation and taxonomic separation

On the 562 external genomes, B2 achieved macro AUPRC of 0.511 versus 0.380 for the taxonomic baseline. The paired difference was 0.131, with a 95% confidence interval of 0.100 to 0.167. This supports transfer of useful genomic information to unseen genera under the external annotation scheme. Nevertheless, interpretation depends on the mapping of trait definitions and missing values between compilations.
Completed external evaluation of the two strongest internal fusion configurations did not support transfer of their internal gains. B2 was consequently retained as the deployment model. These findings distinguish an improvement within the development benchmark from an improvement that persists under an external phenotype collection.
A random-split analysis produced macro AUPRC of 0.647 compared with 0.576 under genus-held-out evaluation. The difference of 0.071 illustrates the sensitivity of performance estimates to the separation of related organisms. The analysis also examined alternative fold allocations, GTDB taxonomy, restriction to complete genomes, and treatment of empty label blocks as unknown, with the main conclusions reported to remain qualitatively consistent.

## Predictability varied among metabolic traits

Aggregate performance concealed substantial variation among traits (Table 2). Sulfate reduction, sulfur oxidation, H2 oxidation, methanogenesis/acetogenesis, aerobic respiration, and ammonia oxidation had B2 AUPRC values between 0.817 and 0.887. Nitrate reduction reached 0.678. Lower values were observed for several other categories, particularly Fe(III) reduction and the residual donor and acceptor groups.
The positive counts demonstrate why absolute AUPRC values should be interpreted with care. Nitrite oxidation had only 12 positives among 358 genomes, yet its AUPRC of 0.337 was well above the approximate positive prevalence. Conversely, a common trait can have a high AUPRC while requiring more careful threshold selection for useful binary predictions. MCC supplies a complementary view of the selected calls, but practical screening also requires attention to precision, uncertainty, and experimental workload.

Table: Per-trait B2 performance under genus-held-out evaluation. The combined methanogen/acetogen and residual categories are retained as defined in the benchmark.

|  Trait | Positive Genomes | AUPRC | MCC |
| -------- | -------- | -------- | -------- |
| Sulfate reduction | 63 | 0.887 | 0.76 |
| Sulfur oxidation | 131 | 0.876 | 0.70 |
| H2 oxidation | 208 | 0.873 | 0.50 |
| Methanogens / acetogens | 75 | 0.863 | 0.85 |
| O2 respiration | 177 | 0.828 | 0.68 |
| Ammonia oxidation | 32 | 0.817 | 0.74 |
| Nitrate reduction | 102 | 0.678 | 0.48 |
| Sulfur reduction | 50 | 0.530 | 0.44 |
| Fe(II) oxidation | 43 | 0.482 | 0.35 |
| CO oxidation | 45 | 0.442 | 0.35 |
| Nitrite oxidation | 12 | 0.337 | 0.43 |
| Fe(III) reduction | 32 | 0.214 | 0.07 |
| Other acceptors | 23 | 0.136 | 0.19 |
| Other donors | 23 | 0.095 | 0.08 |


Outputs for the 228 genomes without benchmark phenotype labels included raw and calibrated scores and reliability flags. Binary calls were restricted to traits showing predictive skill beyond training lineages; other outputs remained scores without a categorical call. These outputs prioritize hypotheses for investigation rather than establishing observed physiology.

## Pathway annotations complemented broad trait labels

Scoring the 71 KEGG modules identified a median of six complete modules per genome. Mean completeness was higher in genomes labeled positive than negative for corresponding traits: 0.81 versus 0.15 for dissimilatory sulfate reduction, 0.60 versus 0.09 for SOX thiosulfate oxidation, 0.52 versus 0.07 for nitrification, 0.49 versus 0.06 for CO2-dependent methanogenesis, and 0.46 versus 0.11 for denitrification.
These annotations supplied distinctions not represented by the broad phenotype categories. For example, denitrification and dissimilatory nitrate reduction to ammonium were represented by separate modules despite contributing to the same nitrate-reduction category. Pathway annotations also provided finer resolution within the combined methanogenesis/acetogenesis category. Agreement between completeness and labels makes the modules useful supporting evidence, but completeness describes encoded potential rather than demonstrated pathway activity.

## Organism identity changed the LLM comparison

On the 96-genome subset, anonymized gene-evidence prompts yielded macro AUPRC of 0.578 for Claude and 0.588 for GPT-6 Astra, compared with 0.666 for B2, 0.687 for B2XL, and 0.697 for the strongest fusion model (Table 3). Both LLM configurations exceeded the taxonomic baseline of 0.485 but performed below the trained models. These subset scores should not be compared directly with the full-cohort values in Table 1.
Supplying organism names raised Claude's score to 0.777. Named GPT reached 0.731 among the 84 genomes with complete responses. On those same 84 genomes, its paired difference from the strongest fusion model was 0.035, with a confidence interval including zero. The comparison therefore uses matched cases, rather than subtracting a 96-genome model score from the 84-genome LLM result.


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

Named-organism answers can benefit from prior taxonomic or physiological knowledge. This is valuable when the aim is to synthesize existing evidence, including through TogoMCP, but it does not establish equivalent performance for an uncharacterized genome. The comparison alone cannot determine whether the additional information came from memorization, retrieval, or another use of organism identity.

## Phenotype disagreements identified curation priorities

Across 179 shared species, agreement between phenotype compilations was Cohen's kappa 0.61, compared with 0.52 between the model and the external compilation. A prominent discrepancy involved H2 oxidation: the compilations labeled 125 versus 33 species positive. Excluding this category increased agreement between compilations to 0.68. These observations identify label scope and evidence coverage as contributors to evaluation differences.
The analysis also flagged cases in which genomic markers or other physiological records disagreed with negative labels. Such cases are candidates for re-curation using the viewer and supporting sources. A marker-supported prediction is not proof of a labeling error: regulation, pathway direction, growth conditions, and strain differences can also explain disagreement. Cross-compilation agreement likewise does not establish a numerical ceiling on AUPRC, because kappa and AUPRC measure different aspects of agreement and prediction.

# Discussion

The main outcome of the hackathon was a shared workflow connecting sequence-derived annotations with physiological evidence. The RDF resource supplies identifiable proteins, KO assignments, and functional descriptions. Literature-assisted curation supplies candidate phenotype labels and supporting statements. Visualization makes agreement and conflict inspectable, and the benchmark tests prediction beyond labeled training groups. Module annotations provide additional biological context for interpreting broad trait categories.
TogoMCP and supervised prediction have complementary roles in this workflow. TogoMCP supports flexible evidence gathering about characterized organisms, while the trained models provide reproducible scores when physiological information is unavailable. A future MCP prediction tool could combine a trait score with supporting KOs, pathway completeness, and performance measured under genus- and family-held-out evaluation. Reporting whether the queried genome or its lineage appeared in training would further help users assess the evidence. This classifier integration remains future work; the completed integration exposed the underlying RDF annotations.

## Implications for model development and data curation

The results favor a compact KO-based deployment model under the evaluated conditions. Fusion improved internal scores but did not transfer that advantage to the external annotation set, and its internal benefit could not be separated from ensembling KO-based learners. The completed attention comparison also provided no clear improvement over mean pooling. These findings support continued work on targeted genomic evidence and phenotype harmonization alongside further model development, rather than assuming that a larger representation model will improve this benchmark.
The labeled cohort remains small for comparing many configurations, especially for rare traits. In addition, unreported substrates are not necessarily tested negatives, species-level descriptions may differ from strain physiology, and compilations can share underlying publications. The observed disagreements suggest concrete curation targets, including clearer substrate definitions, explicit unknown values, and preservation of strain-specific evidence. They do not demonstrate that label inconsistency is the sole limitation on performance.

## Prospects for cultivation and biological discovery

Compatible donor, acceptor, and carbon fixation predictions could help prioritize substrate combinations for experimental screening. For example, evidence of sulfur oxidation together with respiratory nitrate reduction could motivate testing sulfur-containing electron donors with nitrate as an acceptor. The relevant functions must nevertheless operate together under suitable conditions, and assimilatory transformations must be distinguished from energy-conserving respiration.
Actual medium design additionally depends on nutrient requirements, temperature, pH, substrate concentrations, toxicity, and regulation. Neither KO presence nor module completeness establishes active metabolism, and missing annotations in a MAG may reflect incomplete reconstruction rather than true gene absence. Our focus on known autotrophs also limits extrapolation to other lifestyles. We did not experimentally validate cultivation predictions during this work. Testing defined substrate combinations in cultured strains, followed by evaluation on incomplete and uncharacterized genomes, is necessary to establish practical utility.

# Conclusions

During BioHackathon 2026, we developed resources linking microbial genome annotations to electron donor and acceptor information, made genomic evidence accessible through TogoMCP, and evaluated several approaches to trait prediction. KO features provided a useful reference model, while external validation constrained the interpretation of internal fusion gains. Pathway annotations and phenotype disagreements offered complementary routes for inspecting predictions. The resulting resources provide a basis for reproducible screening, continued curation, and experimental testing of genome-informed cultivation hypotheses.

# Data and software availability

The manuscript repository is BH26-microbial-metabolic-traits. The microbial annotation resource was integrated with TogoMCP, whose source repository documents the service. The electron_predictor repository provides marker-matching software and an autotroph viewer.
The analysis outputs include model artifacts, metric tables, split assignments, and predictions for genomes without benchmark labels. [TO ADD: versioned public links to these outputs, the accession manifests, and source evidence tables.]

# Acknowledgements and declarations

We thank the BioHackathon 2026 organizers and participants and the maintainers of the resources used here. [TO CONFIRM: author contributions, funding, and competing interests.] LLMs assisted evidence extraction, trait prediction experiments, and manuscript drafting; authors must review the final text and underlying evidence.

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
