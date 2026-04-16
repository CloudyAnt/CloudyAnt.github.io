---
title: Linux 标准数据流 (Standard Stream)
date: 2026-04-12 11:00:00
categories:
  - Linux
tags:
  - Standard Stream
  - Redirection
---

## 标准数据流 (Standard Stream)

Linux 命令执行时，通常会打开三个标准数据流：stdin、stdout、stderr
stdin 标准输入，来自键盘输入、pipe、here-document、here-string 或文件等。stdout 和 stderr 分别是标准输出和标准错误输出，默认都流向终端，也可以被重定向到 pipe 或文件等

## 文件描述符 (File Descriptors)

Linux 中几乎所有东西，包括数据流都被视作文件。和一个进程关联的每个文件都会被分配到单独的标识数字，即文件描述符。一般地，0 表示标准输入 (stdin), 1 表示标准输出 (stdout),  2 表示标准错误输出 (stderr)

## 重定向 (Redirection)

### 重定向 stdout 和 stderr

```shell
# 使用 stdout 替换文件内容
echo hello > new 

# 将 stdout 追加到文件末尾
echo hello >> old 

# 重定向 stdout 到 stderr
echo hello >&2 

# 重定向 stderr 到当前 stdout 的目标
command 2>&1 

# 在 bash/zsh 中，&> file 表示 stdout 和 stderr 都重定向到 file（非 POSIX 写法）
command &> log

# 使用 stderr 替换文件内容
command 2> new

# 将 stderr 追加到文件末尾
command 2>> old 

# 重定向所有输出到文件
command > log 2>&1 
# 要先重定向 stdout 到文件，再重定向 stderr 到 stdout
# 若反之，stderr 将被定向到 stdout 的原先的目的地（通常是终端）
# 在 bash/zsh 中，这行命令也可写成 command &> log（非 POSIX 写法）
```

### 指定 stdin 输入源

指定已有文件作为输入
```shell
cat < foo.txt
# 输出：Hello world
```

here-document。指定结束符，程序会将结束符前的所有内容作为 stdin

```shell
name=jakey

# 结束符没有引号时，shell 会尝试解析变量
cat << END_WORD
> hello, $name
> END_WORD
# 输出：hello, jakey

# 结束符带有引号时，shell 会禁用变量替换
cat << 'END_WORD'
> hello, $name
> END_WORD
# 输出：hello, $name
```

here-string。指定预先写的 string 作为 stdin

```shell
cat <<< "Hello world"
# 输出：Hello world
```

pipe。将前一命令输出作为后一命令的输入

```shell
echo 123 | wc
# 输出：1	  1   4
```

process substitution。将命令输出以临时文件描述符路径的形式传给另一个命令。与 pipe 不同的是，它更适合需要“文件名参数”的场景（如 diff）。这里面的命令通常仍在子进程中执行

```shell
cat <(echo 123) <(echo 456)
# 输出：123
# 输出：456
diff <(ls *.c | cut -d. -f1) <(ls *.out | cut -d. -f1)
```

## 其他

### 输出到文件的过程中可以移动、重命名或删除文件

在同一个文件系统下移动或重命名文件，仅改变访问路径或文件名，其在磁盘上的位置，即索引（inode）并不改变。而程序正是通过索引来找文件的
删除文件后，不能再通过常规的方式访问到文件，但程序依然可以通过索引来访问文件。这些文件只有在程序关闭文件描述符后才真正被释放（会造成实际的磁盘占用比系统报告的大）

### 判断输出目的地

一个常见的方法是使用 test 命令（也可使用 [ 和 ]）与 -t 选项，来检查文件描述符（如 1 代表 stdout）是否与终端相连。-t 选项会检查给定文件描述符是否连接到终端：

```shell
if [ -t 1 ]; then
    echo "输出到终端"
else
    echo "输出被重定向到文件或其他地方"
fi
```

通过这个方法可以向不同的输出目的地输出不同的结果，Linux 命令如 ls、git 等都有类似优化。

---

This article was inspired by [ThoughtBot](https://thoughtbot.com/blog/input-output-redirection-in-the-shell) and [HowToGeek](https://www.howtogeek.com/435903/what-are-stdin-stdout-and-stderr-on-linux/)
