# 说明

[gitbook 文档](https://github.com/GitbookIO/gitbook)
[markdown 文档](https://markdown-zh.readthedocs.io/en/latest/)

## 安装 gitbook

```sh
npm install gitbook-cli -g
```

## 预览

> 第一次下载一些文件，请耐心等待

```sh
gitbook serve
```

## 编译

> 注意 docs 之前和./中间需要有空格，是两个参数

```sh
gitbook build ./ docs
```

`编译后git提交后，就可以发布了`
