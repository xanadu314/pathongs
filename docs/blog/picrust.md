

# 强大的菌群功能预测工具

16S测序只能获得菌株科属的信息，对于微生物的功能，可以用一些软件进行预测。

基于物种进行功能预测的原理，简单说来就是将测序、归类后得到的OTU 信息输入到预测软件中，预测软件将OTU序列信息与已测序的微生物基因组数据库中物种进行比对，将OTU注释为对应物种，并根据OTU的丰度输出功能类型以及对应功能丰度。

![img](http://5b0988e595225.cdn.sohucs.com/images/20200324/90b8046a12d044cbba86bdc9df529c21.jpeg)

因此，使用的预测软件以及软件所依赖的微生物基因组数据库在很大程度上决定了功能分析的成败。当前可供使用的软件有很多：PICRUSt、Tax4Fun、FAPROTAX、BugBase 等预测软件侧重方向不同，各有千秋。

2020年，PICRUSt升级到PICRUSt2，不再依赖于Greengene 数据库进行比对，而是采用将待预测的 OTU代表序列置于软件中已有的系统发育树中进行物种注释，使用 IMG 微生物基因组数据进行功能信息的输出。

![PICRUSt2分析实战：16S扩增子OTU或ASV预测宏基因组EC、通路、KO(](https://images1.tqwba.com/20200828/3y5tjsyrhpp.png)



PICRUSt2的分析流程很多，下面介绍最经典的根据fasta序列分析和根据qiime2的结果来分析

## 从fasta序列开始分析

（新版本容易报错，需要将fasta的序列名称改短）

```{bash}
## 安装
conda create -n picrust2 -c bioconda -c conda-forge picrust2
## 将扩增子序列插入参考进化树中
place_seqs.py -s ./eWAT_B1.filtered.fasta -o eWAT_B1.filtered.tre -p 10 --intermediate intermediate/place_seq
## 预测16S的拷贝数，-n计算NSTI，预测基因组的E.C.编号，KO相度
hsp.py -i 16S -t picrust2/epa_result_parsed.newick -o picrust2/marker_nsti_predicted.tsv.gz -p 8 -n
## 预测16S的酶学委员会EC编号
hsp.py -i EC -t picrust2/epa_result_parsed.newick -o picrust2/EC_predicted.tsv.gz -p 8
## 预测16S的基因同源簇KO编号(时间长)
hsp.py -i KO -t picrust2/epa_result_parsed.newick -o picrust2/KO_predicted.tsv.gz -p 8
## 预测E.C.和KO的丰度，—strat_out输出分层结果(极大的增加计算时间)
metagenome_pipeline.py -i otutab.txt \
   -m picrust2/marker_nsti_predicted.tsv.gz \
   -f picrust2/EC_predicted.tsv.gz \
   -o picrust2/EC_metagenome

metagenome_pipeline.py -i otutab.txt \
   -m  picrust2/marker_nsti_predicted.tsv.gz \
   -f picrust2/KO_predicted.tsv.gz \
   -o KO_metagenome --strat_out
   ## MetaCyc通路丰度，基于EC结果汇总
   pathway_pipeline.py -i picrust2/EC_metagenome/pred_metagenome_unstrat.tsv.gz \
    -o picrust2/pathways \
    --intermediate pathways_working \
    -p 8
    ## KEGG通路，基于KO
    pathway_pipeline.py -i picrust2/KO_metagenome/pred_metagenome_unstrat.tsv.gz -o picrust2/KEGG_pathways --no_regroup --map ~/miniconda2/envs/picrust2/lib/python3.6/site-packages/picrust2/default_files/pathway_mapfiles/KEGG_pathways_to_KO.tsv
```



## 从Qiime2的中间结果开始分析

```{bash}
picrust2_pipeline.py -s dna-sequences.fasta -i feature-table.biom  -o picrust2_out_pipeline -p 8
```

### 核心输出结果

注：结果表全为gz压缩文件，可用less/zless查看，可用gunzip解压。

- EC_metagenome_out - 目录包括非分层的预测宏基因组EC数量 (pred_metagenome_unstrat.tsv), 基于预测16S拷贝数校正的特征表 (seqtab_norm.tsv), 每个样本的NSTI权重  (weighted_nsti.tsv)
- KO_metagenome_out - 和 EC_metagenome_out 类似, 但为宏基因组KO表
- pathways_out - 文件夹包括预测的通路丰度和覆盖度，基于EC数量丰度，一般仅有400多行

### 额外输出文件

可能对进一步分析的经验用户更有用：

- EC_predicted.tsv - 每个ASV/OTU中预测的EC数量;
- intermediate - 目录包括MinPath中间文件(minpath_running)，用序列取代流程的文件(包括JPLACE文件: intermediate/place_seqs/epa_out/epa_result.jplace).
- KO_predicted.tsv - 和EC_predicted.tsv类似, 每个ASV/OTU中预测的KO数量; 为KO 预测中间文件.
- marker_nsti_predicted.tsv - 16S预测的拷贝数和NSTI
- out.tre - 参考序列的树文件，这个树应该比你自己建的树更专业