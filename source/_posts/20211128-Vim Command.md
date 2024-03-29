---
title: Vim Command
---
# Move

| 命令                     | 功能                                         | 示例 | 说明                                                         |
| ------------------------ | -------------------------------------------- | ---- | ------------------------------------------------------------ |
| w/W                      | 跳到下一个单词的开头                         |      | w是传统意义上的单词，而W则将一个非空字符串序列视为一个单词。 |
| b/B                      | 跳到上一个单词的开头                         |      | b和w类似，B和W类似。                                         |
| e/E                      | 跳到下一个单词的结尾                         |      | e和w类似，E和W类似。                                         |
| ge/gE                    | 跳到上一个单词的结尾                         |      | ge和w类似，gE和W类似                                         |
| f/F{character}           | 跳到下一个/上一个character处                 |      |                                                              |
| t/T{character}           | 跳到下一个/上一个character的前一个字符出     |      | 这里的前一个字符，是指搜索方向上的前一个，即当T的时候是指字符后面的一个字符。 |
| 0                        | 移动到当前行的第一个字符                     |      |                                                              |
| ^                        | 移动到当前行的第一个非空字符                 |      |                                                              |
| $                        | 移动到当前行的最后一个字符                   |      |                                                              |
| g_                       | 移动到当前行的最后一个非空字符               |      |                                                              |
| }/{                      | 向下/向上跳过一个段落                        |      | 这里段落的定义应该是以空行隔开的行为一段。                   |
| ctrl+d                   | 向下滚动半页                                 |      |                                                              |
| ctrl+u                   | 向上滚动半页                                 |      |                                                              |
| /{pattern} or ?{pattern} | 搜索/是向下搜索，?是向上搜索                 |      | 搜索后，使用n可以看跳到下一个，N跳到上一个，这里上一个和下一个是对应于搜索方向的。gn表示跳转到下一个并选中，gN类似。 |
| gd                       | 跳到定义处                                   |      | 是VScode Vim定义的，不是原生的命令                           |
| gf                       | 跳到导入的文件处                             |      | 是VScode Vim定义的，不是原生的命令                           |
| gg                       | 跳到文件的第一个行                           |      | 前面若加上行号则是跳转到指定行，即23gg跳转到第23行，gg=1gg   |
| G                        | 跳转到文件最后一行                           |      |                                                              |
| %                        | 光标在({[]})中或上时，会跳转到匹配的符号上。 |      |                                                              |

# Edit



| 命令   | 功能                             | 示例 | 说明                           |
| ------ | -------------------------------- | ---- | ------------------------------ |
| d      | 删除                             |      | 支持motion                     |
| u      | 撤销                             |      |                                |
| ctrl+r | 撤销撤销                         |      |                                |
| ctrl+o | 回到上次编辑的地方               |      |                                |
| c      | 改变，删除并插入                 |      | 支持motion                     |
| dd     | 删除整行                         |      |                                |
| D      | 从光标删除的行尾                 |      |                                |
| .      | 重复上一次的操作                 |      | 前面也可以加数字表示重复次数   |
| cc     | 类似dd                           |      |                                |
| C      | 类似D                            |      |                                |
| y      | 复制                             |      | 支持motion                     |
| p      | 粘贴                             |      | d、c、y、x等操作都会改变粘贴板 |
| g~     | 转换大小写                       |      |                                |
| gU/u   | 变为大写/小写                    |      |                                |
| >      | 向右缩进                         |      | 支持motion                     |
| <      | 移除缩进                         |      | 支持motion                     |
| =      | 格式化代码                       |      |                                |
| x      | 删除光标所在字符                 |      | 等价于dl                       |
| X      | 删除光标前的字符                 |      | 等价于dh                       |
| s      | 删除光标前的字符，并进入插入模式 |      | 等价于ch                       |
| r      | 替换单个字符                     |      |                                |
| ~      | 切换单个字符的大小写             |      |                                |
|        |                                  |      |                                |

一些编辑操作可以与移动操作结合，具体格式为{operator}{count}{motion}，定义了操作作用的范围。如d2w表明删除后面两个单词，能够与移动结合的编辑操作在说明中说明。

text object描述文档中各个部分的结构化文本片段：单词、句子、引用文本、段落、块、(HTML) 标签等。您可以将它们与运算符结合使用来更改单词、句子、段落、 等等，具体格式为{operator}{a|i}{text-object}

其中，text-object有以下几个：w（单词），s（句子），p（段落），"（引号）等，还有各种括号也是。a表示环绕，i表示内部，text-object为w、s、p是二者无区别，但若是引号和各种括号时，则a会连引号和括号一起修改，而i则保留引号和括号。

# insert

| 命令     | 功能                                 | 示例 | 说明 |
| -------- | ------------------------------------ | ---- | ---- |
| i        | 在当前光标处插入，即在选中字符前插入 |      |      |
| I        | 在当前行的开头插入                   |      |      |
| a        | 在当前光标后插入，即在选中字符后插入 |      |      |
| A        | 在当前行后面插入                     |      |      |
| o        | 在下面插入一行                       |      |      |
| O        | 在上面插入一行                       |      |      |
| ctrl + o | 跳到上一次编辑的位置                 |      |      |
| gi       | 跳到最近一次编辑的地方               |      |      |
| ctrl+h   | 移除光标前一个字符                   |      |      |
| ctrl+w   | 移除光标前一个单词                   |      |      |
| ctrl+u   | 移除光标前一行                       |      |      |

# Select

| 命令   | 功能                               | 示例 | 说明                                |
| ------ | ---------------------------------- | ---- | ----------------------------------- |
| v      | 进入选择模式，以字符为单位选择。   |      |                                     |
| V      | 进入选择模式，以行为单位进行选择。 |      |                                     |
| ctrl+v | 进入选择模式，以块为单位进行选择。 |      |                                     |
| I      | 块选择模式下，在块前进入插入模式。 |      | 在VScode Vim中，v/V模式下也可以使用 |
| A      | 块选择模式下，在块后进入插入模式。 |      | 在VScode Vim中，v/V模式下也可以使用 |

选择模式也可以配合移动和编辑操作，具体的格式为{v|V|ctrl+v}{count}{motion}{operator}，含义是，进入选择模式，选择哪些字符/行/块，并对选择的块进行操作。



# 复制粘贴

| 命令              | 功能 | 说明                                                     |
| ----------------- | ---- | -------------------------------------------------------- |
| y                 | 复制 |                                                          |
| p                 | 粘贴 |                                                          |
| ctrl+r {register} | 粘贴 | 在插入模式下粘贴，若粘贴的内容是一行时，粘贴时不会换行。 |

## Register

register类似剪贴板，可以存储一些东西

默认的register名是"，命名的register是a-z，这两种存储器存储的是复制和粘贴的内容。

register 0存储最后复制的内容，删除和剪切不会影响。

resigert 1-9存储最后9个剪切或删除的内容。

使用指定register为{register name}c/d/y{motion}

使用:reg {register}可以查看register的内容。

# 命令行模式

命令行模式下可以执行Ex命令（以:开头）和搜索模式（以/和?开头）

Ex命令可以配置Vim、执行系统范围的操作、访问外部的shell命令等。VSCodeVim只支持有限的Ex命令。

| 命令                                     | 功能               | 说明                                                         |
| ---------------------------------------- | ------------------ | ------------------------------------------------------------ |
| :edit {path}/:e                          | 打开或创建文件     | 是相对于当前打开文件的路径，且不支持TAB补全。                |
| :write/:w                                | 保存文件           |                                                              |
| :quit/:w                                 | 关闭文件           |                                                              |
| !                                        | 强制执行命令       | 和其他命令配合使用                                           |
| all/a                                    | 对所有文件执行命令 | 和其他命令配合使用                                           |
| @:                                       | 重复前一个命令     | 之后使用@@命令来重复                                         |
| :[range]s/{pattern}/{substitute}/{flags} | 替换文本           | flag：g表示当前行，%表示整个文件，i表示大小写敏感，c表示需要确认每个替换 |

还可以再命令行模式下使用一些normal模式下的命令，区别是，命令行模式下的命令会一次操作多行，格式为[range] command [options]

例如，10,12d a的意思是删除10到12行的内容，放到寄存器a中。

行数的表达有以下几种方式

1. 10,12：指定开始和结束行数
2. 10,+2：指定开始行和便宜数
3. .,+2：使用 . 来表示当前行
4. %：表示整个文件
5. 0,\$：0表示文件的开头，\$表示文件的结尾。

# 分割

| 命令              | 功能                                    | 说明 |
| ----------------- | --------------------------------------- | ---- |
| :sp/:vsp {path}   | 水平/竖直分割窗口，在新窗口打开指定文件 |      |
| ctrl+w s/ctrl+w v | 水平/竖直分割窗口                       |      |
| ctrl+w hjkl       | 在窗口间移动                            |      |

# 标签

| 命令           | 功能              | 说明 |
| -------------- | ----------------- | ---- |
| :tabnew {file} | 在新tab中打开文件 |      |
| :tabn          | 跳转到下一个tab   |      |
| :tabp          | 跳转到前一个tab   |      |
| :tabo          | 关闭其他tab       |      |
| :tabfir        |                   |      |
| :tabl          |                   |      |
| :tabe {file}   |                   |      |
| :tabc          |                   |      |



# Surrounding

|      |                 |      |
| ---- | --------------- | ---- |
| ds   | 删除surrounding |      |
| ys   | 增加surrounding |      |
| cs   | 替换surrounding |      |

ds似乎只能删除同一行的surrounding，也不知道如何配合motion，试了没效果。使用dst可以删除标签，t指tag

cs和ds的问题一样，被替换的似乎只能是单个字符，替换的可以是标签。cst替换当前的标签为指定的标签

ys可以配合text-object进行使用。

more example : https://towardsdatascience.com/how-i-learned-to-enjoy-vim-e310e53e8d56
