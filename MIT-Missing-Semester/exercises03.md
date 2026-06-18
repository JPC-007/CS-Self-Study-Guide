# Exercises

1. Complete `vimtutor`. Note: it looks best in a [80x24](https://en.wikipedia.org/wiki/VT100) (80 columns by 24 lines) terminal window.
   ```
   done!
   ```

2. Download our [basic vimrc](https://missing.csail.mit.edu/2020/files/vimrc) and save it to `~/.vimrc`. Read through the well-commented file (using Vim!), and observe how Vim looks and behaves slightly differently with the new config.
   ```
   done!
   ```

3. Install and configure a plugin: [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim).

   1. Create the plugins directory with `mkdir -p ~/.vim/pack/vendor/start`
   2. Download the plugin: `cd ~/.vim/pack/vendor/start; git clone https://github.com/ctrlpvim/ctrlp.vim`
   3. Read the [documentation](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md) for the plugin. Try using CtrlP to locate a file by navigating to a project directory, opening Vim, and using the Vim command-line to start `:CtrlP`.
   4. Customize CtrlP by adding [configuration](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options) to your `~/.vimrc` to open CtrlP by pressing Ctrl-P.

   ```
   done!
   ```

4. To practice using Vim, re-do the [Demo](https://missing.csail.mit.edu/2020/editors/#demo) from lecture on your own machine.
   ```python
   import sys
   
   def fizz_buzz(limit):
       for i in range(1, limit + 1):
           if i % 15 == 0:
               print('fizzbuzz')
           elif i % 3 == 0:
               print('fizz')
           elif i % 5 == 0:
               print('buzz')
           else:
               print(i)
   
   def main():
       fizz_buzz(int(sys.argv[1]))
   
   if __name__ == "__main__":
       main()
   ```

5. Use Vim for *all* your text editing for the next month. Whenever something seems inefficient, or when you think “there must be a better way”, try Googling it, there probably is. If you get stuck, come to office hours or send us an email.
   ```
   I'll have a try.
   ```

6. Configure your other tools to use Vim bindings (see instructions above).
   ```
   I'll have a try.
   ```

7. Further customize your `~/.vimrc` and install more plugins.

8. (Advanced) Convert XML to JSON ([example file](https://missing.csail.mit.edu/2020/files/example-data.xml)) using Vim macros. Try to do this on your own, but you can look at the [macros](https://missing.csail.mit.edu/2020/editors/#macros) section above if you get stuck.

能理解。这题比上一题更“Vim 味儿”一点：它不是让你写 XML 转 JSON 程序，而是让你用 **Vim macros** 把一个固定格式的 XML 文件批量改成 JSON。

Missing Semester 官方这一节的练习要求是：下载 `example-data.xml`，然后用 Vim 宏把它转成“由对象组成的数组”，每个对象里有 `name` 和 `email` 两个字段。课程页面也强调这里练的是 Vim 的 macro：`q{character}` 开始录制，`q` 停止，`@{character}` 回放，`999@q` 这种可以重复执行宏直到出错停止。([Missing Semester](https://missing.csail.mit.edu/2020/editors/?utm_source=chatgpt.com))

你可以这样做：

```bash
wget https://missing.csail.mit.edu/2020/files/example-data.xml
cp example-data.xml example-data.json
vim example-data.json
```

原始文件大概结构是：

```xml
<people>
  <person>
    <name>...</name>
    <email>...</email>
  </person>
  <person>
    <name>...</name>
    <email>...</email>
  </person>
</people>
```

目标大概是变成：

```json
[
  {
    "name": "...",
    "email": "..."
  },
  {
    "name": "...",
    "email": "..."
  }
]
```

课堂给的宏思路是这样的。

先删掉 XML 最外层的 `<people>` 和 `</people>`：

```vim
Gdd
ggdd
```

然后移动到第一行 `<name>`，录制一个“把一行 XML 标签改成 JSON 字段”的宏，存在寄存器 `e`：

```vim
qe^r"f>s": "<Esc>f<C"<Esc>q
```

注意：这里的 `<Esc>` 不是让你输入字符 `<`、`E`、`s`、`c`，而是真的按键盘上的 Esc。

这个宏的效果是把：

```xml
    <name>John Doe</name>
```

变成：

```json
    "name": "John Doe"
```

它同样也能把：

```xml
    <email>john@example.com</email>
```

变成：

```json
    "email": "john@example.com"
```

然后移动到第一行 `<person>`，录制一个“处理一个 person 块”的宏，存在寄存器 `p`：

```vim
qpS{<Esc>j@eA,<Esc>j@ejS},<Esc>q
```

它的意思是：

```text
把 <person> 这一行改成 {
下一行运行 @e，把 name 改成 JSON 字段，然后行尾加逗号
下一行运行 @e，把 email 改成 JSON 字段
下一行把 </person> 改成 },
```

然后录制一个“处理当前 person，并移动到下一个 person”的宏，存在寄存器 `q`：

```vim
qq@pjq
```

最后批量执行：

```vim
999@q
```

Vim 宏执行到文件末尾时会因为移动失败而停止，这正是这里利用的机制。

最后手动修一下 JSON：

```vim
G$x
ggO[<Esc>
Go]<Esc>
:wq
```

含义是：

```text
G$x       到最后一行，删掉最后一个多余逗号
ggO[      文件顶部插入 [
Go]       文件底部插入 ]
:wq       保存退出
```

最后可以用这个检查 JSON 是否合法：

```bash
python -m json.tool example-data.json
```

所以这道题的本质是：

> 用 Vim 的宏录制一组重复编辑动作，然后把很多重复的 XML 块批量转换成 JSON。

这题标了 **Advanced**，所以一开始看懵很正常。你真要练的话，不建议上来就背完整宏；更好的方式是先手动改一个 `<name>` 行，感觉每个 Vim 操作在干嘛，然后再录成 macro。



# 笔记

> 本笔记整合本次所有练习任务：Vim 基础命令、`vimtutor`、配置文件、插件安装、全工具 Vim 键位、代码编辑练习、Vim 宏高阶实战，按任务顺序整理，兼顾**实操步骤、核心命令、考点、易错点**，用于复习与作业复盘。

# 一、Vim 基础命令（入门核心）

## 通用语法规则

Vim 普通模式标准格式：`[次数] + [操作符] + [选区/位移]`

1. 仅「次数 + 位移」：只移动光标，无编辑动作
2. 「操作符 + 选区 / 位移」：执行增 / 删 / 改；前缀加数字代表重复执行

- 常用操作符：`d`(删除)、`c`(修改，删内容并进入插入模式)、`y`(复制)
- 文本对象：`i`(inner，内部，不含包裹符号)、`a`(around，环绕，含包裹符号)
- 位移：`w` 按单词移动

## 常用命令详解

| 命令  | 功能说明                  | 补充说明                                                     |
| ----- | ------------------------- | ------------------------------------------------------------ |
| `3w`  | 光标**向后移动 3 个单词** | `w` 跳到下一个单词首字符，标点单独分词                       |
| `7dw` | 向后删除 7 个单词         | `dw` 删除当前剩余字符 + 下一个完整单词；等价连续执行 7 次`dw` |
| `ci(` | 修改括号`()`内部内容      | 删除括号内所有内容，**保留括号**，自动进入插入模式           |
| `da'` | 删除整串单引号字符串      | 连带两侧单引号 + 内部内容全部删除                            |

### 拓展区分（易混命令）

- `daw`：删除光标所在**整个单词**（光标在词中 / 词首都生效）
- `ca(`：删除括号 + 内部所有内容（区别于 `ci(`）
- `di'`：仅删除单引号内部文字，保留引号本身（区别于 `da'`）

# 二、任务 1：完成 `vimtutor` 官方教程

## 任务要求

完整学习 Vim 自带入门教程；建议在 **80 列 × 24 行** 终端窗口下使用，排版最优。

## 实操命令

1. 启动英文原版教程（默认）
   1. `vimtutor`
2. 启动中文教程
   1. `vimtutor zh`

## 教程结构 & 对应知识点

教程共 7 个课时，本次所学命令对应位置：

1. 单词移动、删除命令（`3w`/`7dw`）→ Lesson 2
2. 文本对象（括号、引号操作 `ci(`/`da'`）→ Lesson 5

## 操作说明

- 按照页面提示修改示例文本即可，无需手动保存；学习完毕输入 `:q` 退出。
- 调整终端：`stty cols 80 rows 24`（临时设置窗口尺寸）。

# 三、任务 2：部署基础 `~/.vimrc` 配置文件

## 任务要求

1. 下载基础 Vim 配置文件，保存为 `~/.vimrc`
2. 使用 Vim 通读注释版配置文件
3. 观察配置前后 Vim 界面、行为变化

## 1. 前置操作（备份原有配置）

防止旧配置丢失，执行备份：

```bash
cp ~/.vimrc ~/.vimrc.bak
```

恢复旧配置：`cp ~/.vimrc.bak ~/.vimrc`

## 2. 下载配置文件（二选一，macOS 环境）

- 方式 1（`curl`，mac 原生自带）
  - `curl -o ~/.vimrc https://raw.githubusercontent.com/learn-vim/learn-vi-rs/master/files/basic.vimrc`
- 方式 2（`wget`）
  - `wget https://raw.githubusercontent.com/learn-vim/learn-vi-rs/master/files/basic.vimrc -O ~/.vimrc`

## 3. 通读配置文件

1. 打开文件：
   1. `vim ~/.vimrc`
2. 浏览命令：
   1. `j/k`：逐行移动光标
   2. `Ctrl+f/Ctrl+b`：整页翻页
   3. `/关键词`：搜索配置项（如 `/set`）
3. 阅读完成退出：`Esc` → `:q`

## 4. 核心配置效果（配置前后差异）

| 配置指令                    | 生效效果                       |
| --------------------------- | ------------------------------ |
| `set number`                | 左侧显示行号（默认关闭）       |
| `set cursorline`            | 高亮光标所在行                 |
| `set mouse=a`               | 开启鼠标点击、选中文本功能     |
| `set ignorecase + hlsearch` | 搜索忽略大小写、匹配内容高亮   |
| `set tabstop=4 + expandtab` | Tab 键转为 4 个空格            |
| `set colorcolumn=80`        | 第 80 列显示竖线，规范代码行宽 |

## 补充命令

- 重载配置（不重启 Vim）：`:source ~/.vimrc`
- 临时使用 Vim 默认配置（跳过 `.vimrc`）：`vim -u NONE`

# 四、任务 3：安装 & 配置 `ctrlp.vim` 文件检索插件

## 任务要求

使用 Vim8+ 原生 `pack` 插件管理器安装插件，配置快捷键 `Ctrl+P` 一键唤起。

## 1. 创建插件目录（Vim 标准自动加载目录）

```bash
mkdir -p ~/.vim/pack/vendor/start
```

- `-p`：递归创建目录，目录已存在也不会报错。

## 2. 克隆插件源码

```bash
cd ~/.vim/pack/vendor/start
git clone https://github.com/ctrlpvim/ctrlp.vim
```

> 报错 `git: command not found`：mac 安装命令行工具 `xcode-select --install`

## 3. 查看插件官方文档

Vim 内置帮助系统：

```vim
:help ctrlp
```

浏览后 `:q` 退出帮助。

## 4. 插件基础使用

1. 进入任意项目目录，启动 Vim：`vim`
2. 手动唤起插件：`:CtrlP`
3. 操作：输入文件名模糊搜索、`↑/↓` 选择、`Enter` 打开文件、`Esc` 关闭面板。

## 5. 配置快捷键（`Ctrl+P` 唤起）

1. 打开配置文件：`vim ~/.vimrc`
2. 文件末尾添加映射规则：
   1. `nnoremap <C-p> :CtrlP<CR>`
   2. `nnoremap`：普通模式安全映射；`<C-p>` = `Ctrl+P`；`<CR>` = 回车键
3. `Esc` → `:wq` 保存退出，重启 Vim 生效。

# 五、任务 4：课堂 Demo 练习 — Vim 编辑 FizzBuzz 代码

## 任务要求

全程使用 Vim 编辑，修复代码 5 个问题，练习基础编辑命令。

### 原始问题代码

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print('fizz')
        if i % 5 == 0:
            print('fizz')
        if i % 3 and i % 5:
            print(i)

def main():
    fizz_buzz(10)
```

### 5 个待修复问题

1. `main` 函数未调用
2. 循环从 0 开始，要求从 1 开始
3. 15 的倍数分行输出 fizz/buzz
4. 5 的倍数错误输出 `fizz`（应为 `buzz`）
5. 硬编码参数 `10`，改为命令行传参

### 核心 Vim 操作（练习重点）

`gg`(文件头)、`G`(文件尾)、`o/O`(新建行)、`ciw`(修改单词)、`f字符`(字符跳转)、`cw`(修改到词尾)

### 最终修复代码 + 运行测试

```python
import sys

def fizz_buzz(limit):
    for i in range(1, limit + 1):
        if i % 15 == 0:
            print('fizzbuzz')
        elif i % 3 == 0:
            print('fizz')
        elif i % 5 == 0:
            print('buzz')
        else:
            print(i)

def main():
    fizz_buzz(int(sys.argv[1]))

if __name__ == "__main__":
    main()
```

运行命令：`python3 fizz_buzz.py 15`

# 六、任务 5：全平台工具配置 Vim 按键绑定

## 任务要求

所有日常工具开启 Vim 编辑模式，统一 `hjkl` 光标移动、普通 / 插入模式习惯。

## 1. Shell（mac 默认 Zsh）

### 开启 Vi/Vim 编辑模式

1. 编辑配置：`vim ~/.zshrc`
2. 添加配置（含模式提示符、优化按键延迟）：
   1. `set -o vi set KEYTIMEOUT=1 function zle-line-init zle-keymap-select {  case $KEYMAP in    vicmd)      PROMPT="%F{red}[N]%f %~ $ " ;;    viins|main) PROMPT="%F{green}[I]%f %~ $ " ;;  esac } zle -N zle-line-init zle -N zle-keymap-select`
3. 生效：`source ~/.zshrc`

### 使用规则

- 默认 `[I]` 插入模式（正常输入）；按 `Esc` 切 `[N]` 普通模式（`hjkl` 移动）。

## 2. Git 配置

1. 设置默认编辑器为 Vim：
   1. `git config --global core.editor "vim"`
2. `git log`/`git diff` 分页器 `less` **原生支持 Vim 按键**，无需额外配置。

## 3. VS Code

1. 扩展商店安装插件：`Vim`（作者 vscodevim）
2. 可选优化（`settings.json`）：
   1. `{  "vim.useSystemClipboard": true }`

## 4. JetBrains 系列 IDE (IDEA/PyCharm)

顶部菜单 `Tools` → 勾选 `Vim Emulation`，内置 Vim 模拟器。

## 5. 其他工具

- `man`/`less` 命令：原生支持 Vim 翻页、搜索、退出（`q`）
- Tmux：配置 `~/.tmux.conf` 实现 `Ctrl+h/j/k/l` 切换窗格
- 浏览器：Chrome 安装 `Vimium`、Safari 安装 `Vimari`，键盘导航网页

# 七、任务 6：深度定制 `~/.vimrc` + 安装扩展插件

## 1. 增强版 `~/.vimrc`（完整配置 + 释义）

打开文件 `vim ~/.vimrc`，追加 / 替换以下配置：

```vim
" 基础兼容 & 编码
set nocompatible
set encoding=utf-8
set clipboard=unnamedplus     " 打通系统剪贴板
set timeoutlen=300 ttimeoutlen=10  " 优化Esc延迟

" 界面设置
set number relativenumber     " 行号+相对行号（快速跨行移动）
set cursorline colorcolumn=80
set showmode showcmd laststatus=2

" 缩进规范
set expandtab tabstop=4 softtabstop=4 shiftwidth=4
set autoindent smartindent

" 搜索优化
set ignorecase smartcase hlsearch incsearch

" 编辑体验
set scrolloff=5 linebreak
autocmd BufWritePre * %s/\s\+$//e  " 保存自动删除行尾空格

" 代码折叠
set foldenable foldmethod=indent foldlevel=99

" 自定义快捷键
nnoremap <F2> :set number!<CR>
nnoremap <F3> :set cursorline!<CR>
nnoremap <leader>q :wq<CR>
nnoremap <leader>s :source ~/.vimrc<CR>
nnoremap <C-p> :CtrlP<CR>

" 文件类型支持
filetype plugin indent on
```

保存重载：`:source ~/.vimrc`

## 2. 主流插件安装（统一目录：`~/.vim/pack/vendor/start`）

进入插件目录：`cd ~/.vim/pack/vendor/start`

| 插件名称        | 安装命令                                                     | 核心功能        | 基础用法                                                 |
| --------------- | ------------------------------------------------------------ | --------------- | -------------------------------------------------------- |
| NERDTree 文件树 | `git clone https://github.com/preservim/nerdtree.git`        | 左侧文件目录树  | 配置 `nnoremap <C-n> :NERDTreeToggle<CR>`，`Ctrl+N` 开关 |
| vim-surround    | `git clone https://github.com/tpope/vim-surround.git`        | 操作括号 / 引号 | `ds)`删括号、`cs"'`替换引号                              |
| vim-commentary  | `git clone https://github.com/tpope/vim-commentary.git`      | 一键注释        | `gcc`注释单行、可视模式`gc`批量注释                      |
| vim-airline     | `git clone https://github.com/vim-airline/vim-airline.git`  `git clone https://github.com/vim-airline/vim-airline-themes.git` | 美化状态栏      | 配置 `let g:airline_theme='molokai'`                     |
| vim-polyglot    | `git clone https://github.com/sheerun/vim-polyglot.git`      | 全语言语法高亮  | 安装即用                                                 |
| molokai 配色    | `git clone https://github.com/tomasr/molokai.git`            | 深色主题        | 追加 `colorscheme molokai`                               |

## 3. 插件管理

- 更新插件：进入插件目录执行 `git pull`
- 卸载插件：`rm -rf 插件文件夹名`

# 八、任务 7：高阶实战 — Vim 宏实现 XML → JSON（重点作业）

## 任务目标

使用 **Vim 宏（录制 / 回放）** 批量将 XML 结构体转为标准 JSON 数组，全程自动化编辑。

## 测试用原始 XML 文件

```xml
<people>
  <person>
    <name>Alice</name>
    <email>alice@test.com</email>
  </person>
  <person>
    <name>Bob</name>
    <email>bob@test.com</email>
  </person>
</people>
```

## 前置规则

1. 全程**普通模式**操作；录制宏禁止误按空格 / 方向键。
2. 光标定位优先使用 `^`（跳至行首第一个非空格字符，适配缩进）。
3. 单宏测试通过后，再执行批量操作。
4. 清空寄存器：`qeq`(清空 e)、`qpq`(清空 p)、`qaq`(清空 a)。

### 步骤 1：文件预处理（删除外层根标签）

```vim
ggdd    " 删除第一行 <people>
Gdd     " 删除最后一行 </people>
```

处理后仅保留 `<person>` 块。

### 步骤 2：录制宏 `e`（最小单元：单行标签转换）

**功能**：`<key>value</key>` → `"key": "value"`

1. 光标定位到 `<name>` 行，按 `^` 定位。
2. 录制 & 逐键操作：
   1. `qe          " 开始录制到寄存器e r"          " < 替换为 " f>          " 跳转到 > s":         " 删除>，插入 ": Esc         " 回普通模式 f<          " 跳转到 </key> 的 < C           " 删除光标到行尾，进入插入模式 "           " 输入闭合双引号 Esc         " 回普通模式 q           " 结束录制`
3. **测试**：光标移到 `<email>` 行，执行 `@e`，格式无误再继续。

### 步骤 3：录制宏 `p`（单 `<person>` 块转换）

**功能**：整段 XML 块转为 JSON 对象 `{ ... },`

1. 按 `u` 撤销，还原 XML；光标定位到 `<person>` 行，`^` 定位。
2. 录制 & 逐键操作：
   1. `qp          " 开始录制到寄存器p c{          " 替换整行为 { Esc j           " 下移到name行 @e          " 执行宏e A,          " 行尾加逗号 Esc j           " 下移到email行 @e          " 执行宏e A,          " 行尾加逗号 Esc j           " 下移到</person>行 c},         " 替换整行为 }, Esc q           " 结束录制`
3. **测试**：执行 `@p`，整块转换正常再继续。

### 步骤 4：录制宏 `a`（全局遍历批量处理）

**功能**：处理当前块 + 自动搜索跳转下一个块（核心：不用 `j` 跳转）

1. 多次 `u` 还原全文件；光标定位到第一个 `<person>` 行。
2. 录制 & 逐键操作：
   1. `qa                  " 开始录制到寄存器a @p                  " 执行单块转换宏 /  <person>         " 搜索下一个带2空格缩进的<person> Enter               " 确认搜索，光标跳转 q                   " 结束录制`
3. **测试**：执行 `@a`，处理当前块并跳转到下一块。

### 步骤 5：全局批量转换

```vim
999@a   " 重复执行999次，自动处理所有块
```

### 步骤 6：收尾（修正为标准 JSON）

1. 删除最后一行多余逗号：
   1. `G$x`
2. 外层包裹数组 `[]`：
   1. `ggI[Esc    " 文件开头插入 [ GA]Esc     " 文件结尾插入 ]`
3. 保存文件：`:w`

### 最终标准结果

```json
[
  {
    "name": "Alice",
    "email": "alice@test.com"
  },
  {
    "name": "Bob",
    "email": "bob@test.com"
  }
]
```

## 宏作业高频易错点（复习重点）

1. 宏 `e` 漏输闭合双引号，导致格式错误。
2. 未使用 `^` 定位光标，缩进错乱，宏失效。
3. 遍历宏使用 `j` 逐行跳转，行数变化后光标错位（必须用搜索 `/`）。
4. 批量执行前未做单宏测试，错误扩散。
5. JSON 最后一行存在多余逗号，语法不合法。

## 急救命令（卡壳专用）

- `u`：撤销上一步操作
- `:reg`：查看所有寄存器 / 宏内容
- `qeq/qpq/qaq`：清空对应宏寄存器

# 九、全流程总复习思维导图

1. 入门：基础命令 → `vimtutor` 练习
2. 配置：基础 `vimrc` 部署 → 深度定制配置
3. 插件：原生 `pack` 安装 `ctrlp.vim` → 扩展常用插件
4. 适配：全工具开启 Vim 键位，统一操作习惯
5. 实操：代码编辑练习（FizzBuzz）
6. 高阶：Vim 宏录制 → XML 转 JSON 自动化实战



