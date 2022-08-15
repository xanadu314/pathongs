# Tidyverse

Tidyverse是由R语言大神Hadley Wickham开发的一系列用于数据科学的R语言包集合，所有包共享底层设计和语法。包括的包主要有ggplot2，dplyr，tidyr，readr，purrr，tibble，stringr, forcats。
Tidyverse涵盖了数据科学的整个工作流程。即数据导入、数据清洗、数据变换、数据可视化、数据建模以及文档沟通。



在这里写一个用tiduverse清洗数据并且批量计算相关系数的脚本

```{r}
df <- readr::read_table("~/Downloads/test/test2.txt") #读取文件
head(df)
																	BD1         BD2        BD5         BD4
89179a8e50bcef5dcb481a64dc60fa09 -1.15772834 -1.15772834  1.6476223 -0.94108515
3da54c8f56be3e7e66f56aadde22acef -0.19223394  2.79115524 -0.5764467  0.07717421
ce32f7075191611f2446847cd2a77e9e -0.94197732  3.26792574 -0.3191527  0.50023456
ed87db9b9b7e35cb8e37a22bfdf295ba  1.07772642  0.06934341 -0.3622937  1.98773060
1eba3d8906f7ab196667dffcd2c92952 -0.05149243 -0.19716533 -0.5813576 -0.92552982
7df6d1288abb65b112af91bb7e985c2c -0.58046993 -0.58046993 -0.2225874 -0.48351371
                                        BD6        BD3
89179a8e50bcef5dcb481a64dc60fa09 -0.9675704  0.5819640
3da54c8f56be3e7e66f56aadde22acef  1.2810919  1.1073543
ce32f7075191611f2446847cd2a77e9e  0.5604836  1.0658226
ed87db9b9b7e35cb8e37a22bfdf295ba  1.1256861 -1.1062836
1eba3d8906f7ab196667dffcd2c92952 -0.7686513  0.6544608
7df6d1288abb65b112af91bb7e985c2c -0.2639364 -0.3894091
                                                             name
89179a8e50bcef5dcb481a64dc60fa09 89179a8e50bcef5dcb481a64dc60fa09
3da54c8f56be3e7e66f56aadde22acef 3da54c8f56be3e7e66f56aadde22acef
ce32f7075191611f2446847cd2a77e9e ce32f7075191611f2446847cd2a77e9e
ed87db9b9b7e35cb8e37a22bfdf295ba ed87db9b9b7e35cb8e37a22bfdf295ba
1eba3d8906f7ab196667dffcd2c92952 1eba3d8906f7ab196667dffcd2c92952
7df6d1288abb65b112af91bb7e985c2c 7df6d1288abb65b112af91bb7e985c2c
df3 <- melt(df,id.vars="name") #长宽转换
                             name variable       value
1 89179a8e50bcef5dcb481a64dc60fa09      BD1 -1.15772834
2 3da54c8f56be3e7e66f56aadde22acef      BD1 -0.19223394
3 ce32f7075191611f2446847cd2a77e9e      BD1 -0.94197732
4 ed87db9b9b7e35cb8e37a22bfdf295ba      BD1  1.07772642
5 1eba3d8906f7ab196667dffcd2c92952      BD1 -0.05149243
6 7df6d1288abb65b112af91bb7e985c2c      BD1 -0.58046993

models <- df3 %>% split(.$name) %>% map(~lm(value ~ c(1,2,3,4,5,6), data =. )) #批量计算与c(1,2,3,4,5,6)的相关系数
coeff <- models %>% map(summary) %>% map_dbl(~.$r.squared) #批量提取相关系数

plots <- mtcars %>% split(.$cyl) %>% 
  map(~ ggplot(data = ., aes(x = mpg, y = wt)) + geom_point())
paths <- stringr::str_c(names(plots), ".pdf")
pwalk(list(paths, plots), ggsave, path = "~/Downloads/test/")#用pwalk批量保存图片

```

