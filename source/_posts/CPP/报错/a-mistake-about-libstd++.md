---
title: _zst28__throw_bad_array_new_lengthv error
tags:
  - CPP
  - error
  - mingw
  - windows
categories:
  - CPP
  - error
abbrlink: b1bcf10b
date: 2023-03-17 23:16:39
---

今天敲代码突然出现一个奇葩的报错。

使用了 `std::vector` 或 `std::array` 时初始化会无法运行。（下为复现）

<img  alt="图 1" src="1679067649543.png"  />  


在终端编译出exe文件后，打开出现如下错误提示

<img alt="图 2" src="1679067949644.png" />  


起初以为是 g++ 版本问题， 当时使用的版本是 msys2 `gcc 11.2.0` 。

将 gcc 版本升级到 `gcc 12.2.0` 后，问题仍然存在。

便认为是 MSYS2 环境中 使用 `-static-libgcc` 导致的问题。

便将 gcc 环境换成了 MinGW-W64。

但仍然没解决问题，一筹莫展时看到这位大佬的博客，

<img  alt="图 1" src="1679068672457.png" />  


意思大概是系统的 `libstd++` 不匹配，系统没有从编译器目录获得这个库。

将 gcc 环境变量 `XXX\mingw64\bin` 优先级变高（将它移到前面）后问题得到解决。

排查后发现是 neovim 的环境变量 `XXX\Neovim\bin` 导致的问题。

