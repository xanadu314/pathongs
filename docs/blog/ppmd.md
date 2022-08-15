# ppmd

定量转换出问题，先不考虑定量

```R
library(xcms)
library(faahKO)
library(RColorBrewer)
library(pander)
library(magrittr)
library(pheatmap)
library(SummarizedExperiment)

## Get the full path to the mzML files
mzMLpath <- dir(pattern = "mzML", full.names = TRUE,recursive = TRUE,path = "./WT")
## Create a phenodata data.frame
pd <- data.frame(sample_name = sub(basename(mzML), pattern = ".mzML",replacement = "", fixed = TRUE),sample_group = c(rep("WT", 3)),stringsAsFactors = FALSE)
raw_data <- readMSData(files = mzML, pdata = new("NAnnotatedDataFrame", pd),
                       mode = "onDisk")
raw_data <- readMSData(files = mzML, pdata = new("NAnnotatedDataFrame", pd),
                       mode = "onDisk")
```

## 原始数据转换

### Thermo raw data

```bash
mono /beegfs/home/hyx/genetv/proteomics/thermorawfileparser/ThermoRawFileParser.exe -i 20141024_liver_12299_A_01.raw -f 0 -s |grep -E '=|IONS' > 20141024_liver_12299_A_01.mgf &
# 后面优化打包成一个shell脚本
```

## 转换成enviGCMS格式

```R
#' Perform MS/MS pmd annotation for mgf file
#' @param file mgf file generated from MS/MS data
#' @param digitrt mass accuracy, integer
#' @param digitmz retention time accuracy, integer
#' @return csv file with a csv file with m/z and retention time of peaks
#' @export
library(MSnbase)
ppmdanno <- function(file,digitrt,digitmz,name){
  namemgf <- basename(file)
  sample <- MSnbase::readMgfData(file)
  prec <- MSnbase::precursorMz(sample)
  #mz <- MSnbase::mz(sample)
  #ins <- MSnbase::intensity(sample)
  charge<-MSnbase::precursorCharge(sample)
  pepmass<-round(prec*charge,digitmz)
  rt<-round(MSnbase::rtime(sample),digitrt)
  group<-paste0("M",pepmass,"T",rt)
  pepmassrt<-data.frame(group,pepmass,rt,stringsAsFactors = FALSE)
  
  pepmassrt<-rbind(data.frame(group="group",pepmass="pepmass",rt="rt"),pepmassrt)
  
  colnames(pepmassrt) <- c("","mz","rt")
  write.csv(pepmassrt,col.names = T,row.names = F,quote = F,paste0(name,".csv"))
}
```

