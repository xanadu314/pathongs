# Vim 使用



---

vim 工具的基本使用

3种模式：

- normal 模式
- insert 模式
- visual 模式


| 按键 | 功能 |
| ---- | ---- |
| i    | 进入insert模式，处于可以编辑状态 |
| ESC  | 进入normal模式 |
| a    | 进入insert模式，但与i不同，是从光标的下一位进行编辑，相当与add |
| v    | 进入visual模式 |
| CTRL+f, CTRL+d | 向下翻页 |
| CTRL+n, CTRL+u | 向上翻页 |





## 修改

最基本的`i`进入编辑模式后可以像常规文本编辑器进行修改。其他常见的修改操作：

- x 删除当前光标下的字符
- X 删除当前光标下前一个字符
- s 删除当前光标下的字符，并且进入insert模式
- r 按下后，再按下下一个字符时会将该字符取代当前光标位置的字符，相当与替换replace
- d 删除命令
- D 删除当前光标下后一个字符直到该行结尾，新的


`y`是复制行为，常见用法是:

- `yy`复制当前行
- `y5y`是从当前光标行开始复制5行，数字是几就复制几行
- `yw`是从当前光标开始复制到该单词结束，如果是中文则复制到空格或标点处
- `yiw`表示复制光标所在的单词，如果是中文则复制到空格或标点处。
- `yt,`

`d`是删除行为，常见的用法是：

- `dd`删除当前行，实际效果与剪切一致，可以用`pp`将删除行粘贴
- `d5d`是从当前光标行开始删除5行，数字是几就删除几行
- `dw`删除
- `daw`删除当前光标所在单词,会一直删除到空格
- `diw`删除当前光标所在单词,不会一直删除到空格
- `dt,`往后删除，一直搜索到逗号停止。可以用其他字母或者符号代替，表示一直删除到搜索到的所在符号前
- `de`删除到该单词结尾，`D`elete to the `E`nd


:g/pattern/d



`:Sex`垂直分屏打开新窗口显示文件
`:Tex`以tab方式打开新窗口显示文件

`gt`向前切换tab
`gT`向后切换tab



~/.vimrc

.vimrc 是 vim 的配置文件。
