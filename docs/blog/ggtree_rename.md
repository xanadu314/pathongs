# R语言ggtree：将进化树中的序列id改名

之前TBtools有这个功能，后来改为收费了。感谢Y叔在ggtree中提供了这个功能。

2022-01-19



不同软件建出来的树文件，序列命名可能有差异，对于空格和特殊字符串的处理也不尽相同。为了达到文章发表的标准，我们通常会手动更改树的label，但是对于分支较多的树，改起来非常麻烦。

## 确定是否给树定根

首先有一些技巧用来判断是有根树还是无根树。也可以自己手动通过增减括号来定根：

假设有下面三种形式的进化树：

[1] (a, b, c);    无根树

[2] (a, b, c):1;  有根树

[3] ((a,b),c);    有根树

除第一种形式的进化树外，其它两种形式的进化树都是有根树。R语言对有根树、无根树的判断是：

- 如果最外层大括号内**只有两个分枝**，即为有根树，如[3]；
- 如果最外层大括号内有三个或以上分枝，一般为无根树，如[1]；
- 但是，如果大括号外存在枝长参数，如[2]中的:1，这种情况认为三个分枝以一定的枝长连接到根上，为有根树。

得到自己的进化树文件：

```{bash}
(Synergus:0.1976902387,(((((Periclistus:0.1403183720,Synophromorpha:0.0325185390)93:0.0313182375,(Xestophanes:0.0275715134,(Diastrophus:0.0456139475,Gonaspis:0.1146402107)97:0.0603746476)86:0.0275523221)91:0.0396704245,Ibalia:0.1295291852)93:0.0678466304,(((Liposthenes_ker:0.0568838340,Rhodus:0.4243267334)73:0.0825510697,Plagiotrochus:0.0778290252)71:0.0457931797,Phanacis_2:0.1416544135)42:0.0142517743)48:0.0209026386,(((Liposthenes_gle:0.1641119081,((((Antistrophus:0.1098867540,Hedickiana:0.2313789580)73:0.0566918206,Neaylax:0.1747090949)53:0.0027850349,(Isocolus:0.0980216531,Aulacidea:0.1315344980)40:0.0147148853)54:0.0123010924,((Andricus:0.0479556214,Neuroterus:0.0392025403)95:0.0395094917,Biorhiza:0.0640188941)87:0.0159496082)20:0.0000025961)50:0.0194234721,((((Panteliella:0.0792235900,Diplolepis:0.3184402599)84:0.0461941800,Phanacis_1:0.1153410113)66:0.0099961323,(Eschatocerus:0.2548694740,Parnips:0.0000022831)64:0.0802390069)34:0.0241704495,((Barbotinia:0.0731026287,Aylax:0.0957869567)87:0.0269932737,Iraella:0.0390833327)95:0.0797807340)18:0.0000021284)23:0.0095262346,Timaspis:0.0585073936)19:0.0170106400)57:0.0526944283,(Ceroptres:0.1057541047,(Pediaspis:0.1932340906,Paramblynotus:0.1711455809)28:0.0000021043)48:0.0416999011);

```

## 生成改名的表格

整理出要更改的内容。例如这里的x是原名字，y是更改后的名字

| x                | у    |
| ---------------- | ---- |
| Aylax            | 1    |
| Barbotinia       | 2    |
| Iraella          | 3    |
| lbalia           | 4    |
| Eschatocerus     | 5    |
| [ Timaspis       | 6    |
| Liposthenes_ gle | 7    |
| Diastrophus      | 8    |
| Parnips          | 9    |
| Panteliella      | 10   |
| Synophromorpha   | 11   |

## 读取树文件

```R
library(treeio)

tree<-read.newick("ggtree_practice_aligned.fasta.treefile",
                  node.label = "support")
```

## 可视化

```R
ggtree(tree)+
  geom_tiplab()+
  xlim(NA,0.8)
```

## 读取表格文件

```{R}
df<-read.csv("pra.csv",header=T)
```

## 替换label内容

```R
tree1<-tree
tree1@phylo$tip.label<-
  df[match(tree1@phylo$tip.label,df$x),]$y
```

match(x,y):返回一个和x长度相同且和y中元素相等的向量不等则返回NA

## 添加其它信息

```{R}
tree1@phylo$node.label<-tree1@data$support
write.tree(tree1@phylo,file = "pra.nwk")
```

更多信息可以参考ggtree的data-book [Chapter 4 Phylogenetic Tree Visualization | Data Integration, Manipulation and Visualization of Phylogenetic Trees (yulab-smu.top)](http://yulab-smu.top/treedata-book/chapter4.html)