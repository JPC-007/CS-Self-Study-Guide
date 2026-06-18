1. For this course, you need to be using a Unix shell like Bash or ZSH. If you are on Linux or macOS, you don’t have to do anything special. If you are on Windows, you need to make sure you are not running cmd.exe or PowerShell; you can use [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) or a Linux virtual machine to use Unix-style command-line tools. To make sure you’re running an appropriate shell, you can try the command `echo $SHELL`. If it says something like `/bin/bash` or `/usr/bin/zsh`, that means you’re running the right program.

```shell
jpc@PengchengdeMac-mini ~ % echo $SHELL
/bin/zsh
```

2. Create a new directory called `missing` under `/tmp`.

```shell
jpc@PengchengdeMac-mini ~ % mkdir /tmp/missing
jpc@PengchengdeMac-mini ~ % cd /tmp
jpc@PengchengdeMac-mini /tmp % ls
2f093fe9-a235-5d1c-9a62-3fae2cdf7eb1
7c3338c5-8eec-50ae-bc4a-1f6969419936
boost_interprocess
logitech_kiros_agent-452dff54306495e1b48f832a66623b16
logitech_kiros_logivoice-452dff54306495e1b48f832a66623b16
logitech_kiros_updater
missing
paragon-uc-cache.log
powerlog
Sublime Text.4cff18d2bab96a93366319a9e0fa060d.452dff54306495e1b48f832a66623b16.sock
verge
jpc@PengchengdeMac-mini /tmp %
```

3. Look up the `touch` program. The `man` program is your friend.

```shell
jpc@PengchengdeMac-mini ~ % man touch
TOUCH(1)                                    General Commands Manual                                   TOUCH(1)

NAME
     touch – change file access and modification times

SYNOPSIS
     touch [-A [-][[hh]mm]SS] [-achm] [-r file] [-t [[CC]YY]MMDDhhmm[.SS]] [-d YYYY-MM-DDThh:mm:SS[.frac][tz]]
           file ...

DESCRIPTION
     The touch utility sets the modification and access times of files.  If any file does not exist, it is
     created with default permissions.

     By default, touch changes both modification and access times.  The -a and -m flags may be used to select
     the access time or the modification time individually.  Selecting both is equivalent to the default.  By
     default, the timestamps are set to the current time.  The -d and -t flags explicitly specify a different
     time, and the -r flag specifies to set the times those of the specified file.  The -A flag adjusts the
     values by a specified amount.

     The following options are available:

     -A      Adjust the access and modification time stamps for the file by the specified value.  This flag is
             intended for use in modifying files with incorrectly set time stamps.

             The argument is of the form “[-][[hh]mm]SS” where each pair of letters represents the following:

                   -       Make the adjustment negative: the new time stamp is set to be before the old one.
                   hh      The number of hours, from 00 to 99.
                   mm      The number of minutes, from 00 to 59.
                   SS      The number of seconds, from 00 to 59.

             The -A flag implies the -c flag: if any file specified does not exist, it will be silently
             ignored.

     -a      Change the access time of the file.  The modification time of the file is not changed unless the
             -m flag is also specified.

     -c      Do not create the file if it does not exist.  The touch utility does not treat this as an error.
             No error messages are displayed and the exit value is not affected.

     -d      Change the access and modification times to the specified date time instead of the current time
             of day.  The argument is of the form “YYYY-MM-DDThh:mm:SS[.frac][tz]” where the letters represent
             the following:
                   YYYY    At least four decimal digits representing the year.
                   MM, DD, hh, mm, SS
                           As with -t time.
                   T       The letter T or a space is the time designator.
                   .frac   An optional fraction, consisting of a period or a comma followed by one or more
                           digits.  The number of significant digits depends on the kernel configuration and
                           the filesystem, and may be zero.
                   tz      An optional letter Z indicating the time is in UTC.  Otherwise, the time is assumed
                           to be in local time.  Local time is affected by the value of the TZ environment
                           variable.

     -h      If the file is a symbolic link, change the times of the link itself rather than the file that the
             link points to.  Note that -h implies -c and thus will not create any new files.

     -m      Change the modification time of the file.  The access time of the file is not changed unless the
             -a flag is also specified.

     -r      Use the access and modifications times from the specified file instead of the current time of
             day.

     -t      Change the access and modification times to the specified time instead of the current time of
```

4. Use `touch` to create a new file called `semester` in `missing`.

