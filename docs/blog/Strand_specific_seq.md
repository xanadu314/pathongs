# 链特异性建库和测序

> 关于链特异性测序的若干问题，很久以前就以为自己想清楚了，但是每次提起它的时候又容易重新产生各种各样的小困惑。于是整理一下，以免以后再时不时犯迷糊。很多东西就是这样，你以为的明白并不是真的明白，一年前的明白和一年后的明白也不是同一个明白。我这么说，不知道你能明白还是不明白。

好像没有看到中文的文章完整的写链特异性建库的原理和参数，在这里总结一下。

## RNAseq的基本流程

![rnaseq](http://image.sciencenet.cn/home/201712/18/125123z27bmm6wrlqomomy.jpg)

二代测序需要把RNA破碎成小片段，然后将RNA转变成一条cDNA，这一步需要用到反转录酶 reverse transcriptase (RT) 才能用RNA作为模板合成DNA。

不论是转录还是反转录都需要引物。通常如果我们要mRNA，那就可以用oligo-dT作为RT的引物，但是用它有两个问题，第一个是只能反转录那些有A尾巴的RNA，第二个问题是RT不是一个**高度持续性**的聚合酶，可能让转录提前发生终止，造成的结果就是3'端要比5'端reads富集，这样就会使得后续定量分析带来bias。

另一种常用的引物称为**随机引物**，随机引物的好处是没有A尾巴的诸如ncRNA也被留下了，对于没有polyA的原核生物RNA测序也只能用随机引物，而且不会存在明显的3'端偏差。但是很多研究也发现，所谓的随机引物根本就不随机，**这也是测序结果中，通常前6个碱基的GC含量分布特别不均匀的原因**。这几个碱基GC含量均匀很可能不是接头或者barcode那些东西，其实是Illumina 测序RT这一步的random hexamer priming 造成的bias，很多人在处理数据的时候会把这几个碱基去掉，其实很多时候真多RNA-seq数据去不去掉基本什么影响，不过开头如果有低质量的碱基倒是应该去掉。

随后是第二条链合成，这一步用是DNA聚合酶，以刚才合成的第一条链作为模板。

接下来就是在序列两端加上接头，加接头一方面是为了让机器可以识别这些序列，把这些序列固定；二是为了让多个样品可以同时上机，平摊每个样品的测序价格。双端测序为了让read从两边开始延伸，也需要在两端有所需的引物。

所谓双端测序，因为很多时候read的长度要短于insert，为了增加覆盖度于是就想出了从insert两端同时测序的办法。使得测序深度增加的同时也能够用来判断isoform方向。

对于illumina数据，有一条5-3的universal adaptor；还有一条是3-5的indexed adatpor，这条引物含有特异常的barcode。需要说明的是，在双端测序中，如果insert 不是足够长，那么R1可能就会测到R2的引物，同时R2可能会测到R1引物的反向互补序列。

大概的意思就是下面两张图。

![125221eawbdzyetwn6n1y1.jpg (640×597) (sciencenet.cn)](http://image.sciencenet.cn/home/201712/18/125221eawbdzyetwn6n1y1.jpg)

加了接头以后进行PCR的扩增。扩增后就开始测序，测序的过程如下图所示。

![125249tpibynz9rbzv94by.jpg (640×480) (sciencenet.cn)](http://image.sciencenet.cn/home/201712/18/125249tpibynz9rbzv94by.jpg)

![125258vn8tk8ibpiftbplb.jpg (640×327) (sciencenet.cn)](http://image.sciencenet.cn/home/201712/18/125258vn8tk8ibpiftbplb.jpg)

测序的基本思想是机器识别四种碱基发出的不同颜色的荧光，根据不同荧光，判断碱基的种类。



## 链特异性测序

和普通的RNAseq不同，链特异性测序可以保留最初产生RNA的方向，普通建库方式为什么不行呢？因为传统建库方式通过两个接头的ligation把RNA已经变成了双链DNA，最后的文库中一部被测序的链对应正义链（sense strand），一部分被测序的链测是反义链。

链特异性建库方式有不止一种，对应到不同的软件又有不同的叫法，下面是几种称呼。**要记住的是dUTP 测序方式的名字是fr-firstrand，也是RF。**

![125516q0vp00ctfcabf03f.jpg (640×209) (sciencenet.cn)](http://image.sciencenet.cn/home/201712/18/125516q0vp00ctfcabf03f.jpg)

不同的链特异性建库的方法，后面分析所使用的参数是不完全相同的。

![125531kxcz2iib4cv6gb76.jpg (600×437) (sciencenet.cn)](http://image.sciencenet.cn/home/201712/18/125531kxcz2iib4cv6gb76.jpg)

最常见的链特异性测序有两种，一种是dUTP method，一种是RT method

![125606jasueae08emgjeve.jpg (640×445) (sciencenet.cn)](http://image.sciencenet.cn/home/201712/18/125606jasueae08emgjeve.jpg)

### dUTP method

首先利用随机引物合成RNA的一条cDNA链，在合成第二条链的时候用dUTP代替dTTP，加adaptor后用UDGase处理，将有U的第二条cDNA降解掉。这样剩下的链的序列和最初的mRNA的序列是互补的，也就是说，dUTP方式测序R1文件中read1的方向和基因的方向（正义链）是相反的，而R2文件中的read2方向和基因的方向是相同的。

### RT method

这种方法的本质上就是对mRNA先加上adatper再进行反转录，这样所有的就能保存序列信息都是5' adapter -> gene -> 3' adapter的顺序。因此能够进行链特异性建库。

## 常用比对软件的参数设置

- hisat2 --rna-= RF
- tophat --library-type option fr-firststrand
- htseq-count：–s reverse
- rsem：--forward-prob0
- trinity --SSlibtype RF

## 常用计数软件参数设置

[subread.sourceforge.net/featureCounts.html](http://subread.sourceforge.net/featureCounts.html)

Perform strand-specific read counting (use '-s 2' if reversely stranded):

```bash
featureCounts -s 1 -t exon -g gene_id -a annotation.gtf -o counts.txt mapping_results_SE.bam
```



[科学网—链特异RNA-seq数据不这么看就浪费了 | antisense上的lncRNA-seq - 王程扬的博文 (sciencenet.cn)](https://wap.sciencenet.cn/home.php?mod=space&uid=3372875&do=blog&quickforward=1&id=1090305)

