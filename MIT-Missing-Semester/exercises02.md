# Exercises

1. Read [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) and write an `ls` command that lists files in the following manner

   - Includes all files, including hidden files
   - Sizes are listed in human readable format (e.g. 454M instead of 454279954)
   - Files are ordered by recency
   - Output is colorized

   A sample output would look like this

   ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
   ```

   ``````
   ls -laht --color=auto
   ``````

2. Write bash functions `marco` and `polo` that do the following. Whenever you execute `marco` the current working directory should be saved in some manner, then when you execute `polo`, no matter what directory you are in, `polo` should `cd` you back to the directory where you executed `marco`. For ease of debugging you can write the code in a file `marco.sh` and (re)load the definitions to your shell by executing `source marco.sh`.
   ``````
   #!/bin/bash
   
   # 函数 marco：保存当前工作目录到环境变量
   marco() {
     # 将当前目录（pwd）存入变量 MARCO_DIR
     export MARCO_DIR=$(pwd)
     # 打印提示，方便调试
     echo "✅ 已保存目录：$MARCO_DIR"
   }
   
   # 函数 polo：切换到 marco 保存的目录
   polo() {
     # 容错：如果没执行过 marco，提示错误
     if [ -z "$MARCO_DIR" ]; then
       echo "❌ 错误：请先执行 marco 保存目录！"
       return 1
     fi
   
     # 切换目录（引号处理带空格的路径，更安全）
     cd "$MARCO_DIR" || return 1
     echo "✅ 已回到目录：$MARCO_DIR"
   }
   ``````

3. Say you have a command that fails rarely. In order to debug it you need to capture its output but it can be time consuming to get a failure run. Write a bash script that runs the following script until it fails and captures its standard output and error streams to files and prints everything at the end. Bonus points if you can also report how many runs it took for the script to fail.

   ```
    #!/usr/bin/env bash
   
    n=$(( RANDOM % 100 ))
   
    if [[ n -eq 42 ]]; then
       echo "Something went wrong"
       >&2 echo "The error was using magic numbers"
       exit 1
    fi
   
    echo "Everything went according to plan"
   ```

   ```
   #!/bin/bash
   
   # 1. 初始化运行次数计数器
   count=0
   
   # 2. 清空旧日志（保证每次运行都是全新的日志）
   > stdout.log
   > stderr.log
   
   # 3. 无限循环，直到目标脚本失败
   while true; do
       # 每次循环，计数+1
       ((count++))
   
       # 4. 运行目标脚本
       # stdout 追加写入 stdout.log
       # stderr 追加写入 stderr.log
       ./random.sh >> stdout.log 2>> stderr.log
   
       # 5. 判断上一条命令是否失败（退出码≠0）
       if [ $? -ne 0 ]; then
           break  # 失败了，跳出循环
       fi
   done
   
   # 6. 打印最终结果
   echo "✅ 脚本在第 $count 次运行时失败！"
   echo -e "\n===== 标准输出（stdout）====="
   cat stdout.log
   echo -e "\n===== 标准错误（stderr）====="
   cat stderr.log
   ```

4. As we covered in the lecture `find`’s `-exec` can be very powerful for performing operations over the files we are searching for. However, what if we want to do something with **all** the files, like creating a zip file? As you have seen so far commands will take input from both arguments and STDIN. When piping commands, we are connecting STDOUT to STDIN, but some commands like `tar` take inputs from arguments. To bridge this disconnect there’s the [`xargs`](https://www.man7.org/linux/man-pages/man1/xargs.1.html) command which will execute a command using STDIN as arguments. For example `ls | xargs rm` will delete the files in the current directory.

   Your task is to write a command that recursively finds all HTML files in the folder and makes a zip with them. Note that your command should work even if the files have spaces (hint: check `-d` flag for `xargs`).

   If you’re on macOS, note that the default BSD `find` is different from the one included in [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands). You can use `-print0` on `find` and the `-0` flag on `xargs`. As a macOS user, you should be aware that command-line utilities shipped with macOS may differ from the GNU counterparts; you can install the GNU versions if you like by [using brew](https://formulae.brew.sh/formula/coreutils).

   ```
   find . -type f -name "*.html" -print0 | xargs -0 zip html_files.zip
   ```

5. (Advanced) Write a command or script to recursively find the most recently modified file in a directory. More generally, can you list all files by recency?

   ```
   find . -type f -print0 | xargs -0 ls -lt
   find . -type f -print0 | xargs -0 ls -lt | head -n 1
   
   # 所有文件：最旧 → 最新
   find . -type f -print0 | xargs -0 ls -ltr
   ```



# 笔记

# 一、Ls 定制输出考点（第一题）

## 1. 万能最终命令

```
ls -laht --color=auto
```

## 2. 参数对应需求

- **-l**：长格式输出（权限、用户、大小、时间）
- **-a**：显示所有文件（隐藏文件、. .. 目录）
- **-h**：人类可读大小（K/M/G）
- **-t**：按修改时间排序（最新在上）
- **--color=auto**：彩色输出

# 二、Marco & Polo 函数 + 你所有疑问

## 1. 核心功能

- `marco`：保存**当前终端所在目录**
- `polo`：切回 marco 保存的目录

## 2. 完整核心代码

```Plain
marco() {
  export MARCO_DIR=$(pwd)
}

