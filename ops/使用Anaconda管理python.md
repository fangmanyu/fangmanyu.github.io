## 更换国内源

```shell
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/

conda config --set show_channel_urls yes
```

科大镜像

## 创建环境

```
conda create -n env_name package_list
```
 多个包之间使用空格隔开

```
conda create -n py3 python=3.7
```
可以指定python版本

## 进入环境
```
source activate env_name
```
windows环境下使用
```
activate env_name
```

## 退出环境
```shell
source deactivate
```

windows环境下使用
```shell
deactivate
```

## 安装包

### 使用conda install

```shell
conda install mysqlclient
```
如果出现包找不到的异常
```shell
Loading channels: done

PackagesNotFoundError: The following packages are not available from current channels:

  - mysqlclient

Current channels:

  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/win-64
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/noarch

To search for alternate channels that may provide the conda package you're
looking for, navigate to

    https://anaconda.org

and use the search bar at the top of the page.
```
这时可以通过 `anaconda search -t conda mysqlclient` 命令进行搜索
```shell
Using Anaconda API: https://api.anaconda.org
Packages:
     Name                      |  Version | Package Types   | Platforms       | Builds
     ------------------------- |   ------ | --------------- | --------------- | ----------
     Carta/mysqlclient         |   1.3.12 | conda           | linux-64, osx-64 | py36h99db15f_0, py36hb591d34_0
                                          : Python interface to MySQL
     anaconda/mysqlclient      |   1.3.13 | conda           | linux-ppc64le, linux-64, win-32, osx-64, linux-32, win-64 | py37h1de35cc_0, py36h14c3975_0, py36hf
a6e2cd_0, py35h1de35cc_0, py27h14c3975_0, py37hfa6e2cd_0, py37h14c3975_0, py36h1de35cc_0, py27h1de35cc_0, py35hfa6e2cd_0, py35h14c3975_0, py27h0c8e037_0
                                          : Python interface to MySQL
     bioconda/mysqlclient      |   1.3.10 | conda           | linux-64, osx-64 | py27_1, py34_1, py27_0, py36_0, py34_0, py35_0, py35_1
                                          : Python interface to MySQL
     c3i_test2/mysqlclient     |   1.3.12 | conda           | linux-ppc64le   | py36h14c3975_0, py35h14c3975_0, py27h14c3975_0
                                          : Python interface to MySQL
     conda-forge/mysqlclient   |   1.3.13 | conda           | linux-64, osx-64 | py36_0, py27_1000, py36_1000, py27_0
                                          : Python interface to MySQL
     creditx/mysqlclient       |    1.3.7 | conda           | linux-64        | py35_0
     criteo_infratools/python-mysqlclient |    1.3.6 | conda           | linux-64        | py27_2
     gibbycu/mysqlclient       |    1.3.3 | conda           | win-32          | py27, py34
     josh/mysqlclient          |    1.3.6 | conda           | linux-64, win-64 | py27_0, py34_0
                                          : Python interface to MySQL
     lsst/lsst-mysqlclient     |          | conda           | linux-64, osx-64 | b8bcc43_0
     ostrokach-forge/mysqlclient |   1.3.10 | conda           | linux-64        | py27_1, py27_0, py36_1, py36_0, py35_2, py35_0, py35_1
                                          : Python interface to MySQL
     ostrokach/mysqlclient     |    1.3.9 | conda           | linux-64        | py36_2, py35_2, py27_2, py35_0, py35_1
     pcds-tag/mysqlclient      |   1.3.12 | conda           | linux-64        | py37_0, py36_0, py35_0
                                          : Python interface to MySQL
     pdrops/mysqlclient        |   1.3.12 | conda           | linux-64        | py27_1, py27_0
     pmuller/python-mysqlclient |    1.3.6 | conda           | linux-64        | py27_2, py27_1
     rogerramos/mysqlclient    |   1.3.10 | conda           | linux-64        | py27_3
                                          : Python interface to MySQL
     wwbp/mysqlclient          |    1.3.9 | conda           | linux-64        | py35_0
Found 17 packages

Run 'anaconda show <USER/PACKAGE>' to get installation details
```
接着跟着提示，选择一个package
```shell
(ArticleSpider) D:\doc\python\ArticleSpider>anaconda show anaconda/mysqlclient
Using Anaconda API: https://api.anaconda.org
Name:    mysqlclient
Summary: Python interface to MySQL
Access:  public
Package Types:  conda
Versions:
   + 1.3.12
   + 1.3.13

To install this package with conda run:
     conda install --channel https://conda.anaconda.org/anaconda mysqlclient
```
根据提示，完成安装
```shell
conda install --channel https://conda.anaconda.org/anaconda mysqlclient
```

### 使用pip install

如果想要在当前环境下使用pip安装依赖，则需要先安装pip

```shell
conda install -y pip
```

此时使用当前环境的pip

```shell
$ pip --version
pip 19.0.1 from D:\Anaconda\envs\novel\lib\site-packages\pip (python 3.7)
```

接下来使用pip安装

---

## 导出/导入环境

### 导出环境

```shell
conda env export > environment.yaml
```

在当前位置导出 `environment.yaml`文件，里面包含环境信息

### 导入环境

```shell
conda env create -f environment.yaml
```

## 导出/导入依赖

### 导出项目依赖

```shell
conda list -e >requirements.txt
```

### 导入依赖

```shell
conda install --yes --file requirements.txt
```

