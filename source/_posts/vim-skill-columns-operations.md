title: VIM技巧:多行操作
date: 2013-08-16 21:45:53
tags: vim
categories: Develop
---

## 用 `Ctrl + v` 进行列操作

### 按下 `Ctrl + v` 后移动光标，然后按 `I` 插入内容，按 `Esc` 结束。

![insert columns](http://i1273.photobucket.com/albums/y408/angryz/vim-column-edit-000_zps5e6b57f1.gif)


### 按下 `Ctrl + v` 后移动光标，然后按 `x` 删除所选内容。

![delete columns](http://i1273.photobucket.com/albums/y408/angryz/vim-column-edit-001_zpse263eb02.gif)


### 按下 `Ctrl + v` 后移动光标，然后按 `c` 替换所选内容，按 `Esc` 结束。

![modify columns](http://i1273.photobucket.com/albums/y408/angryz/vim-column-edit-002_zps14ec65fa.gif)


### 按下 `Ctrl + v` 后移动光标，按 `$` 移至行末，然后按 `A` 追加内容，按 `Esc` 结束。

![append columns](http://i1273.photobucket.com/albums/y408/angryz/vim-column-edit-003_zps24fffda8.gif)


用 `Ctrl + v` 还何以有更多种用法，请自行发掘。


## 用替换正则表达式的方式操作

### 替换行首占位符 `^` 来实现插入内容到行首。
```vim
:s/^/#/g
```


### 替换行末占位符 `$` 来实现追加内容到行末。
```vim
:s/$/\/\/comment/g
```


### 操作指定区间的行。
```vim
:35,39s/red/blue/g
```
本例是将第35行至39行的内容中所有的 *red* 替换为 *blue*。


通过替换正则表达式，可以实现各种操作，请灵活利用。
