# Install R-3.1.3 tarball on Cloudera

	注意事項：
		1. 文章僅包含安裝函式庫
		2. 安裝好後可執行『 R -f {file path}/{file name}.R 』得到結果
		3. 安裝須使用 root 權限執行

### Download tarball

[下載網址](http://cran.r-project.org/src/base/R-3/R-3.1.3.tar.gz)

### Install Dependency Packages

	$ yum install gcc-gfortran	
	$ yum install readline-devel 
	$ yum install libXt-devel 

### Unzip

	$ tar -zxvf R-3.1.3.tar.gz
	$ cd R-3.1.3
	
### Congiure and Make Install
	
	$ ./configure --enable-R-shlib
	$ make
	$ make install