```shell
jpc@PengchengdeMac-mini ~ % mkdir /tmp/missing
jpc@PengchengdeMac-mini ~ % touch /tmp/missing/semester
jpc@PengchengdeMac-mini ~ % cd /tmp/missing
jpc@PengchengdeMac-mini missing % ls
semester
```

5. Write the following into that file, one line at a time:

```
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
```

The first line might be tricky to get working. It’s helpful to know that `#` starts a comment in Bash, and `!` has a special meaning even within double-quoted (`"`) strings. Bash treats single-quoted strings (`'`) differently: they will do the trick in this case. See the Bash [quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) manual page for more information.

```shell
# 写入第一行（单引号包裹，原样输出!）
echo '#!/bin/sh' > /tmp/missing/semester

# 写入第二行（追加，单引号避免特殊字符问题）
echo 'curl --head --silent https://missing.csail.mit.edu' >> /tmp/missing/semester

cat > /tmp/missing/semester << 'EOF'
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
EOF
```

6. Try to execute the file, i.e. type the path to the script (`./semester`) into your shell and press enter. Understand why it doesn’t work by consulting the output of `ls` (hint: look at the permission bits of the file).

```shell
cd /tmp/missing
./semester
bash: ./semester: Permission denied
ls -l semester
-rw-r--r-- 1 youruser yourgroup 0 月 日 时:分 semester
chmod +x semester
ls -l semester
-rwxr-xr-x
./semester
```

7. Run the command by explicitly starting the `sh` interpreter, and giving it the file `semester` as the first argument, i.e. `sh semester`. Why does this work, while `./semester` didn’t?

```shell
./semester：尝试直接执行文件 → 必须有 x 执行权限
sh semester：执行 sh 程序，让它读取并运行文本文件 → 只需要 r 读权限
补充关键知识点（Linux 通用规则）
文本文件本身不需要执行权限，只要能被读取，解释器就能运行它
只有你想把文件本身当成程序直接运行时，才需要 x 权限
之前用 chmod +x 就是给文件加了 x，让 ./semester 可以运行
```

8. Look up the `chmod` program (e.g. use `man chmod`).

```shell
略
```

9. Use `chmod` to make it possible to run the command `./semester` rather than having to type `sh semester`. How does your shell know that the file is supposed to be interpreted using `sh`? See this page on the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line for more information.

```shell
运行 chmod +x /tmp/missing/semester 赋予执行权限
直接执行 ./semester 即可运行
Shebang (#!/bin/sh) 是系统的「指路牌」，告诉内核用 sh 解释脚本
```

10. Use `|` and `>` to write the “last modified” date output by `semester` into a file called `last-modified.txt` in your home directory.

```shell
jpc@PengchengdeMac-mini missing % ./semester | grep "last-modified" > ~/last-modified.txt
jpc@PengchengdeMac-mini missing % cat ~/last-modified.txt
last-modified: Sat, 09 May 2026 15:49:16 GMT
jpc@PengchengdeMac-mini missing %
```

11. Write a command that reads out your laptop battery’s power level or your desktop machine’s CPU temperature from `/sys`. Note: if you’re a macOS user, your OS doesn’t have sysfs, so you can skip this exercise.

```shell
电池电量：cat /sys/class/power_supply/BAT0/capacity
CPU 温度：cat /sys/class/thermal/thermal_zone0/temp
核心：/sys 把硬件信息变成文本文件，直接读取即可完成题目要求
```



# 笔记

## 练习 1：检查当前 Shell 环境

### 题目

确认正在使用 Bash/Zsh 等标准 Unix Shell，禁止使用 Windows cmd/PowerShell。

### 实操命令

```bash
echo $SHELL
```

### 执行结果参考

```shell
jpc@PengchengdeMac-mini ~ % echo $SHELL
/bin/zsh
```

### 核心知识点

1. `$SHELL` 是系统环境变量，用于查看当前登录使用的 Shell 解释器。
2. 判定标准：输出包含 `/bin/bash`、`/usr/bin/zsh` 即为合规 Unix Shell。

## 练习 2：在 `/tmp` 下创建 `missing` 目录

### 题目

新建目录 `/tmp/missing`。

### 实操命令

```bash
# 创建目录
mkdir /tmp/missing
# 进入目录 + 验证创建成功
cd /tmp
ls
```

### 核心知识点

1. `mkdir 路径`：创建目录（`mkdir` = make directory）。
2. `/tmp`：系统**临时目录**，所有用户均可读写，系统会自动清理过期文件。
3. `ls`：列出目录内文件 / 文件夹。

## 练习 3：查看 `touch` 命令手册

### 题目

