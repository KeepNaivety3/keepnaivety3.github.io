---
title: 使用源代码编译安装qBittorrent
date: 2024-01-15
categories: [Linux, 编译]
tags: [qbittorrent, CentOS]
---

> 对于 CentOS Stream 9，现在可以直接使用 [EPEL](https://rhel.pkgs.org/9/epel-x86_64/qbittorrent-4.6.2-1.el9.x86_64.rpm.html) 源安装 `qBittorrent-4.6.2`，对于 CentOS Strean 8 也有 `qbittorrent-4.2.5` 可供选择。
{: .prompt-tip }

因为 yum 源中并没有 qBittorrent 可以选择，同时我会选择安装 `qBittorrent-4.3.9` + `libtorrent-rasterbar 1.2` 作为我的 BT 客户端，这里提供一个从源码编译安装的方式。

对于 `qBittorrent-4.3.9` 的使用有以下要求:
  - Boost > 1.65.0
  - libtorrent > 1.0.6
  - Qt > 5.9

## 安装前准备

安装必要的工具

```bash
yum install -y epel-release
yum install -y autoconf automake gcc gcc-c++ git glib2 glibc gmp gnutls libblkid libcap libffi libgcc libgcrypt libgpg-error libicu libidn2 libmount libselinux libstdc++ libtasn1 libtool libunistring libuuid lz4-libs make nettle openssl-devel openssl-libs p11-kit pcre pcre2 qt5-qtbase systemd-libs tar wget xz-libs zlib zlib-devel
yum install -y qt5-linguist qt5-qttools-devel qt5-qtsvg-devel
```

安装 Screen，避免编译时错误断开造成的终端，在进行编译安装前新建一个会话。

```bash
yum install -y screen
screen -S qBittorrent
```

## 编译安装 Boost 库

截至写文章时，最新的 Boost 库版本是 1.84.0

```bash
wget https://boostorg.jfrog.io/artifactory/main/release/1.84.0/source/boost_1_84_0.tar.gz
```

编译安装

```bash
# 如果你的服务器只有一个核心，请去掉 -j$(( $(nproc) - 1 ))
export DIR_BOOST="/opt/boost"
tar -xvf boost_1_84_0.tar.gz
cd boost_1_84_0
./bootstrap.sh --prefix=${DIR_BOOST}
./b2 install --prefix=${DIR_BOOST} --with=all -j$(( $(nproc) - 1 ))
```

## 编译安装 Libtorrent

`Libtorrent` 是 `Arvid Norberg` 编写的一个库，`qBittorrent` 依赖于该库。在编译 `qBittorrent` 之前，有必要编译并安装 `libtorrent`。

克隆这个 repo：``git clone --depth 1 -b RC_1_2 https://github.com/arvidn/libtorrent.git``

```bash
cd libtorrent
./autotool.sh
./configure --prefix=/usr --disable-debug --enable-encryption --with-boost=${DIR_BOOST}
make -j$(( $(nproc) - 1 ))
make install
ln -s /usr/lib/pkgconfig/libtorrent-rasterbar.pc /usr/lib64/pkgconfig/libtorrent-rasterbar.pc
```

## 编译安装 qBittorrent-nox

qBittorrent 的源码可以直接从 [Github release](https://github.com/qbittorrent/qBittorrent/releases/tag/release-4.3.9) 下载 `.tar` 的压缩档，也可以选择克隆这仓库：``git clone --depth 1 -b v4_3_x https://github.com/qbittorrent/qBittorrent``

```bash
cd qBittorrent
./configure --prefix=/usr --disable-gui CPPFLAGS=-I/usr/include/qt5 --with-boost=${DIR_BOOST}
make -j$(( $(nproc) - 1 ))
make install
```

如果你选择添加 `--disable-gui` 这个参数，就仅能通过 `WebUI` 访问 `qBittorrnet`。`WebUI` 的默认端口是 `8080`，使用默认账号密码访问 `http://localhost:8080`：

```
Username: admin
Password: adminadmin
```

## 将 qBittorrent-nox 添加为服务

要使用 `systemctl` 控制 `qBittorrent`，需要编写一个 `.service` 文件，添加一个 `/etc/systemd/system/qbittorrent.service` 文件：

```
[Unit]
Description=qbittorrent torrent server

[Service]
User=root
ExecStart=/usr/bin/qbittorrent-nox
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

## 为 qBittorrent WebUI 端口设置 Apache httpd 的反向代理

```
ProxyPass "/qbt/" "http://127.0.0.1:8080/"
ProxyPassReverse "/qbt/" "http://127.0.0.1:8080/"
```

## 疑难解答

如果出现类似的问题：

```
qbittorrent-nox: error while loading shared libraries: libtorrent-rasterbar.so 10: cannot open shared object file： 没有此类文件或目录
```

这是因为 qBittorrent 需要的库不在 ``/usr/lib64/`` 中。创建一个软链接来解决这个问题。像这样做：

```bash
ln -s /usr/lib/libtorrent-rasterbar.so.10 /usr/lib64/libtorrent-rasterbar.so.10
```

对于缺少 ``libboost_system.so.1.84.0``：

```bash
ln -s /opt/boost/lib/libboost_system.so.1.84.0 /usr/lib64/libboost_system.so.1.84.0
```