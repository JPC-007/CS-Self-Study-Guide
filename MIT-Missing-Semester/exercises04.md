# Exercises

1. Take this [short interactive regex tutorial](https://regexone.com/).
   ```
   done!
   ```

2. Find the number of words (in `/usr/share/dict/words`) that contain at least three `a`s and don’t have a `'s` ending. What are the three most common last two letters of those words? `sed`’s `y` command, or the `tr` program, may help you with case insensitivity. How many of those two-letter combinations are there? And for a challenge: which combinations do not occur?
   ```
   [I] ~ $ echo "1.符合条件单词总数："
   cat /usr/share/dict/words | tr A-Z a-z | awk '/a.*a.*a/ && !/'\''s$/ && length($0)>=2' | wc -l
   1.符合条件单词总数：
       7596
   [I] ~ $ echo -e "\n2.频次最高的3个双字母后缀： "
   cat /usr/share/dict/words | tr A-Z a-z | awk '/a.*a.*a.*a/ && !/'\''s$/ && length($0)>=2' | sed -E 's/.*(..)$/\1/' | sort | uniq -c | sort -nr | head -n3
   
   2.频次最高的3个双字母后缀：
    113 ae
    101 an
    100 al
   
   [I] ~ $ echo -e "\n3.不重复双字母组合总种类："
   cat /usr/share/dict/words | tr A-Z a-z | awk '/a.*a.*a.*a/ && !/'\''s$/ && length($0)>=2' | sed -E 's/.*(..)$/\1/' | sort | uniq | wc -l
   
   3.不重复双字母组合总种类：
         70
         
   [I] ~ $ printf "%s\n" {a..z}{a..z} > all_2letter.txt
   [I] ~ $ cat /usr/share/dict/words \
   > | tr A-Z a-z | awk '/a.*a.*a.*a/ && !/'\''s$/ && length($0)>=2' | sed -E 's/.*(..)$/\1/' | sort | uniq > exist_2letter.txt
   [I] ~ $ comm -23 all_2letter.txt exist_2letter.txt
   ```

3. To do in-place substitution it is quite tempting to do something like `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`. However this is a bad idea, why? Is this particular to `sed`? Use `man sed` to find out how to accomplish this.
   ```
   根源：Shell 重定向 > 的执行优先级高于 sed 命令本身，执行顺序拆解：
   Shell 解析整条命令，先识别输出重定向 > input.txt；
   操作系统打开 input.txt，并直接截断清空该文件（文件内容全部删除，变成空文件）；
   之后才启动 sed 程序，sed 去读取 input.txt 时，文件已经是空的；
   sed 没有任何文本可以处理，最终把空内容写回文件；
   结果：原文件全部数据丢失，变成空白文件。
   
   sed -i.bak 's/old/new/' input.txt
   # 不需要备份时手动删除备份文件
   rm input.txt.bak
   ```

4. Find your average, median, and max system boot time over the last ten boots. Use `journalctl` on Linux and `log show` on macOS, and look for log timestamps near the beginning and end of each boot. On Linux, they may look something like:

   ```
   Logs begin at ...
   ```

   and

   ```
   systemd[577]: Startup finished in ...
   ```

   On macOS, [look for](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/):

   ```
   === system boot:
   ```

   and

   ```
   Previous shutdown cause: 5
   ```

   ```shell
      ➜  ~ journalctl $(journalctl --list-boots | tail -n 10 | awk '{print "-b " $1}') \
   | grep 'systemd\[1\]: Startup finished in'  
   6月 20 10:09:32 jpc-ubuntu systemd[1]: Startup finished in 13.896s (firmware) + 11.110s (loader) + 493ms (kernel) + 2.856s (initrd) + 7.774s (userspace) = 36.131s.
   ➜  ~ journalctl $(journalctl --list-boots | tail -n 10 | awk '{print "-b " $1}') \
   | grep 'systemd\[1\]: Startup finished in' \
   | sed -E 's/.*= ([0-9.]+)s.*/\1/' \
   | grep -E '^[0-9.]+$' \
   | R --no-echo -e '
   times <- scan(file = "stdin", quiet = TRUE)
   cat("====== 最近开机启动耗时统计 ======\n")
   cat("有效样本数：", length(times), "次\n")
   cat("平均值：", mean(times), "秒\n")
   cat("中位数：", median(times), "秒\n")
   cat("最大值：", max(times), "秒\n")
   cat("\n完整五数汇总：\n")
   print(summary(times))
   '
   ====== 最近开机启动耗时统计 ======
   有效样本数： 1 次
   平均值： 36.131 秒
   中位数： 36.131 秒
   最大值： 36.131 秒

   完整五数汇总：
      Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   36.13   36.13   36.13   36.13   36.13   36.13 
   ```

5. Look for boot messages that are not shared between your past three reboots (see journalctl’s -b flag). Break this task down into multiple steps. First, find a way to get just the logs from the past three boots. There may be an applicable flag on the tool you use to extract the boot logs, or you can use sed '0,/STRING/d' to remove all lines previous to one that matches STRING. Next, remove any parts of the line that always varies (like the timestamp). Then, de-duplicate the input lines and keep a count of each one (uniq is your friend). And finally, eliminate any line whose count is 3 (since it was shared among all the boots).
   ```
   journalctl -b 0 -b -1 -b -2 \
   | awk '{$1=$2=$3=$4=""; sub(/^[ \t]+/, ""); print}' \
   | sed -E 's/\[[0-9]+\]/[]/g' \
   | sed -E 's/[0-9]{2}:[0-9]{2}:[0-9]{2}\.[0-9]+/[TIME]/g' \
   | sort \
   | uniq -c \
   | awk '$1 != 3'
   ```
6. Find an online data set like this one, this one, or maybe one from here. Fetch it using curl and extract out just two columns of numerical data. If you’re fetching HTML data, pup might be helpful. For JSON data, try jq. Find the min and max of one column in a single command, and the difference of the sum of each column in another.
   ```
   ➜  ~ curl -s https://raw.githubusercontent.com/fivethirtyeight/data/master/us-weather-history/KNYC.csv | awk -F',' 'NR > 1 {print $3, $4}' | head -n 10
   72 89
   72 91
   69 87
   65 74
   63 81
   66 84
   72 90
   71 91
   71 88
   72 83
   
   ➜  ~ curl -s https://raw.githubusercontent.com/fivethirtyeight/data/master/us-weather-history/KNYC.csv \
   | awk -F',' 'NR > 1 {
      if (min == "" || $3 < min) min = $3
      if (max == "" || $3 > max) max = $3
   }
   END {
      print "最低温最小值：", min, "°F"
      print "最低温最大值：", max, "°F"
   }'
   最低温最小值： 2 °F
   最低温最大值： 77 °F
   ➜  ~ curl -s https://raw.githubusercontent.com/fivethirtyeight/data/master/us-weather-history/KNYC.csv \ 
   | awk -F',' 'NR > 1 {
     if (min == "" || $4 < min) min = $4
     if (max == "" || $4 > max) max = $4
   }
   END {
     print "最高温最小值：", min, "°F"
     print "最高温最大值：", max, "°F"
   }'
   最高温最小值： 19 °F
   最高温最大值： 92 °F
   
   ➜  ~ curl -s https://raw.githubusercontent.com/fivethirtyeight/data/master/us-weather-history/KNYC.csv \
   | awk -F',' 'NR > 1 {
     sum_min += $3
     sum_max += $4
   }
   END {
     print "最低温总和", sum_min
     print "最高温总和", sum_max
     print "两列总和差值", sum_max - sum_min
   }'
   最低温总和 17245
   最高温总和 22533
   两列总和差值 5288

# 笔记

## 一、课程核心思想

数据整理（Data Wrangling）的本质是**将原始、杂乱的数据，通过一系列工具的组合转换为目标格式**。核心方法论是通过管道 `|` 串联工具，每一步只做一件事，逐步迭代得到想要的结果。

### 典型场景：远程日志分析的优化

```bash
# 低效写法：全量拉取到本地再过滤，浪费带宽
ssh myserver journalctl | grep sshd | grep "Disconnected from"

# 高效写法：在远端完成过滤，只传输结果
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
less ssh.log
```

> 核心原则：尽可能在数据源端完成过滤，减少无效数据传输。

## 二、正则表达式（Regular Expressions）

正则是文本匹配与提取的核心基础，sed、awk、grep 等工具都依赖正则实现复杂逻辑。

### 2.1 正则速查表

#### 一、字符类匹配

| 符号     | 含义                                       |
| -------- | ------------------------------------------ |
| `abc`    | 匹配字面字母 abc（普通字符按原样匹配）     |
| `123`    | 匹配字面数字 123                           |
| `.`      | 匹配任意单个字符（换行符除外）             |
| `.`      | 匹配字面句号 `.`（转义特殊字符）           |
| `\d`     | 匹配任意数字                               |
| `\D`     | 匹配任意非数字字符                         |
| `\w`     | 匹配任意字母、数字、下划线（单词字符）     |
| `\W`     | 匹配任意非单词字符                         |
| `\s`     | 匹配任意空白字符（空格、制表符、换行等）   |
| `\S`     | 匹配任意非空白字符                         |
| `[abc]`  | 匹配字符集中的任意一个（a、b、c 任选其一） |
| `[^abc]` | 匹配不在字符集中的任意字符（取反）         |
| `[a-z]`  | 匹配 a 到 z 范围内的任意小写字母           |
| `[0-9]`  | 匹配 0 到 9 范围内的任意数字               |

#### 二、量词（重复次数）

| 符号    | 含义                                 |
| ------- | ------------------------------------ |
| `{m}`   | 前一个字符恰好重复 m 次              |
| `{m,n}` | 前一个字符重复 m 到 n 次             |
| `*`     | 前一个字符重复零次或多次             |
| `+`     | 前一个字符重复一次或多次             |
| `?`     | 前一个字符为可选（出现 0 次或 1 次） |

#### 三、位置锚定

| 符号  | 含义                           |
| ----- | ------------------------------ |
| `^`   | 匹配行的开头                   |
| `$`   | 匹配行的结尾                   |
| `^…$` | 完整匹配从开头到结尾的整行内容 |

#### 四、分组与逻辑

| 符号      | 含义                                                    |                         |
| --------- | ------------------------------------------------------- | ----------------------- |
| `(…)`     | 捕获组，括号内的匹配内容会被保存，可通过 `\1` `\2` 引用 |                         |
| `(a(bc))` | 嵌套捕获子组，外层为第 1 组，内层为第 2 组              |                         |
| `(.*)`    | 捕获任意长度的全部内容                                  |                         |
| `(abc     | def)`                                                   | 或逻辑，匹配 abc 或 def |

### 2.2 关键特性：贪婪匹配

`.*` 和 `.+` 默认是**贪婪模式**，会匹配尽可能长的文本。

- sed 原生不支持非贪婪匹配，复杂场景可改用 `perl -pe 's/.*?pattern//'`

### 2.3 捕获组用法

用 `()` 包裹的匹配内容会被保存为编号捕获组，替换时通过 `\1` `\2` `\3` 按序号引用。 **示例：从 SSH 日志中提取用户名**

```bash
sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

其中 `\2` 对应第二个捕获组，也就是目标用户名。

### 2.4 sed 正则兼容注意事项

- sed 默认使用**基础正则（BRE）**，`+ ? () {} |` 等元字符需要加反斜杠转义才生效
- 加 `-E` 参数启用**扩展正则（ERE）**，语法更通用，推荐优先使用
- `\d` `\w` `\s` 这类 PCRE 风格的转义符，sed 原生不支持，需用 `[0-9]` `[a-zA-Z0-9_]` `[[:space:]]` 替代

## 三、核心工具详解

### 3.1 sed - 流编辑器

sed 是逐行处理文本的流编辑器，最常用的是 `s` 替换命令，格式为：

```Plain
s/正则匹配/替换内容/
```

其他常用能力：

- 插入文本（`i` 命令）
- 打印指定行（`p` 命令，配合 `-n` 使用）
- 按行号 / 正则选择处理范围

### 3.2 awk - 流式文本编程语言

awk 是专为文本流设计的轻量编程语言，基本结构为：`模式 { 执行动作 }`

- 默认按空白字符分割字段，`$1` ~ `$n` 对应第 n 列，`$0` 代表整行
- `-F` 参数指定分隔符，如 `-F','` 适配 CSV 格式
- `BEGIN {}`：处理输入前执行，用于初始化变量
- `END {}`：所有行处理完成后执行，用于输出最终结果

**示例：统计单次出现、以 c 开头 e 结尾的用户名数量**

```bash
awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += 1 } END { print rows }'
```

### 3.3 排序与去重

- `sort`：文本排序；`-n` 数值排序；`-r` 倒序；`-k1,1` 仅按第一列排序
- `uniq -c`：合并相邻的重复行，前缀标注出现次数
- **经典组合**：`sort | uniq -c` 统计每条内容的出现次数

### 3.4 文本拼接与统计

- `paste -sd,`：将多行内容合并为一行，用逗号分隔
- `wc -l`：统计文本行数

### 3.5 数值统计与可视化

1. **bc - 命令行计算器** 支持从标准输入读取数学表达式，示例：对一列数字求和
   1. `数据 | paste -sd+ | bc -l`
2. **R - 统计分析** 可直接读取标准输入，输出均值、中位数、最值等统计量
   1. `数据 | R --no-echo -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'`
3. **gnuplot - 命令行绘图** 支持直接从管道读取数据生成图表

### 3.6 xargs - 批量参数传递

将标准输入的内容转换为命令行参数，配合数据整理实现批量操作。 **示例：批量卸载旧版 Rust 工具链**

```bash
rustup toolchain list | grep nightly | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

### 3.7 二进制数据处理

管道不仅支持文本，同样可以处理二进制数据流。 **示例：摄像头采集 → 灰度处理 → 压缩 → 远程展示**

```bash
ffmpeg -i /dev/video0 -frames 1 -f image2 - \
 | convert - -colorspace gray - \
 | gzip \
 | ssh mymachine 'gzip -d | tee copy.jpg | feh -'
```

## 四、课后习题完整解析

### 习题 1：交互式正则练习

通过在线正则练习网站巩固语法，重点掌握贪婪匹配、捕获组、字符集的使用。

### 习题 2：系统字典单词统计

**题目**：统计 `/usr/share/dict/words` 中满足「至少含 3 个字母 a、不以 `'s` 结尾」的单词；统计这些单词末尾两个字母的出现频率。 **分步实现**：

```bash
cat /usr/share/dict/words \
| tr '[:upper:]' '[:lower:]' \          # 全部转小写，消除大小写差异
| grep -E '.*a.*a.*a.*' \               # 过滤含至少3个a的单词
| grep -v "'s$" \                       # 排除以's结尾的单词
| sed -E 's/.*(..)$/\1/' \              # 提取末尾两个字母
| sort | uniq -c \                      # 统计出现次数
| sort -rn                              # 按次数倒序排列
```

### 习题 3：sed 原地修改的陷阱

**问题**：为什么 `sed s/REGEX/SUB/ input.txt > input.txt` 是错误写法？ **原因**：shell 会先执行重定向 `> input.txt`，直接清空原文件；sed 再读取时文件已为空，最终数据全部丢失。

> 该问题不是 sed 独有，所有支持重定向的命令都不能用 `command file > file` 的写法。 **正确做法**：使用 sed 内置的原地修改参数 `-i`

```bash
sed -i 's/old/new/g' input.txt
```

### 习题 4：系统开机时间统计

**题目**：提取最近 10 次开机的启动耗时，计算平均值、中位数、最大值。 **核心思路**：

1. `journalctl -b` 批量提取多次开机日志
2. 过滤 PID=1 的系统级启动日志（排除用户会话干扰）
3. sed 提取耗时数值
4. R 计算统计指标 **完整可用命令（自动适配开机次数不足 10 次的场景）**：

```bash
journalctl $(journalctl --list-boots | awk '{print "-b " $1}') \
| grep 'systemd\[1\]: Startup finished in' \
| sed -E 's/.*= ([0-9.]+)s.*/\1/' \
| grep -E '^[0-9.]+$' \
| R --no-echo -e '
times <- scan(file="stdin", quiet=TRUE)
cat("有效样本数：", length(times), "次\n")
cat("平均值：", mean(times), "秒\n")
cat("中位数：", median(times), "秒\n")
cat("最大值：", max(times), "秒\n")
'
```

### 习题 5：三次开机的非共享日志

**题目四步框架**：

1. 获取最近 3 次开机日志
2. 去除每次都会变化的字段（时间戳、PID 等）
3. 去重并统计每条日志的出现次数
4. 过滤掉出现次数 = 3 的共有日志 **优化后完整命令（适配中文 Ubuntu 环境）**：

```bash
journalctl -b 0 -b -1 -b -2 \
| awk '{$1=$2=$3=$4=""; sub(/^[ \t]+/, ""); print}' \   # 删除前4列（时间、主机名）
| sed -E 's/\[[0-9]+\]/[]/g' \                          # 统一替换进程PID
| sed -E 's/[0-9]{2}:[0-9]{2}:[0-9]{2}\.[0-9]+/[TIME]/g' \ # 替换应用内嵌时间戳
| sort | uniq -c \                                      # 去重计数
| awk '$1 != 3'                                         # 过滤共有日志
```

### 习题 6：在线数据集数值统计

**题目**：用 curl 获取在线数据集，提取两列数值；单条命令计算一列的最值，以及两列总和的差值。 **示例数据集**：FiveThirtyEight 纽约天气 CSV 数据集

```bash
# 1. 单列最值计算（最高温列）
curl -s https://raw.githubusercontent.com/fivethirtyeight/data/master/us-weather-history/KNYC.csv \
| awk -F',' 'NR>1{
    if(min=="" || $4<min) min=$4
    if(max=="" || $4>max) max=$4
} END{print "最低值:", min; print "最高值:", max}'

# 2. 两列总和的差值
curl -s https://raw.githubusercontent.com/fivethirtyeight/data/master/us-weather-history/KNYC.csv \
| awk -F',' 'NR>1{sum_min+=$3; sum_max+=$4} END{print "两列总和差值:", sum_max - sum_min}'
```

## 五、实操避坑指南（实战踩坑总结）

### 5.1 zsh 环境兼容问题

- **现象**：`$(...)` 命令替换的结果不会按空格拆分参数，导致多参数命令失效
- **解决方案**：使用 `${=$(...)}` 强制按单词分割
  - `journalctl ${=$(journalctl --list-boots | tail -n 10 | awk '{print "-b " $1}')}`

### 5.2 引号闭合陷阱

- **现象**：命令行出现 `quote>` `pipe quote>` 提示符，命令不执行
- **原因**：单引号 / 双引号 / 括号未闭合，shell 认为命令未写完
- **解决**：补全闭合符号，或按 `Ctrl+C` 取消后重新输入
- **注意**：awk 多行写法结尾必须是 `}'`，先闭合大括号再闭合单引号

### 5.3 复制粘贴的隐藏问题

- **现象**：粘贴命令后出现 `bad pattern: ^[[200~` 报错
- **原因**：复制时带入了终端「括号粘贴模式」的不可见控制字符
- **解决**：关键命令手动输入，或粘贴前清理格式

### 5.4 日志清洗的粒度原则

- 清洗目标：让「本质逻辑相同的日志」文本完全一致，保证去重统计准确
- 必清字段：系统时间戳、主机名、进程 PID
- 可选清洗：应用内嵌时间、线程号、DBus 连接号、IP 地址、哈希值等，按需调整粒度

### 5.5 sed 正则适配性

不同系统、不同版本的日志格式存在差异，不要写死正则。优先使用宽松匹配，从左到右逐段调试管道命令，每一步验证输出后再追加下一段逻辑。