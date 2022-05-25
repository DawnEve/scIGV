# scIGV aims to view single cell peaks in a browser using IGV.js

As the document of igv.js is Not Good, the first thing is to try some key parameters and functions.


> igv.js: https://github.com/igvteam/igv.js

> examples: https://igv.org/web/release/2.12.6/examples/

> wiki: https://github.com/igvteam/igv.js/wiki



## lean IGV.js
```
## version
$ wget https://igv.org/web/release/2.12.2/dist/igv.min.js
```

### some key settings in tracks
```
tracks: [
	{
		"name": "Cluster1_NK",
		"url": "bam/down/BMMC_donor1_cluster1.down0.05.bam",
		"indexURL": "bam/down/BMMC_donor1_cluster1.down0.05.bam.bai",
		"format": "bam",
		//"type": "annotation",

		"color": "red", //set colors

		showAlignments: false,

		height:100, //total height: default 300
		coverageTrackHeight:100, //coverage height: default 50
		alignmentRowHeight:0, //height of reads, default 14 per reads;
	},
]
```


### Data server must support `range` header
refer this [issue](https://github.com/igvteam/igv.js/issues/1508)
```
$ python3 -m RangeHTTPServer 8890
Serving HTTP on 0.0.0.0 port 8890 (http://0.0.0.0:8890/) ...
```










## get sub bam from whole 10x bam 

### On server 193
```
# in R Seurat:
getwd()  #[1] "/docker/jinwf/wangjl/BMMC", # /data:/docker
outputRoot #[1] "demo/cluster/"

scObj@meta.data$cell=rownames(scObj@meta.data)
df1=scObj@meta.data[, c("seurat_clusters", "cell")]
head(df1)

writeLines( df1[which(df1$seurat_clusters==1),]$cell, paste0(outputRoot, "BMMC_cluster1.cid.txt"))
writeLines( df1[which(df1$seurat_clusters==3),]$cell, paste0(outputRoot, "BMMC_cluster3.cid.txt"))
writeLines( df1[which(df1$seurat_clusters==6),]$cell, paste0(outputRoot, "BMMC_cluster6.cid.txt"))


# in shell
$ cd /home/wangjl/data/BMMC/demo/bams/
$ samtools view -@ 50 -D CB:../cluster/BMMC_cluster1.cid.txt -o BMMC_donor1_cluster1.bam /data/rawdata/bmmmc/HealthyControl_1/frozen_bmmc_healthy_donor1_possorted_genome_bam.bam

$ samtools view -@ 100 -D CB:../cluster/BMMC_cluster3.cid.txt -o BMMC_donor1_cluster3.bam /data/rawdata/bmmmc/HealthyControl_1/frozen_bmmc_healthy_donor1_possorted_genome_bam.bam

$ samtools view -@ 100 -D CB:../cluster/BMMC_cluster6.cid.txt -o BMMC_donor1_cluster6.bam /data/rawdata/bmmmc/HealthyControl_1/frozen_bmmc_healthy_donor1_possorted_genome_bam.bam

$ samtools index -@ 20 BMMC_donor1_cluster6.bam

# trans from server to Y station
$ scp bams/BMMC_donor1_cluster6.bam* wangjl@y.biomooc.com:/home/wangjl/test/IGVjs_test/bam/
```


## On Y station
```
## down sample
$ cd /home/wangjl/test/IGVjs_test/bam/
$ mkdir down
$ samtools view -s 0.05 -b -@ 50 BMMC_donor1_cluster3.bam > down/BMMC_donor1_cluster3.down0.05.bam
$ samtools view -s 0.05 -b -@ 50 BMMC_donor1_cluster1.bam > down/BMMC_donor1_cluster1.down0.05.bam
$ samtools view -s 0.05 -b -@ 50 BMMC_donor1_cluster6.bam > down/BMMC_donor1_cluster6.down0.05.bam

$ samtools index down/BMMC_donor1_cluster3.down0.05.bam
$ samtools index down/BMMC_donor1_cluster1.down0.05.bam
$ samtools index down/BMMC_donor1_cluster6.down0.05.bam


## start server service
$ cd /home/wangjl/test/IGVjs_test/
$ python3 -m http.server 8890
```





## [deeptools](https://deeptools.readthedocs.io/en/develop/content/tools/bamCoverage.html), from bam to bw
```
not use --ignoreDuplicates --scaleFactor 0.5
must: --binSize 1


# v6 按照正负链生成2个bw文件 --filterRNAstrand forward or --filterRNAstrand reverse 
这个在 igv上看是反的，仔细看参数，是过滤掉 forward，那么剩下的是 reverse。
同理 filterRNAstrand reverse 得到的是 forward 链的reads。
$ bamCoverage --bam BMMC_donor1_cluster6.bam \
--outFileName bw/BMMC_donor1_cluster6.v6r.bigWig \
--outFileFormat bigwig \
--binSize 1 \
--filterRNAstrand forward \
-p 12

$ bamCoverage --bam BMMC_donor1_cluster6.bam \
--outFileName bw/BMMC_donor1_cluster6.v6f.bigWig \
--outFileFormat bigwig \
--binSize 1 \
--filterRNAstrand reverse \
-p 12
# 14:32-->14:44


整个文件，转为 bw 
$ bamCoverage --bam BMMC_donor1_cluster6.bam \
--outFileName bw/BMMC_donor1_cluster6.bin1.bigWig \
--outFileFormat bigwig \
--binSize 1 \
-p 12



## --extendReads 0
$ bamCoverage --bam BMMC_donor1_cluster6.bam \
--outFileName bw/BMMC_donor1_cluster6.bv2.bigWig \
--outFileFormat bigwig \
--binSize 1 \
--extendReads 0 \
-p 12
# check md5 same as bin1.bigWig;





$ bamCoverage --bam BMMC_donor1_cluster3.bam \
--outFileName bw/BMMC_donor1_cluster3.bigWig \
--outFileFormat bigwig \
-p 12 \


$ bamCoverage --bam BMMC_donor1_cluster1.bam \
--outFileName bw/BMMC_donor1_cluster1.bigWig \
--outFileFormat bigwig \
-p 12 \


$ bamCoverage --bam BMMC_donor1_cluster6.bam \
--outFileName bw/BMMC_donor1_cluster6.bigWig \
--outFileFormat bigwig \
-p 12 \









in IGV tracks:
// wig track
{
	type: "wig",
	name: "CTCF",
	url: "https://www.encodeproject.org/files/ENCFF356YES/@@download/ENCFF356YES.bigWig",
	min: "0",
	max: "30",
	color: "rgb(0, 0, 150)",
	guideLines: [
		{color: 'green', dotted: true, y: 25}, 
		{color: 'red', dotted: false, y: 5}
	]
},

{
	type: "wig",
	name: "Cluster6_mono-v4_1", //binSize 1, default:50
	url: "bam/bw/BMMC_donor1_cluster6.v4_1.bigWig",
	height:50,
	color: "orange", //支持颜色字符串，也支持 16进制颜色
	min: "0",
	//max: "30", //这个可能报错
	//color: "rgb(0, 0, 150)", //支持rgb颜色模式
	guideLines: [
		{color: 'green', dotted: true, y: 25}, 
		{color: 'red', dotted: false, y: 5}
	]
},

先看EEF1D 向左的基因，forward 最大近6k，reverse 最大是1；点开看，大部分reads在 "-" 上，就是方向向左。
FTH1 是重叠基因区域
```








