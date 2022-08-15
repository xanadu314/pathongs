#!/bin/bash
# USAGE: to produce SE-m6Aseq.sh
# Take 20211211TC1-plaB_re data as example
# LAST MODIFIED: 2021.12.31
# Update for split forward and reverse 
# Update for one by one
# Update for bt2_ribo and bt2_mitochondria remove
# update for samtools view command -o 

set -euo pipefail

TOTAL_DIR=`pwd`
Rawdata="${TOTAL_DIR}/Rawdata"
CLEANDATA="${TOTAL_DIR}/CLEANDATA"
MAPPING_SAM="${TOTAL_DIR}/MAPPING_SAM"
MAPPING_BAM="${TOTAL_DIR}/MAPPING_BAM"
FINAL_BAM="${TOTAL_DIR}/FINAL_BAM"

STAT="${TOTAL_DIR}/STAT"
LOG="${TOTAL_DIR}/LOG"
BW="${TOTAL_DIR}/BW"
SCRIPT="${TOTAL_DIR}/SCRIPT"

STAT_QC="${STAT}/QC"
STAT_MAPPING="${STAT}/MAPPING"
RIBO="${STAT_MAPPING}/RIBO"
MITO="${STAT_MAPPING}/MITO"
GENOME="${STAT_MAPPING}/GENOME"
SPIKEIN="${STAT_MAPPING}/SPIKEIN"

ht2_mm10="/disk1/home/user_04/reference/mm10/mm10"
ht2_ribo="/disk1/home/user_04/reference/mm10/ribo"
ht2_mito="/disk1/home/user_04/reference/mm10/mito"
spike_index_pos='/disk1/home/user_04/reference/sn/positive'
spike_index_neg='/disk1/home/user_04/reference/sn/negative'

THREADS='40'

mkdir -p ${CLEANDATA} ${MAPPING_SAM} ${MAPPING_BAM} ${FINAL_BAM} ${RIBO} ${MITO} ${GENOME} ${SPIKEIN} ${STAT_QC} ${BW} ${SCRIPT} ${LOG} 

cd ${TOTAL_DIR}

# change name to _R2.fq.gz
cd ${Rawdata}
for file in *.fq*
# p5_IP_rep1_R2.fq.gz
do 
mv --no-clobber ${file} $(echo ${file} | sed 's/-/_/g;s/.R2./_R2./g;s/.R1./_R1./g')
# mv ${file} `echo ${file} | sed 's/_1/R1/g;s/fq.gz/fq/g'`
done

# one by one write script and submit 

```{bash
for R2 in ${Rawdata}/*_R2.fq.gz
do 
  SAMPLE_NAME="$(basename ${R2} _R2.fq.gz)"

  echo '#!/bin/bash' > ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "set -euo pipefail" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh 
  MAPPING_RESULT=${MAPPING_BAM}/${SAMPLE_NAME}
  echo "mkdir -p ${MAPPING_RESULT}">> ${SCRIPT}/${SAMPLE_NAME}_run.sh 

  echo "echo Start collecting informations and split bam at "'`date`'"!" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "samtools idxstats -@ ${THREADS} ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.MAPQ30.bam > ${STAT_MAPPING}/${SAMPLE_NAME}_mm10.MAPQ30_idxstats.txt &" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "samtools flagstat -@ ${THREADS} ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.MAPQ30.bam > ${STAT_MAPPING}/${SAMPLE_NAME}_mm10.MAPQ30_flagstat.txt &" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "(samtools view -bhS -@ 40 -f 16 ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.MAPQ30.bam -o ${FINAL_BAM}/${SAMPLE_NAME}.rev.bam)> ${LOG}/${SAMPLE_NAME}.revbam.log 2>&1" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "(samtools view -bhS -@ 40 -F 16 ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.MAPQ30.bam -o ${FINAL_BAM}/${SAMPLE_NAME}.fwd.bam)> ${LOG}/${SAMPLE_NAME}.fwdbam.log 2>&1" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "echo Finish collecting informations and split bam at "'`date`'"!" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh


  echo "echo Start converting bam to BW at "'`date`'"!" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "samtools index -@ 40 ${FINAL_BAM}/${SAMPLE_NAME}.rev.bam" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "samtools index -@ 40 ${FINAL_BAM}/${SAMPLE_NAME}.fwd.bam" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "bamCoverage --bam ${FINAL_BAM}/${SAMPLE_NAME}.rev.bam --outFileName ${BW}/${SAMPLE_NAME}.rev.bw --binSize 5 --normalizeUsing CPM --numberOfProcessors 15 > ${LOG}/${SAMPLE_NAME}.bamCoverage.rev.log 2>&1 &" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "bamCoverage --bam ${FINAL_BAM}/${SAMPLE_NAME}.fwd.bam --outFileName ${BW}/${SAMPLE_NAME}.fwd.bw --binSize 5 --normalizeUsing CPM --numberOfProcessors 15 > ${LOG}/${SAMPLE_NAME}.bamCoverage.fwd.log 2>&1 &" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "echo Finish converting bam to BW at "'`date`'"!" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
```






