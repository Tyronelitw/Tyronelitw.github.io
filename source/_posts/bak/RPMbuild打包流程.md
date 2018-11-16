---
title: RPMbuild打包流程
date: 2018-04-17 18:02:57
categories:
- 技术
tags:
- 技术
- 文档
---
## RPMbuild编译流程
这是一个编写 RPM 文件的简明教程, 展示了如何快速创建一个简单的源代码包和二进制软件包。关于如何创建 RPM 文件的完整信息（包括更详细的技巧），请参考 [How to create an RPM package](https://fedoraproject.org/wiki/How_to_create_an_RPM_package)。
##基础知识
#1.RPM(RedHat Package Manager)是RedHat的软件包管理器。通过rpm可以在软件安装的过程中就把相关的依赖一起部署完毕，此外，rpm还有数据库协助软件升级、文件校验等；结合yum升级方式，选择rpm打包是很好的一种方法。
#2.若要构建一个标准的 RPM 包，您需要创建 .spec 文件，其中包含软件打包的全部信息。然后，对此文件执行 rpmbuild 命令，经过这一步，系统会按照步骤生成最终的 RPM 包。一般情况，您应该把源代码包，比如由开发者发布的以 .tar.gz 结尾的文件，放入 ~/rpmbuild/SOURCES 目录。将.spec 文件放入 ~/rpmbuild/SPECS 目录，并命名为 "软件包名.spec" 。
## 环境构建
#1.创建一个打包专用的普通用户，避免使用root用户打包发生错误破坏系统：
```bash
useradd makerpmgroupadd mockusermod -a -G mock makerpmpasswd makerpm
```
#2.安装打包需要用到的开发工具：
```bash
yum install  rpmdevtools 
```
#3.切换到打包专用账号下，执行以下命令：
```bash
su - makerpmrpmdev-setuptree
```
将会创建 ~/rpmbuild 目录作为 RPM 编译目录。此目录包含若干子目录，用于保存项目源码、RPM 配置文件以及最终的源码和二进制包。其中，SOURCES目录中存放的是源文件的压缩包（一般是tar.gz格式）以及其所需要的配置文件和patch等。BUILD目录存放build阶段解压的源文件压缩包。SPECS目录存放的是spec文件，打包的核心就是对该spec文件的编写。之后通过对该spec文件执行rpmbuild操作进行打包，最终生成的二进制RPM包存放于RPMS，源文件RPM包存放于SRAMS。同时在install阶段，为保证非root用户能打包，引入了BuildRoot，整个安装过程会在该目录发生。再根据spec文件中的%files指定打包需要哪些文件（从BUILDROOT的相应路径找到文件）。
## 编译过程
#1.我们打包所需的项目源码通常称为 'upstream' 源。我们将从项目网站上，将源码下载到 ~/rpmbuild/SOURCE 目录中。我们获得了一个 tarball 压缩文件, 这正是大多数 FOSS 项目首选发布形式。如果是内部代码托管环境，从内部代码服务器clone代码，然后使用python生成tarball压缩文件。我们平时的编译过程中通常使用一下两种方法：
```bash
cd ~/rpmbuild/SOURCES/
```
1.1
```bash
wget https://files.pythonhosted.org/packages/a1/0d/a1b490503545b3b4600b965eae5d44cc2b6ce27cfb44f4debc563dbb56d3/textfsm-0.4.1.tar.gz
```
1.2
```bash
git clone git://git.openstack.org/openstack/nova
cd nova
git checkout 16.1.0
python setup.py sdist
```
#2.生成spec文件：spec文件可以使用vim或者rpmdev工具生成模板文件，也可以使用src格式rpm获取spec文件。
#2.1
```bash
rpmdev-newspec textfsm
```
或者直接使用vim test.spec
#2.2
```bash
wget http://vault.centos.org/centos/7/cloud/Source/openstack-pike/openstack-nova-16.1.0-1.el7.src.rpm
```
#2.2.1有了源码包，可以执行以下命令安装至 ~/rpmbuild 目录：
```bash
rpm -ivh 源码包名*.src.rpm
```
#2.2.2使用rpm2cpio解压源码包：
```bash
rpm2cpio openstack-nova-16.1.0-1.el7.src.rpm |cpio -div
```
#3.spec文件深入解析：
Name:						#  软件包名，应与 SPEC 文件名一致。可通过宏{%name}来引用
Version:					# 上游版本号 
Release: 1%{?dist}				# rpm包的发行编号
Summary:					#  一行简短的软件包介绍。请使用美式英语。请勿在结尾添加标点！
Group:						# 软件包组
License:					# 授权协议
URL:						# 软件包的项目主页
Source0:					# 软件源码包的URL地址
BuildRoot:%{_tmppath}/%{name}-%{version}-%{release}-root-%(%{__id_u} -n)
						# 在 %install 阶段（%build 阶段后）文件需要安装至此位置。
BuildRequires:					# build过程需要安装的依赖
Requires:					# 编译生成的rpm安装所需要的依赖
%description					#  程序的详细/多行描述，请使用美式英语。每行必须小于等于 80 个字符。
%prep						#  打包准备阶段执行一些命令（如，解压源码包，打补丁等），以便开始编译。
%setup -q -n %{name}-%{version} 		# 解压源码包到BUILD目录下
%build						#  包含构建阶段执行的命令，构建完成后便开始后续安装。
%configure
make
%{?_smp_mflags}
%install					#  包含安装阶段执行的命令。
rm -rf %{buildroot}
make install DESTDIR=%{buildroot}
%clean						#  清理安装目录的命令。
rm -rf %{buildroot}
%files						#  需要被打包/安装的文件列表。
%defattr(-,root,root,-)
%doc
%changelog					#  RPM 包变更日志。请使用示例中的格式。注意，不是软件本身的变更日志。
spec文件详细介绍请参照：[spec文件详细介绍] (https://docs.fedoraproject.org/quick-docs/en-US/creating-rpm-packages.html#con_rpm-spec-file-overview)。
#4.在编写spec文件时，使用到很多RPM宏命令，这些宏命令的定义可在/usr/lib/rpm/macros中查看，也可以通过以下命令查看： rpm --showrc有关宏命令的完整列表请参考：[Macros]（http://fedoraproject.org/wiki/Packaging:Guidelines#Macros）
5.编译rpm包：
```bash
rpmbuild  -ba NAME.spec
```
rpmbuild命令可选参数：
-bp         #只执行spec的%pre 段(解开源码包并打补丁，即只做准备)
-bc         #执行spec的%pre和%build 段(准备并编译)
-bi         #执行spec中%pre，%build与%install(准备，编译并安装)
-bl         #检查spec中的%file段(查看文件是否齐全)
-ba         #建立源码与二进制包(常用)
-bb         #只建立二进制包(常用)
-bs         #只建立源码包
## 搭建yum源 
#1.管理yum源需要一个工具createrepo，使用yum安装：
```bash
yum -y install createrepo
```
#2.创建yum仓库目录：
```bash
mkdir -p /date/repo/yum/ctyun/centos/{6,7}/cloud/x86_64/openstack-pike/
```
#3.生成repodate信息：
```bash
cd  /date/repo/yum/ctyun/centos/7/cloud/x86_64/openstack-pike/createrepo -pdo   .    .
```
#4.使用nignx对外提供web服务：
```bash
yum install -y nginx
vim /etc/nginx/nginx.conf
http {
……
autoindex on; 
autoindex_exact_size on;
autoindex_localtime on;
……
server {
……
root         /date/repo/yum/;
……
	}
}
```
#5.重启Nginx服务，配置yum源配置文件：
```bash
vim /etc/yum.repos.d/local.repo
[cloud]
name=CentOS-$releasever - Cloud
baseurl=http://localhost_ip/ctyun/centos/$releasever/cloud/$basearch/openstack-pike/
gpgcheck=0
enabled=1
```
#6.每次将打好的rpm包拷到yum目录后需要重新更新yum索引文件:
```bash
cd /date/repo/yum/ctyun/centos/7/cloud/x86_64/openstack-pike/
createrepo --update . 
```
## 参考文献
1.[Fedora社区关于如何创建rpm的文档：](https://fedoraproject.org/wiki/How_to_create_a_GNU_Hello_RPM_package/zh-cn)
2.[Fedora社区详细编译rpm文档：](https://docs.fedoraproject.org/quick-docs/en-US/creating-rpm-packages.html)
3.[CentOS官方SRC源码包下载地址：](http://vault.centos.org/)

