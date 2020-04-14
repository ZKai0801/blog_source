---
title: 生信软件安装
date: 2019-12-04
tag: ["IT", "Bioinformatics"]
---

生信中很多软件安装起来都相当麻烦，这里推荐两种途径来解决这一问题：bioconda 以及 docker

<!--more-->

### Bioconda
Anaconda 是一个非常常用的包管理软件，同时也会对环境进行统一的管理，而Bioconda正是其之下的一个专门管控生信软件的工具，具体安装步骤如下
```bash
# 首先下载并安装 miniconda （anaconda的简版）
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh

# 默认情况下，miniconda会被安装在家目录之下
# 所有可执行文件，即在 ~//miniconda3/bin 下面
# 而不同的环境，会被安装在 ~/miniconda3/envs 之下

# 防止conda开机自动启动
conda config --set auto_activate_base false

# 添加频道 （国内的源，速度更快）
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda

# 当频道添加完毕，则可开始安装生信软件
# 安装方式有两种，第一中是在当前环境下直接安装：
# 例：
conda install bwa

# 第二种则是建立一个虚拟环境，并安装软件
# 这里我们建立了一个名叫 aligner 的环境，并在这个环境中安装了 tophat, bowtie2 和 star 三款比对软件
conda create -n aligner tophat bowtie2 star

# 一些常用命令：
# 查看现有环境
conda env list

# 进入某一虚拟环境
conda activate aligner

# 推出当前虚拟环境
conda deactivate

# 查看当前环境中已安装的包
conda list

```

---

### Docker
docker不同与bioconda，并非一个专门安装生信软件的管理工具，而是一个用于软件的打包、发布、移植的通用工具。
如所需生信软件在bioconda上并不存在，并且直接安装过于繁琐，则可尝试在docker hub上先搜索有无他人打包完成的镜像。若以上方法均难以实现，再自己配置打包。然而自己制作镜像的难度较大，这里只简单介绍一下docker的基础原理以及使用方法：


1. image -- 镜像 与 container -- 容器

   镜像与容器的关系，有点类似于python中类与实例的关系

   镜像是类，而容器是这个类的一个实例。也就意味着一个镜像可以创建很多的容器，这些容器独立运行，互不干扰。镜像本身并不能运行，它只能当作一个创建容器用的模板。




2. Volume -- 卷（数据管理）

   数据主要分为两类，持久化的与非持久化的。 

   而Docker擅长的是非持久化数据，即每个容器创立之时就会自动创建存储，此存储从属于容器，并与容器保持相同生命周期， 这意味着删除容器也会删除全部非持久化数据。



```bash
# 安装docker
sudo apt-get update
sudo apt-get install -y docker-engine

# 通过 Dockerfile 建立一个 container
# 假设这个文件存储在 ~/docker_test 这个文件夹中
docker build ~/docker_test -t ContainerName:TagName

# 或者在 docker hub 上寻找并拉去一个 container
docker search cnvkit
docker pull docker.io/etal/cnvkit

# 运行docker container中的文件
docker run --rm etal/cnvkit /usr/local/bin/cnvkit.py --help

## 推荐方法
# 在后台运行，container保持运行状态
# -v + -p 挂载并且监听
# 这行命令会输出一行 container ID
docker run -v `pwd`:/data_test -p 5000:80 -itd etal/cnvkit 

# 在已创建的container中执行命令
docker exec [container ID] /usr/local/bin/cnvkit.py batch /data_test/NGS190530-11DF.sorted.dedup.bam -r /data_test/DF.flat.cnn -p 16 -d /data_test

```