# remove no use 
  echo "rm -rf ${CLEANDATA}/${SAMPLE_NAME}_filtered_R2.fq.gz ${CLEANDATA}/${SAMPLE_NAME}_trimmed_R2.fq.gz" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  echo "touch ${TOTAL_DIR}/${SAMPLE_NAME}."'$(date '+%H:%M:%S-%b-%d-%Y')'".finish" >> ${SCRIPT}/${SAMPLE_NAME}_run.sh
  nohup sh ${SCRIPT}/${SAMPLE_NAME}_run.sh ${R2} > ${LOG}/${SAMPLE_NAME}_general.log &
done



```{bash}
#!/bin/bash

#SBATCH -J EPI
#SBATCH -p fat4way
#SBATCH -N 1
#SBATCH -c 20
#SBATCH -o /lustre2/junliu_pkuhpc/WangXW/20211220_TC1_m6A/ALL_batch%j.out
#SBATCH -e /lustre2/junliu_pkuhpc/WangXW/20211220_TC1_m6A/ALL_batch%j.err
#SBATCH --no-requeue
#SBATCH -A junliu_g1
#SBATCH --qos=junliucnl

#------# #------# #------# #------# #------# 
TOTAL_DIR="/lustre2/junliu_pkuhpc/WangXW/20211220_TC1_m6A"
Rawdata="${TOTAL_DIR}/Rawdata"
GENERAL_LOG="${TOTAL_DIR}/GENERAL_LOG"
SCRIPT="${TOTAL_DIR}/SCRIPT"
mkdir -p ${GENERAL_LOG} ${SCRIPT}

#------# #------# #------# #------# #------# 
#------# #------# #------# #------# #------# 
cd ${TOTAL_DIR}

for R2 in ${Rawdata}/*_R2.fq.gz
do SAMPLE_NAME="$(basename ${R2} _R2.fq.gz)"
echo '#!/bin/bash'> ${SCRIPT}/${SAMPLE_NAME}.batch
echo "#SBATCH -J ${SAMPLE_NAME}" >> ${SCRIPT}/${SAMPLE_NAME}.batch
echo "#SBATCH -p cn-long" >>${SCRIPT}/${SAMPLE_NAME}.batch
echo "#SBATCH -N 1" >>${SCRIPT}/${SAMPLE_NAME}.batch
echo "#SBATCH -c 20" >>${SCRIPT}/${SAMPLE_NAME}.batch
echo "#SBATCH -o ${GENERAL_LOG}/${SAMPLE_NAME}.general.out" >>${SCRIPT}/${SAMPLE_NAME}.batch
echo "#SBATCH -e ${GENERAL_LOG}/${SAMPLE_NAME}.general.err" >>${SCRIPT}/${SAMPLE_NAME}.batch
echo "#SBATCH --no-requeue" >>${SCRIPT}/${SAMPLE_NAME}.batch
echo "#SBATCH -A junliu_g1" >>${SCRIPT}/${SAMPLE_NAME}.batch
echo "#SBATCH --qos=junliucnl" >>${SCRIPT}/${SAMPLE_NAME}.batch
echo "sh /lustre2/junliu_pkuhpc/WangXW/20211220_TC1_m6A/test ${R2}" >>${SCRIPT}/${SAMPLE_NAME}.batch
sbatch ${SCRIPT}/${SAMPLE_NAME}.batch
done
```

