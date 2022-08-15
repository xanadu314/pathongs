# Seqkit



---

[Seqkit]() 是由国人 [shenwei]() 开发，功能强大的命令行序列处理工具套件。

## 使用

| 命令            | 功能                                                         |
| --------------- | ------------------------------------------------------------ |
| amplicon        | 通过引物检索扩增子(或其周围的特定区域)                       |
| bam             | 检查和在线绘制BAM记录文件的直方图                            |
| common          | 通过id/名称/序列查找多个文件的公共序列                       |
| concat          | 连接多个文件中具有相同ID的序列                               |
| convert         | 转换FASTQ质量编码格式：支持格式包括：桑格，Solexa和Illumina  |
| duplicate       | 重复序列N次                                                  |
| faidx           | 创建FASTA索引文件并提取子序列                                |
| fish            | 使用局部比对在较大的序列中寻找短序列                         |
| fq2fa           | 转换FASTQ到FASTA                                             |
| fx2tab          | 将FASTA/Q转换为表格格式(包含长度/GC含量/GC偏好)              |
| genautocomplete | 生成shell自动完成脚本                                        |
| grep            | 通过ID/name/sequence/sequence motif搜索序列，允许错配        |
| head            | 打印第一条序列                                               |
| help            | 打印帮助信息                                                 |
| locate          | 定位序列，或者motifs，允许错配                               |
| mutate          | 编辑序列(点突变、插入、删除)                                 |
| pair            | 匹配双端序列文件                                             |
| range           | 打印一个范围内的序列                                         |
| rename          | 重命名重复序列ID                                             |
| replace         | 使用正则表达式修改名称或者序列                               |
| restart         | 重置环状基因组的起始位置                                     |
| rmdup           | 通过id/名称/序列删除重复的序列                               |
| sample          | 按数量或比例对序列进行抽样                                   |
| sana            | 清理损坏的单行fastq文件                                      |
| scat            | real time recursive concatenation and streaming of fastx files |
| seq             | 转换序列(反向，补充，提取ID…)                                |
| shuffle         | 随机序列                                                     |
| sliding         | 序列滑窗提取，支持环形基因组                                 |
| sort            | 按id/名称/序列/长度排序序列                                  |
| split           | 按id/seq区域/大小/部件将序列拆分为文件(主要用于FASTA)        |
| split2          | 按序列数量/文件数将序列拆分为多个文件(FASTA                  |
| stats           | FASTA/Q文件的简单统计                                        |
| subseq          | 通过region/gtf/bed得到子序列，包括侧翼序列                   |
| tab2fx          | 转换表格格式为FASTA/Q格式                                    |
| translate       | 翻译DNA/RNA到蛋白质序列(支持歧义碱基)                        |
| version         | 打印版本信息并检查是否更新                                   |
| watch           | 序列特征的监测和在线直方图                                   |

### grep 命令

spades 组装完毕的序列，提取其中某个 node 或者某些 nodes 序列：

```bash
$ seqkit grep -r -p ^node_1$ scaffolds.fasta > node_1.fasta
```

将 scaffolds 中所有 nodes 提取到单独的 fasta 文件中：

```bash
$ for i in $(awk '/>/' scaffolds.fasta | awk '{print $1}'); \
> do seqkit grep -p ${i:1} scaffolds.fasta > ${i:1}.fasta; \
> done
```

### seq 命令

```bash
$ seqkit seq -h

```

```bash

# tab="-n和-i" 
# 将 reads 的名称信息输出，如果要和 bioawk $name 相同，使用 -i 参数
$ seqkit seq -n SRR1175124_1.fastq.gz SRR1175124_2.fastq.gz
SRR1175124.102354 102354 length=150
$ bioawk -c fastx '{print $name}' SRR1175124_1.fastq.gz SRR1175124_2.fastq.gz
SRR1175124.102354
$ seqkit seq -in SRR1175124_1.fastq.gz SRR1175124_2.fastq.gz
SRR1175124.102354

```

```bash
# 输出小写碱基
$ seqkit seq -sl SRR1175124_1.fastq.gz
# 输出大写碱基
$ seqkit seq -su SRR1175124_1.fastq.gz
```




## 使用案例

```bash
# 下载 NC_001477
$ pefetch -db nuccore -id NC_001477 -format fasta > NC_001477.fasta
# 翻译 DNA 序列为氨基酸序列
$ seqkit translate -j 4 -o NC_001477.pep NC_001477.fasta
# 统计序列的长度和GC
$ seqkit fx2tab -l -g -n -i -H test.fa
```

!!! note "对注释的多拷贝基因提取序列后进行序列比对"

```bash
# 用prokka注释基因组拼接序列
$ prokka --rfam --prefix sample --outdir sample contig.fa
# 用 seqkit 提取多拷贝基因 ncRNA Qrr 的序列
$ grep 'Qrr' sample/sample.gff | for i in $(awk '{print $9}'); do seqkit grep -p ${i:3:14} sample.ffn >> qrr.fa; done
# mafft 序列比对
$ mafft qrr.fa > qrr.mafft
# raxml 快速构建进化树
$ raxmlHPC -f a -p 12345 -x 12345 -# 1000 -m GTRGAMMA -s qrr.mafft -n qrr
```