使用 `man` 工具查阅 `touch` 官方文档。

### 实操命令

```bash
man touch
```

### 核心知识点

1. `man 命令名`：查看命令官方手册（Unix 标配帮助工具）。
   1. 翻页：方向键 / 空格键；退出手册：按 `q`。
2. `touch` 两大核心功能：
   1. 文件不存在：**创建空文件**；
   2. 文件已存在：更新文件的访问 / 修改时间戳。

## 练习 4：用 `touch` 在 `missing` 中创建 `semester` 文件

### 题目

在 `/tmp/missing` 下创建空文件 `semester`。

### 实操命令

```bash
# 直接通过绝对路径创建
touch /tmp/missing/semester
# 进入目录验证
cd /tmp/missing
ls
```

### 补充

`touch` 是创建小型空文件的最简命令。

## 练习 5：向文件写入两行脚本内容

### 题目

向 `semester` 写入以下内容：

```Plain
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
```

### 实操方案（两种常用写法）

#### 方案 1：`echo` + 重定向（逐行写入）

```bash
# > 覆盖写入第一行
echo '#!/bin/sh' > /tmp/missing/semester
# >> 追加写入第二行
echo 'curl --head --silent https://missing.csail.mit.edu' >> /tmp/missing/semester
```

#### 方案 2：`cat + Here Document`（多行一次性写入，推荐）

```bash
cat > /tmp/missing/semester << 'EOF'
#!/bin/sh
curl --head --silent https://missing.csail.mit.edu
EOF
```

### 核心知识点

1. **引号规则（高频易错）**
   1. 单引号 `' '`：原样解析所有字符，**屏蔽** **`!`****、****`$`** **等特殊符号**（本题必须用单引号）；
   2. 双引号 `" "`：无法屏蔽 `!`，会触发 Shell 历史命令扩展，导致写入失败。
2. **输出重定向**
   1. `>`：覆盖写入，清空原文件内容后写入新内容；
   2. `>>`：追加写入，在文件末尾新增内容，不改动原有数据。
3. **Here Document (****`<< EOF`****)**
   1. `<<`：多行输入重定向，终端持续等待手动输入内容；
   2. `'EOF'` 加单引号：关闭特殊字符解析；
   3. 结束标记 `EOF` 要求：**单独一行、前后无任何空格 / 字符**。
4. 脚本内容说明
   1. `#!/bin/sh`：脚本标记（后续详解）；
   2. `curl --head`：仅获取网页响应头；`--silent`：静默模式，不输出冗余日志。

## 练习 6：直接执行脚本 & 排查权限问题

### 题目

执行 `./semester`，分析执行失败原因。

### 实操命令 & 现象

```bash
cd /tmp/missing
# 直接执行脚本，报错
./semester
# 查看文件详细权限
ls -l semester
```

报错信息：

```Plain
bash: ./semester: Permission denied （权限被拒绝）
```

权限输出参考：

```Plain
-rw-r--r--  1 user  group  0  xx xx semester
```

### 核心知识点

1. `ls -l`：查看文件**详细属性**，首段 10 位字符为**权限位**。
2. 基础权限标识：
   1. `r` = 读权限、`w` = 写权限、`x` = 执行权限、`-` = 无对应权限；
3. 权限分段（`-rw-r--r--`）：
   1. 第 1 位：文件类型（`-` 代表普通文件）；
   2. 2~4 位：文件**所有者**权限（本例：`rw-` → 可读、可写、**无执行权限**）；
   3. 5\7 位：同组用户权限；8\10 位：其他用户权限。
4. 失败原因：新建文件默认**没有执行权限** **`x`**，无法直接运行。

## 练习 7：用 `sh` 解释器运行脚本，对比两种执行方式

### 题目

执行 `sh semester`，对比 `./semester` 分析差异。

### 实操命令

```bash
sh semester
```

### 核心原理（重点）

| 执行命令      | 本质                               | 所需权限                  |
| ------------- | ---------------------------------- | ------------------------- |
| `./semester`  | 将文件当作**可执行程序**直接运行   | 必须具备 **执行权限 (x)** |
| `sh semester` | 运行 `sh` 解释器，读取文本文件执行 | 仅需 **读权限 (r)**       |

补充：文件本身只是纯文本，`sh` 作为独立程序读取内容即可运行，因此不需要执行权限。

## 练习 8：查看 `chmod` 命令手册

### 题目

查阅 `chmod` 用法。

### 实操命令

```bash
man chmod
```

### 核心知识点

