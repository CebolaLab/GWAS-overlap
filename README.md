# GWAS-overlap

What is a GWAS 'signal'?

- A **causal variant** increases risk of disease.
- A GWAS (**G**enome **W**ide **A**ssociation **S**tudy) will observe this SNP at a higher frequency in a cohort of cases, compared to controls. 
- Due to haplotype structure, all SNPs in high LD with the causal SNP are also present at a higher frequency in cases.
- All of the variants on the same **haplotype** as the causal SNP will therefore be observed to be *significantly associated* with disease.
- This set of associated SNPs makes up a GWAS 'signal' 

A GWAS **signal** is therefore hypothesised to "capture" or "result from" just **one causal variant**. However, the observed results will consist of a set of associated SNPs in high LD, one of which should be the causal variant.

### Combining GWAS 

- Two independent GWAS may observe significant association resulting from the same causal variant. They therefore observe the same "signal". 
- GWAS reports label each "signal" using one unique identifier, typically the **lead SNP**.
- The **lead SNP** is simply the SNP which, in that study, had the smallest p-value.
- While the causal variant remains the same, two different GWAS may report two different **lead SNPs**. 

This can be due to technical variations in e.g. missing data, imputation quality, random fluctuations, LD structure. At a signal where SNPs are in complete LD (and therefore have the same level of association), small differences in the thousands of samples can be enough to make one SNP *slightly* more significant than another. These variations are random and therefore the lead SNP can differ between studies. 

* * * * SKIP * * * * 

```R
setwd('/rds/general/project/cebolalab_liver_regulomes/live/amp_t2d/LSECs/clustering_LSEC_CREs/Dorka_GWAS_overlaps/NAFLD_variants/NAFLD_Vujkovic')
data=read.table('test')
#The file is concatenated from multiple GWAS, so other studies have been added as new rows

#Name columns 
colnames(data)=c('chr','start','end','lead','SNP')
```

<img src="https://github.com/CebolaLab/GWAS-overlap/blob/main/Figures/Figure1.png" width="200" height="200" 



