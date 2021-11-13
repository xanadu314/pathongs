

# 微生物组溯源分析

介绍两款发表在Nature Methods上的R包，用于扩增子数据的溯源分析或者污染分析。

## SourceTrack

SourceTracker软件于2011年被发表在Nature Methods[https://www.nature.com/articles/nmeth.1650]上，由圣地亚哥大学的Scott教授及其团队完成。

该软件称目标样本为Sink，微生物污染源或来源的样品为Source；基于贝叶斯算法，探究目标样本（Sink）中微生物污染源或来源（Source）的分析。根据Source样本和Sink样本的群落结构分布，来预测Sink样本中来源于各Source样本的组成比例。

其官方网站为：[danknights/sourcetracker (github.com)](https://github.com/danknights/sourcetracker)

软件有Qimme脚本和R脚本两种形式，建议直接使用R脚本。

metadata手动生成，otu表格必须是纯整数型变量

| #Sampleid             | Description | Env     | SourceSink | Study | Details                              |
| --------------------- | ----------- | ------- | ---------- | ----- | ------------------------------------ |
| Run20100430_H2O-1     | PCR         | water 1 | sink       | Lab 1 | PCR_water_sample_1_2010_04_30_run    |
| Run20100524_ESC_C-3ss | Swab 2      | Lab 2   | sink       | Lab 2 | Sterile_swab_sample_3_2010_05_24_run |
| F11Fcsp               | Gut 1       | Gut     | source     | 1     |                                      |
| F12Fcsp               | Gut 2       | Gut     | source     | 1     |                                      |
| F13Fcsp               | Gut 3       | Gut     | source     | 1     |                                      |



```{r}
# This runs SourceTracker on the original "contamination" data set
# (data included in 'data' folder)

# load sample metadata
metadata <- read.table('data/metadata.txt',sep='\t',h=T,row.names=1,check=F,comment='')

# load OTU table
# This 'read.table' command is designed for a 
# QIIME-formatted OTU table.
# namely, the first line begins with a '#' sign
# and actually _is_ a comment; the second line
# begins with a '#' sign but is actually the header
otus <- read.table('data/otus.txt',sep='\t', header=T,row.names=1,check=F,skip=1,comment='')
otus<-otus[rowSums(otus)>10,]
otus <- t(as.matrix(otus))

# extract only those samples in common between the two tables
common.sample.ids <- intersect(rownames(metadata), rownames(otus))
otus <- otus[common.sample.ids,]
metadata <- metadata[common.sample.ids,]
# double-check that the mapping file and otu table
# had overlapping samples
if(length(common.sample.ids) <= 1) {
    message <- paste(sprintf('Error: there are %d sample ids in common '),
                    'between the metadata file and data table')
    stop(message)
}

# extract the source environments and source/sink indices
train.ix <- which(metadata$SourceSink=='source')
test.ix <- which(metadata$SourceSink=='sink')
envs <- metadata$Env
if(is.element('Description',colnames(metadata))) desc <- metadata$Description


# load SourceTracker package
source('src/SourceTracker.r')

# tune the alpha values using cross-validation (this is slow!)
# tune.results <- tune.st(otus[train.ix,], envs[train.ix])
# alpha1 <- tune.results$best.alpha1
# alpha2 <- tune.results$best.alpha2
# note: to skip tuning, run this instead:
alpha1 <- alpha2 <- 0.001

# train SourceTracker object on training data
st <- sourcetracker(otus[train.ix,], envs[train.ix])

# Estimate source proportions in test data
results <- predict(st,otus[test.ix,], alpha1=alpha1, alpha2=alpha2)

# Estimate leave-one-out source proportions in training data 
results.train <- predict(st, alpha1=alpha1, alpha2=alpha2)

# plot results
labels <- sprintf('%s %s', envs,desc)
plot(results, labels[test.ix], type='pie')

# other plotting functions
# plot(results, labels[test.ix], type='bar')
# plot(results, labels[test.ix], type='dist')
# plot(results.train, labels[train.ix], type='pie')
# plot(results.train, labels[train.ix], type='bar')
# plot(results.train, labels[train.ix], type='dist')

# plot results with legend
# plot(results, labels[test.ix], type='pie', include.legend=TRUE, env.colors=c('#47697E','#5B7444','#CC6666','#79BEDB','#885588'))
```

## FEAST

2019年来自加州大学洛杉矶分校的Eran Halperin教授在Nature Methods上发表了快速准确的微生物来源追溯工具FEAST[][FEAST: fast expectation-maximization for microbial source tracking | Nature Methods](https://www.nature.com/articles/s41592-019-0431-x]

1. 快速准确的微生物来源分析一直是本领域的难点，之前发布的SourceTracker仍有速度慢，准确率不高的问题；
2. 本文提出一种新的方法FEAST，可以实现快速、更准确的微生物来源追踪；
3. 软件基于R语言开发，保证了方法跨平台的可用性；
4. 应用于婴儿和厨房两个微生物组项目，结果的微生物来源解释比例更合理；
5. 此方法在分类问题中，也比JSD、加权UniFrac指标有更好的AUC值，在医学诊断中有更好的应用前景。

本R包安装比SourceTrack要复杂一些

注意meta信息需要确定是study是1对1还是1对多，1对多sink study id不要重复，source id留空白。

Installation
---------------------------

*FEAST* will be available on QIIME 2 very soon. Until then you can you can simply install *FEAST* using **devtools**: 
```{r}
devtools::install_github("cozygene/FEAST")
```

## Usage
As input, *FEAST* takes mandatory arguments:

- _C_ - An _m_ by _n_ count matrix, where _m_ is the number samples and _n_ is the number of taxa.
- _metadata_ - An _m_ by 3 table, where _m_ is the number of samples. The metadata table has three columns (i.e., 'Env', 'SourceSink', 'id'). The first column is a description of the sampled environment (e.g., human gut), the second column indicates if this sample is a source or a sink (can take the value 'Source' or 'Sink'). The third column is the Sink-Source id. When using multiple sinks, each tested with the same group of sources, only the rows with 'SourceSink' = Sink will get an id (between 1 - number of sinks in the data). In this scenario, the sources’ ids are blank. When using multiple sinks, each tested with a distinct group of sources, each combination of sink and its corresponding sources should get the same id (between 1 - number of sinks in the data). Note that these names must be respected.
- _EM_iterations_ - A numeric value indicating the number of EM iterations (default 1000).
- _COVERAGE_ - A numeric value indicating the rarefaction depth (default = minimal sequencing depth within each group of sink and its corresponding sources).
- _different_sources_flag_ - A Boolean value indicating the source-sink assignment. different_sources_flag = 1 if different sources are assigned to each sink , otherwise = 0.
- _dir_path_ - A path to an output.txt file.
- _outfile_ - the prefix for saving the output file.

Value: 

*FEAST* returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1. Pij is the contribution of source j to sink i. If Pij == NA it indicates that source j was not used in the analysis of sink i. *FEAST* will save the file "demo_FEAST.txt" (a file containing matrix P) .




Demo
-----------------------
We provide a dataset for an example of FEAST usage. Download the demo files <a href="https://github.com/cozygene/FEAST/tree/FEAST_beta/Data_files">here</a>.

First load the **FEAST** packages into R:
```{r}
library(FEAST)
```

Then, load the datasets:
```{r}
metadata <- Load_metadata(metadata_path = "~/FEAST/Data_files/metadata_example_multi.txt")
otus <- Load_CountMatrix(CountMatrix_path = "~/FEAST/Data_files/otu_example_multi.txt")
```
Run _FEAST_, saving the output with prefix "demo":

```{r}
FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/",
                      outfile="demo")
F
```

_FEAST_ will then save the file
*demo_FEAST.txt* - A file containing an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1.

Graphical representation: 

As input, *PlotSourceContribution* takes mandatory arguments:

- _SinkNames_ - A vector with the sink names to plot.
- _SourceNames_ - A vector with all the sources' names.
- **_Same_sources_flag_ - A Boolean value indicating the source-sink plotting assignment. Same_sources_flag = 1 if the same sources are assigned to the pre-defined sink samples , otherwise = 0.**
- _dir_path_ - A path to an output .png file.
- _mixing_proportions_ - A list of vectors, where entry i corresponds to the vector of source contributions (summing to 1) to sink i.
- _Plot_title_ -  Plot's title and output .png file's name.
- _N_ - Number of barplots in each output .png file.


```{r}
PlotSourceContribution(SinkNames = rownames(FEAST_output)[c(5:8)],
                       SourceNames = colnames(FEAST_output), dir_path = "~/FEAST/Data_files/",
                       mixing_proportions = FEAST_output, Plot_title = "Test_",Same_sources_flag = 0, N = 4)
```



Input format
-----------------------
The input to *FEAST* is composed of two tab-delimited ASCII text files:

(1) count table - An m by n count matrix, where m is the number samples and n is the number of taxa. Row names are the sample ids ('SampleID'). Column names are the taxa ids. Every consecutive column contains read counts for each sample. Note that this order must be respected.


count matrix (first 4 rows and columns):

| | ERR525698 |ERR525693 | ERR525688| ERR525699|
| ------------- | ------------- |------------- |------------- |------------- |
| taxa_1  |  0 | 5 | 0|20 |
| taxa_2  |  15 | 5 | 0|0 |
| taxa_3  |  0 | 13 | 200|0 |
| taxa_4  |  4 | 5 | 0|0 |



(2) metadata - An m by 3 table, where m is the number of samples. The metadata table has three columns (i.e., 'Env', 'SourceSink', 'id'). The first column is a description of the sampled environment (e.g., human gut), the second column indicates if this sample is a source or a sink (can take the value 'Source' or 'Sink'). The third column is the Sink-Source id. When using multiple sinks, each tested with the same group of sources, only the rows with 'SourceSink' = Sink will get an id (between 1 - number of sinks in the data). In this scenario, the sources’ ids are blank. When using multiple sinks, each tested with a distinct group of sources, each combination of sink and its corresponding sources should get the same id (between 1 - number of sinks in the data). Note that these names must be respected.


*using multiple sinks, each tested with the same group of sources:

| SampleID | Env |SourceSink | id |
| ------------- | ------------- |------------- |-------------|
| ERR525693  |  infant gut 2 | Sink | 2 |
| ERR525688   |  Adult gut 1 | Source| NA |
| ERR525699  |  Adult gut 2 | Source | NA |
| ERR525697  |  Adult gut 3 | Source | NA |


*using multiple sinks, each tested with a different group of sources:

| SampleID | Env |SourceSink | id |
| ------------- | ------------- |------------- |-------------|
| ERR525688   |  Adult gut 1 | Source| 1 |
| ERR525691  |  Adult gut 2 | Source | 1 |
| ERR525699  |  infant gut 2 | Sink | 2 |
| ERR525697  |  Adult gut 3 | Source | 2 |
| ERR525696  |  Adult gut 4 | Source | 2 |




Output - 

| infant gut 2  |Adult gut 1 | Adult gut 2| Adult gut 3| Adult skin 1 |  Adult skin 2|  Adult skin 3| Soil 1 | Soil 2 | unknown|
| ------------- | ------------- |------------- |------------- |------------- |------------- |------------- |------------- |------------- |------------- |
|  5.108461e-01  |  9.584116e-23 | 4.980321e-12 | 2.623358e-02|5.043635e-13 | 8.213667e-59| 1.773058e-10 |  2.704118e-14 |  3.460067e-02 |  4.283196e-01 |
