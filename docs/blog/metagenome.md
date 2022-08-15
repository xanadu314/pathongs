# 宏基因组 

整理gtdb的结果

awk -F '[\t,;]' '{print $1"\t"$5";"$6";"$7";"$8}'|awk -F '[\t,;, ]' '{if ($3=="f__") print $1"\t"$2 ;else if ($4=="g__") print $1"\t"$3;if ($5=="s__") print $1"\t"$4;else print $1"\t"$5 }'



cat hyf.bac120.summary.tsv |awk -F '[\t,;]' '{print $1"\t"$5";"$6";"$7";"$8}'|awk -F '[\t,;, ]' '{if ($3=="f__") print $1"\t"$2 ;else if ($4=="g__") print $1"\t"$3;if ($5=="s__") print $1"\t"$4;else print $1"\t"$5 }'



while read id; do  var1=${id};var2=`grep $id unclassified.names.dmp`;var3=${var1}${var2};echo $var3 >> test2.txt; done<test1.txt

while read id1 id2 id3 id4 id5 id6; do echo "sed -i -e 's/gcode.*/|kraken:taxid|${id2}| ${id5}/g' ${id1}.fsa"; done</beegfs/home/hyx/metagenome/huyongfei/99_genome_classify_wf/classify/know_tax2.head.txt

