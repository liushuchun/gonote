添加vscode终端的快捷别名

如下是在官网抄下来的：

> You can also run VS Code from the terminal by simply typing`code`.
>
> To set it up, launch VS Code. Then open the**Command Palette**\(⇧⌘P\) and type`shell command`to find the**Shell Command: Install 'code' command in PATH**command.
>
> Finally, restart the terminal for the new`$PATH`value to take effect. You'll be able to simply type`code .`in any folder to start editing files in that folder.

![](https://code.visualstudio.com/images/mac_shell-command.png)

大致是进入VS Code,执行⇧⌘P,输入shell command,自动执行安装快捷alias.并且重启terminal\(zsh无需重启\)。

**一些快捷键**

快捷键: shift + alt + 鼠标左键

vscode 版本 &gt; 1.2.0

Ctrl+P 模式: \(Mac 是 CMD+P\)

直接输入文件名，快速打开文件

: 跳转到行数，也可以Ctrl+G直接进入\(Mac 是 CMD+G\)

@ 跳转到symbol（搜索变量或者函数），也可以Ctrl+Shift+O直接进入

@:根据分类跳转symbol，查找属性或函数，也可以Ctrl+Shift+O后输入:进入

\# 根据名字查找symbol，也可以Ctrl+T

**编辑:**

上下移动一行： Alt+Up 或 Alt+Down

向上向下复制一行： Shift+Alt+Up 或 Shift+Alt+Down

代码格式化：Shift+Alt+F，或 Ctrl+Shift+P 后输入 format code

更改代码文件语言模式: 显示–&gt;状态栏显示.

**代码重构:**

跳转到定义处：F12

列出所有的引用：Shift+F12

重命名：比如要修改一个方法名，可以选中后按F2，输入新的名字，回车，会发现所有的文件都修改过了。

**显示相关:**

侧边栏显/隐：Ctrl+B

预览markdown: Ctrl+Shift+V

双栏对比: Ctrl+\

**皮肤预览:**

F1后输入 theme 回车，然后上下键即可预览。

**项目成本合算表**

| 单元 | 第一个月 | 第二个月 | 第三个月 | 计算 | 统计 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **硬件** |  |  |  |  | **1,550** |
| 云服务器1500 |  |  |  |  | 1,500 |
| 域名 50 |  |  |  |  | 50 |
| **分析设计阶段** |  |  |  |  | **20,000** |
| 项目分析设计 | 1人\*0.4月 |  |  | 0.4\*20000 | 8000 |
| UI设计绘制 | 1人\*0.8月 |  |  | 0.8\*15000 | 12000 |
| **开发** |  |  |  |  | **165,000** |
| Android系统定制 | 1人\(开发\) | 1人\(测试开发\) |  | 2\*15000 | 30,000 |
| Android驱动编写 |  | 1人\(开发\) | 1人\(测试联调\) | 2\*15000 | 30,000 |
| Android APP开发 | 2人\(开发\) | 2人（开发） | 2人（测试联调） | 2\*15000\*2 | 60,000 |
| 云端后台开发 | 1人\(开发\) | 1人\(开发\) | 1人\(测试联调\) | 1\*15000\*3 | 45,000 |
| **后期实施** |  |  |  |  |  |
| 运营上线 |  |  | 1人\*0.2\(上线\) | 0.2\*15000 | **3,000** |
|  |  |  |  |  | 合计：**189,550** |



