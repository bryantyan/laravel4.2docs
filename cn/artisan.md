# Artisan CLI

* [简介](artisan.md#introduction)
* [用法](artisan.md#usage)

## 简介

Artisan是Laravel中自带的命令行工具的名称。它提供了一些开发过程中有用的命令用。它是基于强大的Symfony Console 组件开发的。

## 用法

### 列出所有可用命令

执行 `list` 命令可以列出所有可用的Artisan命令：

```text
php artisan list
```

### 查看某个命令的帮助文档

每个命令都包含有 "help" 信息，用以提供此命令所允许的参数和选项。只需在命令前边添加 `help` 即可：

```text
php artisan help migrate
```

### 指定配置环境

通过指定 `--env` 选项，你可以指定命令执行时的配置环境：

```text
php artisan migrate --env=local
```

### 显示当前的Laravel版本

你可以使用 `--version` 选项查看当前安装的Laravel的版本：

```text
php artisan --version
```

译者：王赛 [（Bootstrap中文网）](http://www.bootcss.com)

