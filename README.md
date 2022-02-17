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

### Combining credible sets or expanded SNPs 

Depending on the GWAS, either the lead SNP or credible sets may be provided.

Independent GWAS which capture the SAME SIGNAL (i.e. an association caused by the same causal variant) may have different lead SNPs, as described above. These lead SNPs will be in *high LD*. See the example below, where the image shows a haplotype of high LD.

<img src="https://github.com/CebolaLab/GWAS-overlap/blob/main/Figures/Figure2.png" height="200">

The complete 'signal' of association from each GWAS will consist of the credible set, or the expanded set of SNPs. Two GWAS which capture the same causal variant will have an *almost* identical list of SNPs making up the "signal". However, there might be one or two SNPs included in one GWAS and not the other. This may be, for example, if there are several SNPs in high LD (r^2 > 0.8) with the GWAS 1 lead SNP, which do not reach the LD threshold for GWAS 2 lead SNP. In the example below, both GWAS 1 and GWAS 2 capture 2 SNPs in the expanded SNPs which were not captured by the other. 

<img src="https://github.com/CebolaLab/GWAS-overlap/blob/main/Figures/Figure3.png" height="200">

For the combined results, we want the complete list of SNPs from both studies, as shown below. We will assign an arbitrary label to these SNPs which identifies them as belonging to the same signal (e.g. we could use the lead SNP from GWAS 1). 

<img src="https://github.com/CebolaLab/GWAS-overlap/blob/main/Figures/Figure4.png" height="300">

# SKIP (insert plink expansion and concatenating files)

## Data integration

In practise, we will have a dataframe combining GWAS results. There should be a column for the SNPs and a column for the lead SNP which acts as the "label" for the GWAS signal they belong to (remember that each signal is in theory capturing just one causal variant). 

```R
setwd('/rds/general/project/cebolalab_liver_regulomes/live/amp_t2d/LSECs/clustering_LSEC_CREs/Dorka_GWAS_overlaps/NAFLD_variants/NAFLD_Vujkovic')
data=read.table('test')
#The file is concatenated from multiple GWAS, so other studies have been added as new rows

#Name columns 
colnames(data)=c('chr','start','end','lead','SNP')
```

<img src="https://github.com/CebolaLab/GWAS-overlap/blob/main/Figures/Figure1.png" height="200">

In this example, we can see several duplicated SNPs. E.g. rs132641 is present twice with two lead SNPs (rs132665 and rs132662). These two lead SNPs must therefore be labelling the same signal from two indpendent GWAS.

To combine the data for this signal, we want to have:

1. The **SNP column** with the 'expanded' SNPs for this signal, which will consist of the merged lists of expanded SNPs from both studies (we expect this list to be almost identical, since the two lead SNPs will be in high LD, but there may be a couple of SNPs captured in only study as different lead SNPs were expanded).
2. The **lead column** with **ONE** unique identifier to 'label' this signal. We will use one of the two lead SNPs (we will arbitrarily take the first one from the list).

We can identify all the duplicated SNPs:

```R
duplicated.snps=as.character(subset(data$SNP,duplicated(data$SNP)))
#rs132641, rs1547014, rs5752775
```

We will use these duplicated SNPs to identify instances where two GWAS capture the same signal (identified where the same SNP(s) are in the SNP lists for both studies), but have different lead SNPs. We want to later be able to identify these SNPs as coming from the same signal, so we just want one unique identifier we can later filter on. 

Let's work with the first duplicated SNP:

```R
x=duplicated.snps[1] #rs132641
```

In this instance, our first duplicated SNP, rs132641, is matched with two lead SNPs: rs132665 and rs132662.
We now want to take ALL SNPs which are labelled with *either* of these lead SNPs. 
These SNPs make up the 'combined list', as shown in the Figure above. We will replace the value in the 'lead SNPs' column with just one unique identifier (we will use the first lead SNP, rs132665).

```R
#subset the dataframe for rows where th value in the 'lead' column is in our list of lead SNPs. Replace the value in the lead column ($lead) with the lead SNP from the first GWAS (leadSNPs[1])
data[data$lead %in% leadSNPs,]$lead = leadSNPs[1]
```

Before:  <br /> 
<img src="https://github.com/CebolaLab/GWAS-overlap/blob/main/Figures/Figure5.png" height="200">

After:  <br /> 
<img src="https://github.com/CebolaLab/GWAS-overlap/blob/main/Figures/Figure6.png" height="200">

We now want to apply this to the rest of the data.
We can loop through the list of duplicated SNPs. However, some of these will below to the signal which we have just completed. We can therefore create a list of the lead SNPs which we have already merged. While looping through the duplicated SNPs, we will skip any which are assigned to a lead SNP which has already been merged.

```R
#Our list of lead SNPs which have been completed already from above
completed=leadSNPs

for(x in duplicated.snps[-1]){
    #check the lead SNP for this duplicated SNP
    leadSNPs=unique(data[data$SNP==x,]$lead)
    #We want to test if the leadSNPs are in the completed list. We use an if statement which accepts a TRUE/FALSE. However, leadSNPs will be a vector of two or more values and an "if" statement needs just one TRUE/FALSE. We use the SUM command which will give the total number of TRUE values. If the sum is greater than zero, then the lead SNPs are already in our 'completed' list and should be skipped.
    if(sum(leadSNPs %in% completed)==0){
        #If this is a new signal, repeat the process and replace the lead column with the first lead SNP
        data[data$lead %in% leadSNPs,]$lead = leadSNPs[1]
        #Add the leadSNPs to the completed list
        completed=c(completed,leadSNPs)
    }
}

```