```{bash}
#!/bin/bash

# set several traps
set -euo pipefail

#------# #------# #------# #------# #------#
# Liulab m6A-seq upstream V4
# Wang, Xiaowen (vivianwang719@gmail.com)
# LAST MODIFIED: Sun Jan  2 14:22:59 CST 2022
# Update for spike-in then mm10, -f rev -F fwd
# USAGE: sh PE-m6Aseq.sh pwd_of_sample
# Take 20211126_UV_mESC_m6A data as example

#------# #------# #------# #------# #------#
# Locations require to change (based to different computers)

SAMTOOLS="/home/junliu_pkuhpc/miniconda3/envs/epi/bin/${SAMTOOLS}"
HISAT2="/home/junliu_pkuhpc/miniconda3/envs/epi/bin/hisat2"
BAMCOVERAGE="/home/junliu_pkuhpc/miniconda3/envs/epi/bin/bamCoverage"
FASTQC="/home/junliu_pkuhpc/miniconda3/envs/epi/bin/fastqc"
FASTP="/home/junliu_pkuhpc/miniconda3/envs/epi/bin/fastp"
CUTADAPT="/home/junliu_pkuhpc/miniconda3/envs/epi/bin/cutadapt"

ht2_mm10="/gpfs1/junliu_pkuhpc/WangXW/reference/mm10/mm10"
ht2_ribo="/gpfs1/junliu_pkuhpc/WangXW/reference/mm10/ribo"
ht2_mito="/gpfs1/junliu_pkuhpc/WangXW/reference/mm10/mito"
spike_index_pos="/gpfs1/junliu_pkuhpc/WangXW/reference/sn/positive"
spike_index_neg="/gpfs1/junliu_pkuhpc/WangXW/reference/sn/negative"
THREADS="20"

#------# #------# #------# #------# #------#
# Assign the I/O variables
R2=${1}
R1=$(echo ${R2} | sed 's/R2/R1/g')
SAMPLE_NAME=$(basename ${R2} _R2.fq.gz)
TOTAL_DIR=$(echo ${R2} | sed 's/\(.*\)\/Rawdata\/.*/\1/g')

Rawdata="${TOTAL_DIR}/Rawdata"
CLEANDATA="${TOTAL_DIR}/CLEANDATA"
MAPPING_SAM="${TOTAL_DIR}/MAPPING_SAM"
MAPPING_BAM="${TOTAL_DIR}/MAPPING_BAM"
FINAL_BAM="${TOTAL_DIR}/FINAL_BAM"
STAT="${TOTAL_DIR}/STAT"
LOG="${TOTAL_DIR}/LOG"
BW="${TOTAL_DIR}/BW"
SCRIPT="${TOTAL_DIR}/SCRIPT"

MAPPING_RESULT="${MAPPING_BAM}/${SAMPLE_NAME}"
CLEANDATA_RESULT="${CLEANDATA}/${SAMPLE_NAME}"
STAT_QC="${STAT}/QC"
STAT_MAPPING="${STAT}/MAPPING"
STAT_RM="${STAT}/RM"
RIBO="${STAT_MAPPING}/RIBO"
MITO="${STAT_MAPPING}/MITO"
GENOME="${STAT_MAPPING}/GENOME"
SPIKEIN="${STAT_MAPPING}/SPIKEIN"


#------# #------# #------# #------# #------#
if [ ! -d ${TOTAL_DIR} ]; then mkdir -p ${TOTAL_DIR}; fi
if [ ! -d ${CLEANDATA} ]; then mkdir -p ${CLEANDATA}; fi
if [ ! -d ${MAPPING_SAM} ]; then mkdir -p ${MAPPING_SAM}; fi
if [ ! -d ${MAPPING_BAM} ]; then mkdir -p ${MAPPING_BAM}; fi
if [ ! -d ${FINAL_BAM} ]; then mkdir -p ${FINAL_BAM}; fi

if [ ! -d ${STAT} ]; then mkdir -p ${STAT}; fi
if [ ! -d ${LOG} ]; then mkdir -p ${LOG}; fi
if [ ! -d ${BW} ]; then mkdir -p ${BW}; fi
if [ ! -d ${SCRIPT} ]; then mkdir -p ${SCRIPT}; fi
if [ ! -d ${MAPPING_RESULT} ]; then mkdir -p ${MAPPING_RESULT}; fi
if [ ! -d ${CLEANDATA_RESULT} ]; then mkdir -p ${CLEANDATA_RESULT}; fi

if [ ! -d ${STAT_QC} ]; then mkdir -p ${STAT_QC}; fi
if [ ! -d ${STAT_MAPPING} ]; then mkdir -p ${STAT_MAPPING}; fi

if [ ! -d ${RIBO} ]; then mkdir -p ${RIBO}; fi
if [ ! -d ${MITO} ]; then mkdir -p ${MITO}; fi
if [ ! -d ${GENOME} ]; then mkdir -p ${GENOME}; fi
if [ ! -d ${SPIKEIN} ]; then mkdir -p ${SPIKEIN}; fi
if [ ! -d ${STAT_RM} ]; then mkdir -p ${STAT_RM}; fi

#------# #------# #------# #------# #------#
# 00 QC
echo "Start QC at `date`!"
# fastqc
${FASTQC} -t ${THREADS} -o ${STAT_QC} ${R1} > /dev/null 2>&1 &
${FASTQC} -t ${THREADS} -o ${STAT_QC} ${R2} > /dev/null 2>&1 &
# fastp
${FASTP} -i ${R1} -I ${R2} -o ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R1.fq.gz -O ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R2.fq.gz \
         -l 90 -w 16 -j ${STAT_QC}/${SAMPLE_NAME}.fastp.json -h ${STAT_QC}/${SAMPLE_NAME}.fastp.html > ${LOG}/${SAMPLE_NAME}.fastp.log 2>&1
# cutadapt
${CUTADAPT} -j ${THREADS} --times 1 -e 0.1 -O 6 --quality-cutoff 6 -m 70 \
            -a AGATCGGAAGAGC -A AGATCGGAAGAGC -o ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R1.fq.gz -p ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R2.fq.gz \
            ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R1.fq.gz ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R2.fq.gz > ${LOG}/${SAMPLE_NAME}.cutadapt.log 2>&1
# refastqc
${FASTQC} -t ${THREADS} -o ${STAT_QC} ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R1.fq.gz > /dev/null 2>&1 &
${FASTQC} -t ${THREADS} -o ${STAT_QC} ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R2.fq.gz > /dev/null 2>&1 &
echo "Finish QC at `date`!"
echo
echo
#------# #------# #------# #------# #------#
# mapping
# mapping to ribo
echo "Start mapping to ribo at `date`!"
(${HISAT2} -p ${THREADS} -x ${ht2_ribo} --rna-strandness RF --summary-file ${RIBO}/${SAMPLE_NAME}.ribo.summary \
           -1 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R1.fq.gz -2 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R2.fq.gz \
           --un-conc-gz ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.gz \
           | ${SAMTOOLS} view -bhS -@ ${THREADS} -q 30 | ${SAMTOOLS} sort -@ ${THREADS} -o ${MAPPING_RESULT}/${SAMPLE_NAME}.ribo.bam)> ${LOG}/${SAMPLE_NAME}.mapping.ribo.log 2>&1
echo "Finish mapping to ribo at `date`!"

mv ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.1.gz ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.1.fq.gz 
mv ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.2.gz ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.2.fq.gz 

# mapping to mito
echo "Start mapping to mito at `date`!"
(${HISAT2} -p ${THREADS} -x ${ht2_mito} --rna-strandness RF --summary-file ${MITO}/${SAMPLE_NAME}.mito.summary \
           -1 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.1.fq.gz -2 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.2.fq.gz \
           --un-conc-gz ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.gz \
           | ${SAMTOOLS} view -bhS -@ ${THREADS} -q 30 | ${SAMTOOLS} sort -@ ${THREADS} -o ${MAPPING_RESULT}/${SAMPLE_NAME}.mito.bam)> ${LOG}/${SAMPLE_NAME}.mapping.mito.log 2>&1
echo "Finish mapping to mito at `date`!"

mv ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.1.gz ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.1.fq.gz 
mv ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.2.gz ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.2.fq.gz 

# mapping to mm10
echo "Start mapping to mm10 at `date`!"
(${HISAT2} -p ${THREADS} -x ${ht2_mm10} --rna-strandness RF --summary-file ${GENOME}/${SAMPLE_NAME}.mm10.summary \
           -1 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.1.fq.gz -2 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.2.fq.gz \
           | ${SAMTOOLS} view -bhS -@ ${THREADS} -q 30 | ${SAMTOOLS} sort -@ ${THREADS} -o ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.bam)> ${LOG}/${SAMPLE_NAME}.mapping.mm10.log 2>&1
echo "Finish mapping to mm10 at `date`!"

# mapping to spike-in
echo "Start mapping to spike-in at `date`!"
(${HISAT2} -p ${THREADS} -x ${spike_index_pos} --rna-strandness RF --summary-file ${SPIKEIN}/${SAMPLE_NAME}.spike_pos.summary \
           -1 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.1.fq.gz -2 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.2.fq.gz \
           | ${SAMTOOLS} view -bhS -@ ${THREADS} -q 30 | ${SAMTOOLS} sort -@ ${THREADS} -o ${MAPPING_RESULT}/${SAMPLE_NAME}.spike_pos.bam)> ${LOG}/${SAMPLE_NAME}.mapping.spike_pos.log 2>&1
(${HISAT2} -p ${THREADS} -x ${spike_index_neg} --rna-strandness RF --summary-file ${SPIKEIN}/${SAMPLE_NAME}.spike_neg.summary \
           -1 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.1.fq.gz -2 ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.2.fq.gz \
           | ${SAMTOOLS} view -bhS -@ ${THREADS} -q 30 | ${SAMTOOLS} sort -@ ${THREADS} -o ${MAPPING_RESULT}/${SAMPLE_NAME}.spike_neg.bam)> ${LOG}/${SAMPLE_NAME}.mapping.spike_neg.log 2>&1
echo "Finish mapping to spike-in at `date`!"
echo
echo
#------# #------# #------# #------# #------#
# collecting informations
echo "Start collecting informations and split bam at `date`!"

${SAMTOOLS} index -@ ${THREADS} ${MAPPING_RESULT}/${SAMPLE_NAME}.ribo.bam
${SAMTOOLS} index -@ ${THREADS} ${MAPPING_RESULT}/${SAMPLE_NAME}.mito.bam
${SAMTOOLS} index -@ ${THREADS} ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.bam
${SAMTOOLS} index -@ ${THREADS} ${MAPPING_RESULT}/${SAMPLE_NAME}.spike_pos.bam
${SAMTOOLS} index -@ ${THREADS} ${MAPPING_RESULT}/${SAMPLE_NAME}.spike_neg.bam
${SAMTOOLS} idxstats -@ ${THREADS} ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.bam > ${GENOME}/${SAMPLE_NAME}_MAPQ30_idxstats.txt &
${SAMTOOLS} flagstat -@ ${THREADS} ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.bam > ${GENOME}/${SAMPLE_NAME}_MAPQ30_flagstat.txt &
${SAMTOOLS} view -bhS -@ ${THREADS} -f 147 ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.bam -o ${FINAL_BAM}/${SAMPLE_NAME}.rev147.bam
${SAMTOOLS} view -bhS -@ ${THREADS} -f 99 ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.bam -o ${FINAL_BAM}/${SAMPLE_NAME}.rev99.bam
${SAMTOOLS} merge -@ ${THREADS} -f ${FINAL_BAM}/${SAMPLE_NAME}.rev.bam ${FINAL_BAM}/${SAMPLE_NAME}.rev99.bam ${FINAL_BAM}/${SAMPLE_NAME}.rev147.bam
${SAMTOOLS} view -bhS -@ ${THREADS} -f 83 ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.bam > ${FINAL_BAM}/${SAMPLE_NAME}.fwd83.bam
${SAMTOOLS} view -bhS -@ ${THREADS} -f 163 ${MAPPING_RESULT}/${SAMPLE_NAME}.mm10.bam > ${FINAL_BAM}/${SAMPLE_NAME}.fwd163.bam
${SAMTOOLS} merge -@ ${THREADS} -f ${FINAL_BAM}/${SAMPLE_NAME}.fwd.bam ${FINAL_BAM}/${SAMPLE_NAME}.fwd83.bam ${FINAL_BAM}/${SAMPLE_NAME}.fwd163.bam
echo "Finish collecting informations and split bam at `date`!"

#------# #------# #------# #------# #------#

echo "Start converting bam to BW at `date`!"
${SAMTOOLS} index -@ ${THREADS} ${FINAL_BAM}/${SAMPLE_NAME}.rev.bam
${SAMTOOLS} index -@ ${THREADS} ${FINAL_BAM}/${SAMPLE_NAME}.fwd.bam
${BAMCOVERAGE} --bam ${FINAL_BAM}/${SAMPLE_NAME}.rev.bam --outFileName ${BW}/${SAMPLE_NAME}.rev.bw --binSize 5 --normalizeUsing CPM --numberOfProcessors 15 > ${LOG}/${SAMPLE_NAME}.bamCoverage.rev.log 2>&1
${BAMCOVERAGE} --bam ${FINAL_BAM}/${SAMPLE_NAME}.fwd.bam --outFileName ${BW}/${SAMPLE_NAME}.fwd.bw --binSize 5 --normalizeUsing CPM --numberOfProcessors 15 > ${LOG}/${SAMPLE_NAME}.bamCoverage.fwd.log 2>&1
echo "Finish converting bam to BW at `date`!"
#------# #------# #------# #------# #------#
# remove no use
touch ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary

echo >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
(basename ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R1.fq.gz" "have" "reads" "num:) >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
expr $(cat ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R1.fq.gz | wc -l) / 4 >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
echo >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
(basename ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R2.fq.gz" "have" "reads" "num:) >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
expr $(cat ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R2.fq.gz | wc -l) / 4 >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
echo >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
(basename ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R1.fq.gz" "have" "reads" "num:) >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
expr $(cat ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R1.fq.gz | wc -l) / 4 >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
echo >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
(basename ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R2.fq.gz" "have" "reads" "num:) >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
expr $(cat ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R2.fq.gz | wc -l) / 4 >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
echo >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary

(basename ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.1.fq.gz" "have" "reads" "num:) >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
expr $(cat ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.1.fq.gz | wc -l) / 4 >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
echo >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
(basename ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.2.fq.gz" "have" "reads" "num:) >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
expr $(cat ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.2.fq.gz | wc -l) / 4 >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
echo >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
(basename ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.1.fq.gz" "have" "reads" "num:) >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
expr $(cat ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.1.fq.gz | wc -l) / 4 >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
echo >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
(basename ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.2.fq.gz" "have" "reads" "num:) >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary
expr $(cat ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.2.fq.gz | wc -l) / 4 >> ${STAT_RM}/${SAMPLE_NAME}.fqfiles.summary

rm -rf ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.1.fq.gz 
rm -rf ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmribo.2.fq.gz 
rm -rf ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.1.fq.gz 
rm -rf ${CLEANDATA_RESULT}/${SAMPLE_NAME}_rmmito.2.fq.gz 
rm -rf ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R1.fq.gz
rm -rf ${CLEANDATA_RESULT}/${SAMPLE_NAME}_trimmed_R2.fq.gz
rm -rf ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R1.fq.gz 
rm -rf ${CLEANDATA_RESULT}/${SAMPLE_NAME}_filtered_R2.fq.gz

#------# #------# #------# #------# #------#
# remove no use
# touch ${TOTAL_DIR}/${SAMPLE_NAME}.$(date '+%H:%M:%S-%b-%d-%Y').finish
touch ${TOTAL_DIR}/${SAMPLE_NAME}.finish
```

