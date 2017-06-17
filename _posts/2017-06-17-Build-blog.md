---
layout: post
title: Build Blog
description: "jekyll博客搭建"
modified: 2017-06-17
tags: [php]
categories: [intro]
---
## 换电脑要重新搭建环境，就过程中遇到的坑做个总结

## >更新gem源
- `gem sources --add http://gems.ruby-china.org/ --remove https://rubygems.org/`
### warning:
`ERROR:  While executing gem ... (OpenSSL::SSL::SSLError)`
- 注意`http://gems.ruby-china.org/`不是https协议

<!-- more -->

## >运行`jekyll -v`
- 报错：

```
/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/
core_ext/kernel_require.rb:55:in `require': cannot load such file -- bundler (LoadError)

```
## 安装依赖
- `bundle install`

## 启动jekyll
- `bundle exec jekyll server`



