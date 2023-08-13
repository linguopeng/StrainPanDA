# **StrainPanDA**

# **教程**

**iMeta:**  **利用StrainPanDA从宏基因组中同时提取共存菌株的组成和基因成分谱的入门教程**

## **写在前面**

StrainPanDA主要针对宏基因组纵向数据，能同时实现共存菌株的组成和基因成分谱的获取，方便用户快速建立功能和组成之间的关联假设。

本教程主要从安装、数据库准备，运行和输出介绍四个方面介绍如何使用StrainPanDA进行分析。

全文翻译和视频简介见：《[iMeta | 深圳先进院戴磊组开发可同时提取共存菌株的组成和基因成分谱的菌株分析工具](https://mp.weixin.qq.com/s?__biz=MzU2NDY5MjIyMg==&mid=2247485040&idx=1&sn=3c421ac04326791a853354cd76dc5677&chksm=fc4653e1cb31daf7497246d68319ceca4f626e14bd4caaa2f6778ee97a9a2c6138e2ddfcc25e&scene=178&cur_album_id=2140899408157147136#rd)》

中文视频教程见：[iMeta视频教程 | StrainPanDA分析宏基因组共存菌株的组成和基因成分谱](https://mp.weixin.qq.com/s/zNnwZqhnCaA4j-kxjKOjOg)

欢迎大家添加微信huhan_2008，备注StrainPanDA包加入菌株分析用户交流群。


## **安装**

### 1. 安装流程管理工具[Nextflow](https://www.nextflow.io/)，并将nextflow可执行文件添加到$PATH中
```
curl -s https://get.nextflow.io | bash
mv nextflow <YOUR_PATH> # 请确保<YOUR_PATH>在$PATH中。
which nextflow # 显示<YOUR_PATH>, 如果没有显示，则需要重新检查mv操作 
```

注：如果遇到Java版本的问题，请具体看[Nextflow](https://www.nextflow.io/)的官方安装指南。

### 2. 获取StrainPanDA代码

```
cd <PATH_TO_PANDA> # <PATH_TO_PANDA>用于放置StrainPanDA代码的路径
git clone https://github.com/xbiome/StrainPanDA.git 
```

### 3. 安装StrainPanDA组件

**使用** [**docker**](https://docs.docker.com/engine/install/ubuntu/) **进行安装** （推荐方式）：

注：如果国内使用docker下拉镜像速度慢，可以通过[配置镜像加速](https://blog.csdn.net/u014641168/article/details/124651721)来提速。

第一步：获取PanPhlAn（用于将测序数据比对到泛基因组数据库）

```
docker pull yuxiangtan/strainpanda-mapping:dev
docker tag yuxiangtan/strainpanda-mapping:dev strainpanda-mapping:dev 
```

第二步：获取strainpandar (将比对的结果分解为菌株-样本矩阵和基因家族-菌株矩阵)

```
docker pull yuxiangtan/strainpanda-strainpandar:dev
docker tag yuxiangtan/strainpanda-strainpandar:dev strainpanda-strainpandar:dev 
```

第三步：docker容器可用性测试：（确保两个docker镜像已经被完整拉取下来）

```
docker run -u $(id -u):$(id -g) strainpanda-mapping:dev panphlan_profile.py -h
docker run -u $(id -u):$(id -g) strainpanda-strainpandar:dev R --no-save 
```

如果镜像拉取仍有问题，请看[常见问题](https://github.com/xbiome/StrainPanDA/blob/main/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98.md#%E6%97%A0%E6%B3%95%E6%8B%89%E5%8F%96%E9%95%9C%E5%83%8F)

**备选本地安装方式** （建议仅在无法使用docker时使用）

第一步：参考以下链接安装PanPhlAn（用于将测序数据比对到范基因组数据库）

[https://github.com/SegataLab/panphlan/wiki/Home-1_3](https://github.com/SegataLab/panphlan/wiki/Home-1_3)

第二步：安装[R](https://www.r-project.org/)及[strainpandar](https://github.com/xbiome/StrainPanDA/blob/main/src/strainpandar)（用于将比对的结果分解为菌株）

安装下列依赖的R包：`dplyr`, `foreach`, `MASS`, `NMF`, `pracma`, `vegan`,`getopt`,`data.table`,`R.utils`

在命令行运行下列命令安装strainpandar：

```
cd <PATH_TO_PANDA>/StrainPanDA/src
tar -czf strainpandar.tar.gz strainpandar
R CMD INSTALL strainpandar.tar.gz 
```

## **数据库准备**

数据库的文件布局需要遵循这样的规范：文件夹(<PATH_TO_REFERENCE>)需要以版本数(示例为202009)结尾，如ref202009，否则会出现“index is out of range 0..-1 (index = 0)“的报错。每个菌种版本（例如Escherichia-coli-202009）对应一个子文件夹。具体示例如下：

```
ref202009
 ├── Acinetobacter-johnsonii-202009
 └── Escherichia-coli-202009 
```

目前已经建好的泛基因组数据库可以使用wget 等下载工具从Zenodo (doi:10.5281/zenodo.6592017)下载：[StrainPanDA Pre-built pangenome database | Zenodo](https://zenodo.org/record/6592017)

Zenodo里面每一个tar.gz文件代表一个菌种，其格式为：菌种名-版本号.tar.gz (例如Escherichia-coli-202009.tar.gz里Escherichia-coli是菌种名，202009是版本号)

使用前，确保需要分析的菌种的tar.gz文件已经解压并位于数据库文件夹（<PATH_TO_REFERENCE>）下：

```
cd <PATH_TO_REFERENCE>
wget https://zenodo.org/record/6592017/files/Escherichia-coli-202009.tar.gz
tar -zxvf Escherichia-coli-202009.tar.gz #解压Escherichia-coli-202009的数据库
# tar -zxvf *.tar.gz #解压所有的菌种 
```

每个菌种子文件夹里主要有下面几部分：

- 泛基因组文件是StrainPanDA主要的使用文件： ${species_version}_pangenome.csv ，其中"pangenome.csv"是被StrainPanDA自动识别的部分；

- 泛基因组的序列可以在 ${species_version}_centroids.ffn 里找到；

- bowtie2索引文件也是StrainPanDA主要的使用文件：以"bt2" 结尾;

- 泛基因组的基因功能注释在 ${species_version}_emapper_anno.tsv 文件；

- 泛基因组的毒力因子注释（基于VFDB，Apr 9th 2021） 在${species_version}_vfdb_anno.csv文件里，其中第一列是基因家族ID，第二列是VFDB的ID；

- 泛基因组的碳水化合物活性酶注释在 ${species_version}_CAZy_anno.csv文件，基于CAZy (Jul 31st 2019)，其中第一列是基因家族ID，第二列是CAZy的ID。

对于未建库的菌种，或者需要对已有菌种更新重建时，可参考以下中文教程进行重建[StrainPanDA/中文README.md at main · xbiome/StrainPanDA (github.com)](https://github.com/xbiome/StrainPanDA/blob/main/custom_db/%E4%B8%AD%E6%96%87README.md)

如果需要分析自有菌株，请参考[常见问题](https://github.com/xbiome/StrainPanDA/blob/main/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98.md#%E5%A6%82%E4%BD%95%E5%B0%86%E8%87%AA%E6%9C%89%E8%8F%8C%E6%A0%AA%E8%87%AA%E5%B7%B1%E8%AF%95%E9%AA%8C%E6%89%80%E7%94%A8%E7%9A%84%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%AD%E6%B2%A1%E6%9C%89%E7%9A%84%E8%8F%8C%E6%A0%AA%E5%8A%A0%E5%85%A5%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%AD%E8%BF%9B%E8%A1%8C%E5%88%86%E6%9E%90)

## **运行**

### **运行完整分析**

下载测试数据集到文件夹（以<PATH_TO_FASTQ>指代)，测试数据可以从[该链接](https://zenodo.org/record/6997716#.YyAxU6TiuUk)下载，是20个使用E. coli基因组进行模拟的样品。

注意：输入文件后缀可以是1或2接着.fq,.fastq或者.fq.gz, .fastq.gz的任意一种。但是必须是phred33质量分系统的数据。（以前旧的数据可能是phred64系统，PanPhlAN流程会报错）

StrainPanDA需要最少5个样品才能正常运行。如果要得到比较可靠的结果，建议最少提供20个符合假设（这些样品中具有共同菌株或亚种）的关联样品。

创建菌种列表（列表菌种必须对应数据库下面的菌种子目录，可以输入多行，每行一个菌种名，则StrainPanDA会依次分析每个菌种）

```
echo "Escherichia coli" > species_list.txt 
```

#### **使用 docker**

```
nextflow <PATH_TO_PANDA>/StrainPanDA/main.nf -profile docker \
 --ref_path <PATH_TO_REFERENCE> \
 --path <PATH_TO_FASTQ> \
 --ref_list species_list.txt 
```

#### **本地**

```
nextflow <PATH_TO_PANDA>/StrainPanDA/main.nf \
 --ref_path <PATH_TO_REFERENCE> \
 --path <PATH_TO_FASTQ> \
 --ref_list species_list.txt 
```

如果PanPhlAN运行报错，请参考[常见问题](https://github.com/xbiome/StrainPanDA/blob/main/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98.md#%E8%B7%91panphlan%E6%97%B6%E5%A4%B1%E8%B4%A5)

### **仅运行strainpandar(供调试用)**

假设你已经运行StrainPanDA，并获得了计数矩阵{species-version}.counts.csv（见[示例](https://github.com/xbiome/StrainPanDA/blob/main/data/Faecalibacterium-prausnitzii-202009.counts.csv)）。如果你需要调试参数以获得更优的矩阵分解的结果，又不想重复泛基因组比对的步骤，我们提供一个独立的RScript脚本以执行strainpandar部分。

#### **基于 docker**：

```
#进入docker的互动模式
 docker run --rm -t -i -u $(id -u):$(id -g) -v <PATH_TO_PANDA>/StrainPanDA/bin/:/script -v <PATH_TO_csv>:/data -v <PATH_TO_REFERENCE>:/ref -v $PWD:/work -w /work strainpanda-strainpandar:dev /bin/bash
 #在docker内运行R
 Rscript /script/run_strainpandar.r \
 -c /data/Escherichia-coli-202009.counts.csv \
 -r /ref/Escherichia-coli-202009 \
 -o work -t 8 -m 8 -n 0 
```

#### **基于本地安装** ：

```
Rscript <PATH_TO_PANDA>/StrainPanDA/bin/run_strainpandar.r \
 -c data/Escherichia-coli-202009.counts.csv \
 -r <PATH_TO_REFERENCE>/Escherichia-coli-202009 \
 -o work -t 8 -m 8 -n 0 
```

脚本的参数可以从帮助信息中获得

```
Rscript bin/run_strainpandar.r -h
 A wrapper script to perform strain decomposition using strainpandar package.
 Usage: bin/run_strainpandar.r [-[-help|h]] [-[-counts|c] <character>] [-[-reference|r] <character>] [-[-output|o] [<character>]] [-[-threads|t] [<integer>]] [-[-max_rank|m] [<integer>]] [-[-rank|n] [<integer>]]
 -h|--help Show this help message
 -c|--counts Gene-sample count matrix (CSV file) obtained from mapping reads to a reference pangenome [required]
 -r|--reference Pangenome database path [required]
 -o|--output Output prefix [default: ./strainpandar]
 -t|--threads Number of threads to run in parallele [default: 1]
 -m|--max_rank Max number of strains expected [default: 8]
 -n|--rank Number of strains expected. If 0, try to select from 1 to `max_rank`. If not 0, overwrite `max_rank`. [default: 0] 
```

**输出介绍**

输出目录中包含两个子目录:strainpanda_out与work。其中work是nextflow的工作目录，里面主要包含一些流程运行的log和状态信息，如果不使用nextflow 的resume功能，则可以忽略，或者直接rm -rf work删除。

菌株相关的结果位于strainpanda_out目录下面：

```
strainpanda_out
 ├── Escherichia-coli-202009.counts.csv
 ├── Escherichia-coli-202009_mapping
 │   ├── C01_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C02_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C03_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C04_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C05_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C06_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C07_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C08_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C09_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C10_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C11_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C12_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C13_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C14_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C15_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C16_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C17_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C18_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   ├── C19_errfree_r0_Escherichia-coli-202009.csv.bz2
 │   └── C20_errfree_r0_Escherichia-coli-202009.csv.bz2
 ├── Escherichia-coli-202009_strainpandar_out
 │   ├── Escherichia-coli-202009.strainpanda.anno_strain_sample.pdf
 │   ├── Escherichia-coli-202009.strainpanda.genefamily_strain.csv
 │   ├── Escherichia-coli-202009.strainpanda.genefamily_strain.pdf
 │   ├── Escherichia-coli-202009.strainpanda.rds
 │   ├── Escherichia-coli-202009.strainpanda.strain_sample.csv
 │   ├── Escherichia-coli-202009.strainpanda.strain_sample.pdf
 │   ├── Escherichia-coli-202009.strainpanda_all_dis.csv
 │   ├── Escherichia-coli-202009.strainpanda_all_neighbor.csv
 │   ├── Escherichia-coli-202009.strainpanda_str_anno_prof.csv
 │   ├── Escherichia-coli-202009.strainpanda_str_merged_prof.csv
 │   └── Escherichia-coli-202009.strainpanda_str_neighbor.csv
 └── pipeline_info
 ├── strainpanda_DAG.svg
 ├── strainpanda_report.html
 ├── strainpanda_timeline.html
 └── strainpanda_trace.txt 
```

1. 泛基因组的覆盖分布矩阵：{species-version}.counts.csv：每一行是一个基因家族，每一列是一个样品，数值是片段数。该文件由流程中的PanPhlAn产生，同时也是run_strainpandar.r的-c参数的输入。

2. 菌株解构后的矩阵：

2.1. 菌株基因成分谱（基因家族-菌株矩阵 **P** ）：{species-version}.strainpanda.genefamily_strain.csv：每一行是一个基因家族，每一列是一个菌株，数值是二进制（0代表没有，1代表该菌株有该家族）。

2.2 菌株组成矩阵（菌株-样品矩阵 **S** ）：{species-version}.strainpanda.strain_sample.csv：每一行是一个菌株，每一列是一个样品，数值是菌株在样品里的相对丰度。

要获得每个预测菌株的功能注释，可以通过菌株基因成分谱的ID和参考数据库里的注释文件进行匹配获取。参加 [常见问题](https://github.com/xbiome/StrainPanDA/blob/main/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98.md#%E5%A6%82%E4%BD%95%E8%8E%B7%E5%8F%96%E5%9F%BA%E5%9B%A0%E5%AE%B6%E6%97%8F%E7%9A%84%E5%8A%9F%E8%83%BD%E6%B3%A8%E9%87%8A)

**示例**

文章中使用的示例可以在[这里](https://github.com/xbiome/StrainPanDA-data/tree/main/example#readme)访问。

<br>

[常见问题](常见问题.md)


**引文**

Hu, Han, Yuxiang Tan,Chenhao Li, Junyu Chen, Yan Kou, Zhenjiang Zech Xu, Yang‐Yu Liu, Yan Tan, and Lei Dai. 2022. "StrainPanDA: Linked reconstruction of strain composition and gene content profiles via pangenome‐based decomposition of metagenomic data." iMeta. e41. https://doi.org/10.1002/imt2.41
