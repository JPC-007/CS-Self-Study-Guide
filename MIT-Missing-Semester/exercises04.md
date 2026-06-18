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

   ```
   
   ```

5. Look for boot messages that are *not* shared between your past three reboots (see `journalctl`’s `-b` flag). Break this task down into multiple steps. First, find a way to get just the logs from the past three boots. There may be an applicable flag on the tool you use to extract the boot logs, or you can use `sed '0,/STRING/d'` to remove all lines previous to one that matches `STRING`. Next, remove any parts of the line that *always* varies (like the timestamp). Then, de-duplicate the input lines and keep a count of each one (`uniq` is your friend). And finally, eliminate any line whose count is 3 (since it *was* shared among all the boots).

6. Find an online data set like [this one](https://commons.wikimedia.org/wiki/Data:Wikipedia_statistics/data.tab), [this one](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1), or maybe one [from here](https://www.springboard.com/blog/data-science/free-public-data-sets-data-science-project/). Fetch it using `curl` and extract out just two columns of numerical data. If you’re fetching HTML data, [`pup`](https://github.com/EricChiang/pup) might be helpful. For JSON data, try [`jq`](https://stedolan.github.io/jq/). Find the min and max of one column in a single command, and the difference of the sum of each column in another.