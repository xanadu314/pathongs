# Qiime2



## 生成样品信息表

```{bash}
# 可以用perl命令generate manifest.tsv
find `pwd` -name "*gz" | grep '_R1' | sort | perl -e 'print"sampleid\tforward-absolute-filepath\treverse-absolute-filepath\n"; while(<>){chomp; $name=(split("_R|\/", $_))[-2]; $name1=$name; $name2=$_; $name1=~s/_[1|2]_.fq.gz//g; $name2=~s/R1/R2/; print "$name1\t$_\t$name2\n";}' > pe-33-manifest-trimmed.tsv
```

| sample-id | forward-absolute-filepath | reverse-absolute-filepath |
| --------- | ------------------------- | ------------------------- |
| Test1     | Test1_R1.fastq.gz         | Test1_R2.fastq.gz         |
| Test2     | Test2_R1.fastq.gz         | Test2_R2.fastq.gz         |
| Test3     | Test3_R1.fastq.gz         | Test2_R2.fastq.gz         |

## 原始序列质控

```{bash}
cat pe-33-manifest-trimmed.tsv |grep -v sampleid|while read id1 id2 id3; do \
fastp -i ${id2} -I ${id3} -o ${id1}_R1_clean.fastq.gz -O ${id1}_R2_clean.fastq.gz \
-h ${id1}.html -j ${id1}.json; done
```

查看网页版的质控结果，test.html，确定后面Dada2的质控参数。

## 导入双端测序文件

step1: 准备符合import格式的文件 manifest.tsv(包含样本ID以及forward和reverse fq路径的文件)和sample-metadata.tsv(样本的分组信息)

step2: import fastq into qza，双端和单端数据的import参数不同，并且不同的phred score使用的参数也不一样

```{bash}
if [ ! -d result ]; then mkdir -p result; fi
# PE mode 
qiime tools import \
      --type 'SampleData[PairedEndSequencesWithQuality]' \
      --input-path pe-33-manifest-trimmed.tsv \
      --output-path result/paired-end-demux.qza \
      --input-format PairedEndFastqManifestPhred33V2

# SE mode 
qiime tools import \
  --type "SampleData[SequencesWithQuality]" \
  --input-path single-33-manifest.tsv \
  --output-path result/single-end-demux.qza \
  --input-format SingleEndFastqManifestPhred33V2 
```

## Qiime2质控

Step1: 为了更加准确地过滤低质量的碱基，可以再使用qiime2自带summarize插件查看低质量碱基的位置分布，最后再结合第二步usearch和vsearch的primer位置信息设置适合过滤的参数。

```{bash}
qiime demux summarize \
  --i-data result/paired-end-demux.qza \
  --o-visualization result/paired-end-demux-summary.qzv
```

## Dada2 Denoise 

```{bash}
# DADA2 denosie
qiime dada2 denoise-paired \
        --i-demultiplexed-seqs result/paired-end-demux.qza \
        --p-trim-left-f 23 \
        --p-trim-left-r 19 \
        --p-trunc-len-f 0 \
        --p-trunc-len-r 0 \
        --p-n-threads 20 \
        --o-table result/table.qza \
        --o-representative-sequences result/rep-seqs.qza \
        --o-denoising-stats result/stats.qza

# summary feature table
 qiime feature-table summarize \
        --i-table result/table.qza \
        --o-visualization result/table.qzv \
        --m-sample-metadata-file sample-metadata.tsv
```

## Picrust2

得到的table.qza文件和rep-seqs.qza 文件可以解压后用于Picrust2分析。其中table.qza解压后为二进制的feature-table.biom文件。

## 序列合并

```{bash}
qiime vsearch join-pairs  --i-demultiplexed-seqs  ./ewat.qza --o-joined-sequences ewat-joined-filtered.qz
```

## 数据过滤和可视化

```{bash}
qiime demux summarize --i-data ./ewat-joined-filtered.qz.qza --o-visualization ./ewat-demux-joined-filtered-visualized.qzv
```

## qiime单端

由于DADA2对V3-V4的组装效果不好（要求太严格），会过滤掉很多reads，所以现在测序公司通常先做双端reads的merge，再按照单端的流程走dada2

先用fast 对原始数据进行质控和merge

### 构建单端数据表

```{bash}
find `pwd` -name "*gz" | grep '分隔符' | sort | perl -e 'print"sampleid\tabsolute-filepath\n"; while(<>){chomp; $name=(split(".分隔符|\/", $_))[-2]; $name1=$name; $name2=$_; $name1=~s/.fq.gz//g; $name2=~s/R1/R2/; print "$name1\t$_\n";}' > se-33-manifest-trimmed.tsv
```

### 导入合并的测序数据

```{bash}
if [ ! -d result ]; then mkdir -p result; fi
qiime tools import \
  --type "SampleData[SequencesWithQuality]" \
  --input-path single-33-manifest.tsv \
  --output-path result/single-end-demux.qza \
  --input-format SingleEndFastqManifestPhred33V2 
```

### Dada2-denoise

如果不想质控，直接进行denoise

```{bash}
qiime dada2 denoise-single \
--i-demultiplexed-seqs ./result/single-end-demux.qza \
--p-trim-left 0 \
--p-trunc-len 420 \
--o-representative-sequences ./result/rep-seqs-dada2.qza \
--o-table ./result/table.qza \
--o-denoising-stats ./result/dada2-stats.qza 
```

### 导出特征表和参考序列

```{bash}
mkdir phyloseq
qiime tools export \
--input-path table.qza \
--output-path phyloseq

biom convert \
-i phyloseq/feature-table.biom \
-o phyloseq/otu_table.tsv \
--to-tsv
cd phyloseq; sed -i '1d' otu_table.tsv
sed -i 's/#OTU ID/ASV/' otu_table.tsv

qiime tools export \
--input-path rep-seqs-dada2.qza \
--output-path phyloseq
```

### 对序列进行注释

```{bash}
qiime feature-classifier classify-sklearn \
--i-classifier ~/16s/database/silva-138-99-nb-weighted-classifier.qza \
--i-reads result/rep-seqs-dada2.qza \
--o-classification result/taxonomy-dada2-sliva.qza \
--p-n-jobs 20 \
--verbose
```

### 注释结果可视化

```{bash}
qiime metadata tabulate \
--m-input-file ./result/taxonomy-dada2-sliva.qza \
--o-visualization ./result/taxonomy-dada2-sliva.qzv 
```