polo() {
  if [ -z "$MARCO_DIR" ]; then
    echo "请先执行 marco"
    return 1
  fi
  cd "$MARCO_DIR"
}
```

## 3. 疑问 1：[ -z "$MARCO_DIR" ] 是什么？

- **-z** = zero 零长度
- 含义：**判断变量是否为空**
- 逻辑：没运行过 marco → 变量为空 → 提示报错
- 必记：`[ ]` 内部**必须带空格**

## 4. 疑问 2：marco 保存的是哪个路径？

**只保存：你执行 marco 命令时，终端当前所在目录**

和 marco.sh 文件放在哪里 **完全无关**

## 5. 疑问 3：为什么脚本是 Bash，后缀是 .sh？

- Linux **不识别后缀**，后缀是给人看的
- **.sh** = Shell Script 通用后缀
- 真正决定脚本类型的是第一行 shebang：`#!/bin/bash`

## 6. 疑问 4：source 是否必须在脚本目录执行？

**不需要！**

- `source marco.sh` → 只能当前目录找文件
- `source ~/Desktop/marco.sh` → **任意目录都能加载**（绝对路径）

## 7. 疑问 5：为什么不用每次 source？

- source 是**加载到当前终端会话**
- 终端不关闭，函数一直缓存可用
- **关终端必须重新 source**

## 8. 关键原理：source 和 ./ 运行的区别

- `./marco.sh`：子 Shell 运行，函数运行完就丢 ❌
- `source marco.sh`：当前 Shell 运行，永久留存 ✅

# 三、循环重试脚本（随机报错脚本）

## 1. 需求

循环运行脚本，**直到它失败**，保存 stdout、stderr，统计运行次数

## 2. 核心考点

- `$?`：上一条命令退出码，**0成功 非0失败**
- `>` 覆盖 / `>>` 追加
- `2>` 标准错误重定向
- `while true` 无限循环 + `break` 退出

## 3. 关键语法

- `((count++))` 计数自增
- `> file.log` 快速清空文件
- `if [ $? -ne 0 ]` 如果命令失败

# 四、Find + Xargs 高阶必考（重难点）

## 1. 题目核心痛点

普通管道只能传文本，**zip/tar 需要文件名当参数**，必须用 xargs 转换

## 2. 最大坑：文件名带空格

普通 find 会把 `my file.html` 识别成两个文件

**固定解决方案（Linux/Mac 通用）**

- `find -print0`：空字符分隔文件
- `xargs -0`：按空字符解析文件名

## 3. 必考命令1：递归打包所有 html（兼容空格）

```Plain
find . -type f -name "*.html" -print0 | xargs -0 zip html.zip
```

## 4. 必考命令2：递归按修改时间排序

- 所有文件从新到旧：`find . -type f -print0 | xargs -0 ls -lt`
- 只找最新文件：`find . -type f -print0 | xargs -0 ls -lt | head -n 1`

## 5. 你疑惑的 touch 测试命令解析

```Plain
touch test.html "my file.html" sub/index.html
```

- `test.html`：普通文件
- `"my file.html"`：**带空格文件必须加引号**
- `sub/index.html`：子目录文件（需要先 mkdir sub）

## 6. 为什么不用 find -exec 打包？

**-exec 会逐条执行 zip**，每次覆盖压缩包，最后只剩一个文件 ❌

**xargs 一次性传入所有文件**，正常打包 ✅

# 五、Vim 粘贴代码去空格技巧

## 1. 粘贴后批量去行首空格

Vim 普通模式：`:%s/^[[:space:]]*//`

## 2. 粘贴前防自动缩进

`:set paste` 粘贴 → `:set nopaste` 恢复

# 六、终极必考速记清单

- 判断空变量：`[ -z "$VAR" ]`
- 加载函数：`source xxx.sh`
- 空格文件兼容：`-print0 + xargs -0`
- 判断命令成功：`$? -eq 0`
- 时间排序：`ls -lt`
- 无限循环：`while true; do ... done`