

# Blast2SRA database

**2021.3.5**

The Basic Local Alignment Search Tool (*BLAST*) finds regions of local similarity between sequences

![blast](https://blast.ncbi.nlm.nih.gov/images/nucleutide-blast-cover.png)

---

## SRA数据库

Sequence Read Archive (*SRA*) 是NIH的**高通量测序数据**的主要档案，是国际核苷酸序列数据库协作（INSDC）的一部分。其作用是存储包括Illumina、454、IonTorrent、Complete Genomic、PacBio和OxfordNanpores在内的二代测序技术所产生的**原始**序列数据（GEO数据库是经过一定处理的数据）。

SRA数据库主要包含一下几个数据类型：

- EPR/SPR: studies(研究课题)
- SRS：Experiments(实验方案)
- SRX：Samples(样品信息)
- SRR：Runs(测序产生的reads)

### 在线访问SRA数据库

SRA数据库通常与GEO数据库相关联，可以通过GEO![GEO](https://www.ncbi.nlm.nih.gov/geo/)来访问SRA的相关信息。

![GEONCBI](/assets/images/blog/GEO.png)

点击即可以看到所有的分组信息

![GEOGROUP](/assets/images/blog/SRA_group.png)

我们可以根据分组，记下我们需要的Run编号（SRR开头）或者Experiment编号（SRX开头）

### 在线BLAST2SRA

访问NCBI web BLAST页面 https://blast.ncbi.nlm.nih.gov/Blast.cgi

输入需要比对的基因序列，下面选择SRA数据库，再输入SRX编号，根据提示填充所需要的信息。

![blast2sra](/assets/images/blog/blast2sra.png)

点BLAST即可获得比对结果。比对到的数量近似反映出测序测到的reads数目。

![results](//assets/images/blog/results.png)