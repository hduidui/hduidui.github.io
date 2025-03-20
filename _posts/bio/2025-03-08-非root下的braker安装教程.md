---
title: 非root用户容器安装braker过程 
author: hduidui
date: 2025-02-20 21:00:00 +0800
categories: [bio,install]
math: true
tags: [生信,安装教程,braker]     # TAG names should always be lowercase非
---

## 安装容器

安装singularity容器和构建sif文件

braker3在docker网站的地址是https://hub.docker.com/r/teambraker/braker3

```shell
# singularity容器安装方法
conda install conda-forge::singularity

#拉取braker3的docker镜像，构建sif，sif文件是singularity的镜像文件，相当于把整个braker3的环境都打包到这个sif文件里
singularity build braker3.sif docker://teambraker/braker3:latest
```

## 写shell脚本

```shell
# 设置临时的环境变量BRAKER_SIF，这样就可以在任何地方运行braker3.sif
export BRAKER_SIF=/public/home/huangdy/braker3.sif

# SINGULARITY_BIND是固定的写法，是braker3的一个参数，不可以改为其他的名字，这个用于绑定容器(singularity)和宿主机(centos)之间的文件映射。多个路径用逗号分开，映射格式为:
#    宿主机的文件路径:容器的文件路径,宿主机的文件路径:容器的文件路径,宿主机的文件路径:容器的文件路径

# 由于singularity下只有temp文件夹有可写的权限，所以需要写入权限的文件都放在singularity下的tmp目录下，包括后面的AUGUSTUS_CONFIG_PATH

export SINGULARITY_BIND=/public2/huangdy/others/baiyu/Gbim/Geneanno2/02_braker3/Gbim.chromosome.fa.masked:/tmp/Gbim/Gbim.chromosome.fa.masked,/public2/huangdy/others/baiyu/Gbim/Geneanno2/02_braker3/orthologs_protein.fa:/tmp/Gbim/orthologs_protein.fa,/public2/huangdy/others/baiyu/Gbim/Geneanno/02_rna.bam/03_rna_bam/sum.sort.bam:/tmp/Gbim/sum.sort.bam

# 工作文件夹，好像可以不用设置，不用设置应该是当前目录
WORKDIR=/public2/huangdy/others/baiyu/Gbim/Geneanno2/02_braker3

# 输出文件夹，如果文件夹存在，会先删除整个文件夹
wd=output
if [ -d $wd ]; then
    rm -r $wd
fi

# AUGUSTUS_CONFIG_PATH设置的原因是braker3没有考虑到singularity只有部分文件夹有写入权限，无法添加新物种。所以需要把原来的AUGUSTUS的配置文件复制到新的可写路径，然后在运行到时候指定新的AUGUSTUS_CONFIG_PATH路径。作者给的操作方法地址: https://github.com/Gaius-Augustus/BRAKER/issues/609

singularity exec --writable -B ${PWD}:${PWD} ${BRAKER_SIF} braker.pl --AUGUSTUS_CONFIG_PATH=/public2/huangdy/others/baiyu/Gbim/Geneanno2/02_braker3/config --genome=/tmp/Gbim/Gbim.chromosome.fa.masked  --prot_seq=/tmp/Gbim/orthologs_protein.fa --bam=/tmp/Gbim/sum.sort.bam --threads 30 --busco_lineage=insecta_odb10 --workingdir=${wd}
```

## screen运行

```sh
# 由于nohup后台运行的时候，环境变量会失效，导致程序中断，改成screen运行，screen相当于程序最小化，可以随时查看进度，由于没有root权限，需要conda安装。
conda install conda-forge::screen

# 新建braker3名字的后台
screen -S braker3

# 在打开的窗口执行shell脚本，开始执行后，按ctrl + a再按d，退出窗口，进入最小化运行，如果需要查看进度，用命令打开窗口查看:
screen -r braker3

```

