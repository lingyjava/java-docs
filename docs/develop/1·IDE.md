# IDE

- [IDE](#ide)
  - [IDE是什么](#ide是什么)
  - [IDEA](#idea)
    - [安装 \& 破解](#安装--破解)
    - [配置](#配置)
      - [Maven设置](#maven设置)
    - [插件](#插件)
    - [快捷键](#快捷键)
    - [快速输入](#快速输入)
    - [视图设置](#视图设置)

## IDE是什么
IDE是软件开发工具，集成编译环境。
主流Java IDE有：IDEA、EClipse.

## IDEA

### 安装 & 破解
- [IntelliJ IDEA 2022.3.3最新激活破解教程](https://www.exception.site/essay/how-to-free-use-intellij-idea-2019-3)

### 配置
- [IDEA这样配置，好用到爆炸](https://juejin.cn/post/7021148767063638052)

#### Maven设置
IDEA对maven进行了很好的支持，设置好Maven能够提升工作效率，以下是IDEA中Maven的配置方法
TIPS以下的步骤是基于你自己在电脑中安装了Maven的环境下进行说明，如果你没自己安装，那默认使用的是IDEA自带的Maven，Bundled Maven3，就不需要进行以下设置。
1、打开设置（File → Settings），或者使用快捷键Ctrl + Alt + S
2、选择Build,Execution,Deployment →Build Tools → Maven
或者在搜索框中搜索maven
3、设置Maven home directory为你自己安装的Maven目录
4、设置User settings file为你自己安装的Maven目录下conf文件夹下的settings.xml（需要勾选右边的Override）
5、设置local repository为你自己定义的本地Maven仓库目录（需要勾选右边的Override）

### 插件
[IDEA插件市场](https://plugins.jetbrains.com/)

推荐插件：
1. Alibaba Java Coding Guidelines plugin：阿里巴巴开发规约检查，检测代码中不符合规范的地方，并予以提示以及指导如何修改。
2. Chinese ( Simplified)Language Pack/中文语言包
3. GitTooBox：显示git提交记录
4. Key Promoter X：进行操作时提示可以使用的快捷键
5. Maven Helper：检查Maven依赖的版本、结构和冲突
6. Nyan Progress Bar：彩虹兔进度条
7. Rainbow Brackets：彩虹括号，每个相邻的括号颜色不同
8. Randomness：随机一个变量值
9. RestfulTool：通过url跳转到方法位置
10. Translation：翻译插件
11. Free My Batis plugin：mapper层接口和xml文件互相跳转
12. Mybatis Plugin：查看Mybatis执行的SQL语句，收费需破解。
13. Lombok：使用lombok包必备插件。
14. FindBugs-IDEA：帮助你检测你的代码中可能存在的潜在bug并显式地提醒你代码的漏洞在何处。
15. Eclipse-Code-Formatter：格式化代码的插件。

### 快捷键
- [Intellij idea用快捷键自动生成序列化id](https://blog.csdn.net/u013806366/article/details/51911529)

| 快捷键                                       | 作用             |
| -------------------------------------------- | :--------------- |
| 快速打开Action                               | Ctrl + Shift + A |
| 快速查找文件                                 | 双击Shift        |
| 全局搜索                                     | Ctrl + H         |
| 快速生成构造方法、toString()、Getter、Setter | Alt + Insert     |

切换快捷键风格方式：
1、打开设置（File → Settings），或者使用快捷键Ctrl + Alt + S
2、选择Keymap，设置快捷键风格，如Eclipse.

### 快速输入
| 指令       | 作用                         |
| ---------- | :--------------------------- |
| psvm       | 快速输入main方法             |
| sout       | 快速输入Systen.out.println() |
| 变量名.for | 快速创建循环语句             |


### 视图设置
在视图设置中选择开启某些视图可以提升开发效率。
IDEA窗口右侧会显示DB、Maven等常用窗口，可以执行maven的生命周期命令。
