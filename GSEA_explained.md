I was asked to do a Gene Set Enrichment Analysis (GSEA) for RNA-seq data.
One of the most popular tool is [GSEA](http://software.broadinstitute.org/gsea/doc/GSEAUserGuideFrame.html) from broad Institute. To better
understand the underlying method of `GSEA`, I read the [original paper: Gene set enrichment analysis: A knowledge-based
approach for interpreting genome-wide expression profiles](http://software.broadinstitute.org/gsea/doc/subramanian_tamayo_gsea_pnas.pdf) and searched on [biostars](https://www.biostars.org/p/132575/).

I quote from the biostar post:  
>so, to run GSEA you have your list of genes (L) and two conditions (or more), i.e. a microarray with normal and tumor samples. the first thing that GSEA does is to rank the genes in L based on "how well they divide the conditions" using the probe intensity values. at this point you have a list L ranked from 1...n.  
now you want to see whether the genes present in a gene set (S) are at the top or at the bottom of your list...or if they are just spread around randomly. to do that GSEA calculates the famous enrichment score, that becomes normalized enrichment score (NES) when correcting for multiple testing (FDR).  
a positive NES will indicate that genes in set S will be mostly represented at the top of your list L. a negative NES will indicate that the genes in the set S will be mostly at the bottom of your list L.  
let's say that S1 has positive NES and S2 has negative NES. let's say also that your list of 1000 genes is ordered form the most upregulated (top: 1,2,3,....) to the most downregulated (bottom: ....n-3,n-2,n-1,n). a positive NES for S1 will mean that genes over-represented in that gene set are upregulated in your dataset. negative NES for S2 instead indicated the opposite.  
in the results you will also find a heatmap the subset of you data that belong to the signature analyzed. generally what I saw is that the more significantly enriched is the gene set, the better the division between the two conditions in the heatmap.

![](https://cloud.githubusercontent.com/assets/4106146/17038331/e2095c44-4f5a-11e6-9a19-d366912ef943.png)  

So the idea is pretty simple, one need to have a pre-ranked gene list according to [some metrics](http://software.broadinstitute.org/gsea/doc/GSEAUserGuideTEXT.htm#_Metrics_for_Ranking), and GSEA will test various curated gene sets to see if the gene sets is in the front or bottom of the ranked gene list.

**One can run GSEA in two modes:**

#### 1. using raw gene expression data
Supply a expression data file in various [formats](http://www.broadinstitute.org/cancer/software/gsea/wiki/index.php/Data_formats)
one of the common format is GCT: Gene Cluster Text file format (*.gct)).
![](https://cloud.githubusercontent.com/assets/4106146/16968303/8f65e8e6-4dd3-11e6-9a98-093eb0bd1e86.png) 
and a phenotype label file :
![](https://cloud.githubusercontent.com/assets/4106146/16968346/ca846ed4-4dd3-11e6-89a7-be32c0c62e3b.png)

Then GSEA will calculate the rank of the genes by different [matrics](http://software.broadinstitute.org/gsea/doc/GSEAUserGuideTEXT.htm#_Metrics_for_Ranking).

I read the [mannual](http://software.broadinstitute.org/gsea/doc/GSEAUserGuideTEXT.htm#_Run_GSEA_Page) of GSEA and found:
>Metric for ranking genes. GSEA ranks the genes in the expression dataset and then analyzes that ranked list of genes. 
Use this parameter to select the metric used to score and rank the genes; use the Gene list sorting mode parameter to determine 
whether to sort the genes using the real (default) or absolute value of the metric score; and use the Gene list ordering mode 
parameter to determine whether to sort the genes in descending (default) or ascending order. 
For descriptions of the ranking metrics, see [Metrics for Ranking Genes](http://software.broadinstitute.org/gsea/doc/GSEAUserGuideTEXT.htm#_Metrics_for_Ranking).

>Note: The default metric for ranking genes is the signal-to-noise ratio. 
To use this metric, your phenotype file must define at least two categorical phenotypes and your expression dataset must contain at least **three (3)** samples for each phenotype. If you are using a continuous phenotype or your expression dataset contains fewer than three samples per phenotype, you must choose a different ranking metric. 

>**If your expression dataset contains only one sample, you must rank the genes and use the GSEAPreranked Page to analyze the ranked list; none of the GSEA metrics for ranking genes can be used to rank genes based on a single sample.**

#### 2. [using a pre-ranked gene list](http://software.broadinstitute.org/gsea/doc/GSEAUserGuideTEXT.htm#_GSEAPreranked_Page)

**The take-home message is that no matter what mode you use, internally GSEA is going to rank your list of genes first and then it will compare the public-curated gene sets with the ranked gene list you have.**

The key here is that you need to supply **ALL genes** that are detected in the experiment (all probes for microarray and all genes detected in the RNA-seq experiment). Read a blog post by [Mark Ziemann: Data analysis step 8: Pathway analysis with GSEA](http://genomespot.blogspot.com/2014/09/data-analysis-step-8-pathway-analysis.html) and my comments below the post.

by Roberto  
> great post! Just a question on gene expression direction.

>In your examples, genes tend to be consistently only up-regulated or only down-regulated. Now let's say that you have immune genes, both strongly up-regulated and strongly down-regulated. The up-regulated ones are activators of immunity, and the downregulated ones are suppressors of immunity. It makes biological sense that the former go up and the latter go down.

>Then you have a gene set "Immune function" comprising both activators and suppressors of immunity, i.e. it lists genes related to immunity, but it is not skewed towards activation or suppression.

>When you run GSEA pre-ranked, the ES will go up immediately (upregulated activators), then it will decrease slowly, and then it will have another sharp increase (downregulated suppressors).

>Conversely, if both up and downregulated genes were listed at the top (by using the inverse of p-value, without the fold change sign), the ES would achieve a higher max.

>So my question is: if gene sets do not distinguish between activators and repressors, should we forget about the sign and place both up- and down-regulated genes at the top of our ranked list?

by Mark:  
>Hi Roberto, thanks for your comment. Indeed I perform pathway analysis considering up and down regulated genes as separate groups. This does somewhat contradict the KEGG/Reactome gene sets that contain both repressors and activators. These curated gene sets are hand picked largely based on protein functional data so we understand that there are limitations when talking about gene expression assays.

>The solution you propose, that GSEA be based on the variation of a gene set from the null (no change), is totally possible and very interesting but I haven't tried it yet.

>BUT the benefit of keeping the up- and down-regulated analysis separate is that you may be able to pinpoint smaller sub-pathways that are in either direction. For instance "immune function" could be made up of smaller pathways like "IFN signalling" or "LPS stimulation" that are moving in opposing directions. 

>Essentially the assumption with the direction-separated analysis is that the biological mechanisms underlying the up- and down- regulated genes is different, which in general is true and is consistent with our example above with immune function down and cell cycle up.

Me:  
>Hi mark, Stumbling into this old post and thanks for it. I had the same question. so when you give the pre-ranked gene list to GSEA, you separate them to 2 groups: upregulated (+ sign) and downregulated(-sign) and do GSEA respectively? thx.

Mark:  
>
Hi Tommy, I don't separate the lists as GSEA preranked is able to identify up and down regulated pathways by default. Cheers!

Me:  
>so the gene sets denote whether a certain gene is activator or repressor for the pathway? A down-regulated pathway may consist of genes of activators(downregulated) and genes of repressors (upregulated). correct me if I miss something.﻿

Mark:
>GSEA preranked is a pretty simple test. It just checks whether gene set members are clustered at either end of the rank file and what the chances are of this enrichment occurring at random. The gene sets can be curated in any way, you will see that MSigDB contains both curated (KEGG, Reactome) and experimental (microarray, GWAS, proteomics) gene sets. GSEA doesn't know whether each individual gene is an activator or repressor, it only knows the gene name and the gene sets that it is present.

Roberto
> Hi Mark, because GSEA preranked can find enrichment at either end, but not both ends, of the rank file, then it would make sense to test separately upregulated and downregulated genes. I guess this is what Tommy is interested in, and that would be in line with our previous conversation above. I think the concern is that a pathway annotated by, say, KEGG, will comprise both positive regulators of that pathway (activators) and negative regulators (repressors). With the due limitations, if a pathway is "activated", then in a RNA-seq experiment we would see activators being upregulated and repressors being downregulated. Therefore, we would see enrichment of gene set members at both ends on the rank file, not just at the top or just at the bottom. My understanding is that GSEA results are less significant when there is enrichment at both ends. It works better when enrichment is focused at either end, something we can achieve by providing only upregulated or downregulated genes, separately in two separate GSEA runs. 

>This seems also to be the sense of this question in the GSEA FAQ:

>http://www.broadinstitute.org/cancer/software/gsea/wiki/index.php/FAQ#Can_I_use_GSEA_with_gene_sets_that_have_both_up-_and_down-regulated_genes.3F

>Although both the question and the answer are very confusing.

Mark:  
>True that curated pathways like KEGG include activators and repressors, but if you look at the contents of these gene sets, they are predominantly downstream effectors and as such, they will mostly be co-regulated.

>If your null hypothesis is that the gene set members are unchanged (and found in the middle of the rank file), then you might be able to test this with GSEA by assigning a metric based on the significance irrespective of fold change direction.﻿

I will post how to do a GSEA practially use R in a separate post.