`chmod`（change mode）：Unix 系统**修改文件 / 目录权限**的专属命令。

## 练习 9：添加执行权限 + 理解 Shebang

### 题目

使用 `chmod` 赋予脚本执行权限，实现 `./semester` 直接运行；理解 Shell 如何识别解释器。

### 实操命令

```bash
# 为所有用户添加执行权限
chmod +x semester
# 验证权限变化
ls -l semester
# 直接运行脚本
./semester
```

权限变化：`-rw-r--r--` → `-rwxr-xr-x`（新增 `x` 执行位）。

### 核心知识点

1. `chmod +x 文件名`：快速为文件添加执行权限（最常用写法）。
2. **Shebang（****`#!`****）脚本头部标记**
   1. 格式：`#! + 解释器路径`，本例 `#!/bin/sh`；
   2. 作用：当文件拥有执行权限、被直接运行时，系统会读取文件第一行，**自动调用指定的解释器（sh）** 执行脚本，无需手动输入 `sh`。

## 练习 10：管道 `|` + 重定向 `>` 筛选并写入文件

### 题目

提取脚本输出中的 `Last-Modified` 内容，写入**家目录**的 `last-modified.txt`。

### 实操命令

```bash
# 管道过滤 + 重定向写入家目录文件
./semester | grep "last-modified" > ~/last-modified.txt
# 验证文件内容
cat ~/last-modified.txt
```

执行结果参考：

```Plain
last-modified: Sat, 09 May 2026 15:49:16 GMT
```

### 核心知识点

1. **管道** **`|`**：将前一个命令的输出，作为后一个命令的输入，实现命令串联。
   1. 本例：把 `./semester` 的输出传给 `grep`。
2. `grep 关键词`：文本过滤命令，只保留包含指定关键词的行。
3. `~`：代表**当前用户家目录**，`~/文件名` 等价于 `$HOME/文件名`。
4. `>`：将过滤后的最终结果，覆盖写入目标文件。

## 练习 11：读取 `/sys` 硬件信息（macOS 可跳过）

### 题目

从 Linux 专属虚拟文件系统 `/sys` 读取电池电量 / CPU 温度。

### 说明

macOS、Windows 无 `sysfs`（`/sys`），**直接跳过本练习**。

### Linux 系统实操命令

1. 读取笔记本电池电量

```bash
cat /sys/class/power_supply/BAT0/capacity
```

输出：纯数字（代表电量百分比）。

1. 读取 CPU 温度（单位：毫摄氏度，需 ÷1000 转为常规温度）

```bash
cat /sys/class/thermal/thermal_zone0/temp
```

### 核心知识点

`/sys` 是 Linux 内核提供的**虚拟文件系统**，将硬件信息以文本文件形式暴露，直接用 `cat` 读取即可获取硬件数据。

# 一、核心命令速查表

| 命令            | 作用                                  |
| --------------- | ------------------------------------- |
| `echo 内容`     | 终端输出文本                          |
| `mkdir 目录`    | 创建目录                              |
| `touch 文件`    | 创建空文件 / 更新文件时间戳           |
| `ls` / `ls -l`  | 列出目录内容 / 查看文件权限详情       |
| `cat 文件`      | 查看文件内容 / 配合重定向写入多行内容 |
| `man 命令`      | 查看命令官方手册                      |
| `chmod +x 文件` | 给文件添加执行权限                    |
| `grep 关键词`   | 过滤文本内容                          |

# 二、符号规则总结（重中之重）

1. **重定向**
   1. `>`：覆盖输出到文件；`>>`：追加输出到文件；
   2. `<`：文件作为命令输入；`<< EOF`：多行手动输入（Here Document）。
2. **管道**
   1. `|`：串联多个命令，传递数据流。
3. **引号**
   1. 单引号 `' '`：完全屏蔽特殊字符（`!`/`$` 等）；
   2. 双引号 `" "`：保留大部分特殊字符功能，不推荐处理 `!`。
4. **权限**
   1. `r` 读、`w` 写、`x` 执行；
   2. 直接运行文件 → 必须有 `x` 权限；解释器调用文件 → 仅需 `r` 权限。

# 三、高频易错点

1. 写入含 `!` 的内容时，**必须用单引号**，双引号会报错；
2. `EOF` 结束标记必须单独一行、无前后空格；
3. `./文件名` 执行脚本失败，优先检查是否缺少 `x` 执行权限；
4. `~` 仅代表当前用户家目录，不能在路径中随意省略；
5. macOS 不支持 `/sys` 虚拟文件系统，硬件读取练习无需执行。