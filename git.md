# Git

是分布式的版本管理工具。避免了单点故障。

## 优势

1. 大部分操作在本地完成，不需要联网
2. 完整性保证
3. 尽可能添加数据而不是删除和修改数据
4. 分支操作非常快捷流畅
5. 与Linux命令全面兼容

## 下载与安装

Git-2.20.1-64-bit.exe

安装：一般都是默认就行。

## git的结构

* 工作区：写代码
* 暂存区：临时存储
* 本地库：历史版本

## Git和代码托管中心

代码托管：维护远程库

* 局域网：可以搭建GitLab
* 外网
  * GitHub
  * 码云

## 本地库和远程库

* 团队内部协作
* 跨团队协作

## Git的操作

### 初始化一个本地仓库

git init

![1555215077165](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1555215077165.png)

* 设置签名

  * 形式

    用户名：tom

    Email地址：goodStudy@sina.com

  * 作用

    区分不同开发人员的身份，和登录远程库的账户和密码没有任何关系。

  * 命令：

    * 项目级别/仓库级别：仅在当前本地库范围内有效
      * git config user.name tom
      * git config user.email goodStudy@sina.com
    * 查看：cat .git/config
    * 系统用户级别：登录当前操作系统的用户范围。
      * git config --global user.name tom
      * git config --global user.email goodStudy@sina.com
    * 查看：cat ~/.gitconfig
    * 优先级：就近原则，项目级别优先于系统级别。

### 基本操作

1. 查看状态

   ~~~shell
   git status
   ~~~

2. 添加操作

   ~~~shell
   git add [file name]
   ~~~

3. 提价操作

   ~~~shell
   git commit -m "commit log" [file name]
   ~~~

4. 查看历史记录操作

   ~~~shell
   git log
   
   git log --pretty=online
   
   git log --online
   
   git reflog
   ~~~

5. 版本的前进和后退

   * 本质：HEAD指针的移动

   * 基于索引值操作【推荐】

     ~~~~shell
     git reset --hard [局部索引值]
     #例如
     git reset --hard d4bfd04
     ~~~~

   * 使用^符号，只能往后退。一个^符号代表后退一步，两个后退两个

     ~~~shell
     git reset --hard HEAD^
     ~~~

   * 使用~符号,只能后退

     ~~~shell
     git reset --hard HEAD~n
     # 后退n步
     ~~~

6. reset命令的三个参数对比

   * --soft

     仅仅在本地库移动HEAD指针

   * --mixed

     在本地库移动HEAD指针

     重置暂存区

   * --hard

     在本地库移动HEAD指针

     重置暂存区

     重置工作区

7. 删除文件并找回

   * 前提：删除前，文件存在时的状态提交到了本地库
   * 操作：git reset --hard [指针位置]
     * 删除操作已经提交到了本地库：指针位置指向历史记录
     * 删除操作尚未提交到本地库：指针位置使用HEAD

8. 比较文件差异

   * git diff [文件名]
     * 将工作区中的文件和暂存区中的进行比较

   * git diff [本地库中的历史版本] [文件名]
     * 将工作区中的文件和本地库历史记录比较

   * 不带文件名，比较多个文件

### git的分支管理

* 分支的概念

  在版本控制中，使用多条线同时推进多个任务

* 好处
  * ·同时并行推进多个功能开发，提高开发效率
  * 各个分支在开发过程中，如果某个分支开发失败，不会对其他分支有任何影响。失败的分支删除可重新开始。

* 分支的操作

  * 创建分支

    ~~~shell
    git branch [分支名]
    ~~~

  * 查看分支

    ~~~shell
    git branch -v
    ~~~

  * 切换分支

    ~~~shell
    git checkout [分支名]
    ~~~

  * 合并分支

    * 切换到接收修改的分支

      ~~~shell
      git checkout [被合并分支名]
      ~~~

    * 执行一个merge命令

      ~~~shell
      git merge [有新内容的分支名]
      ~~~

  * 解决冲突

    * 冲突的表现

      ![1555229858106](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1555229858106.png)

    * 解决

      * 编辑文件，删除特殊字符
      * 把文件修改到满意的程度，保存退出
      * git add [文件名]
      * git commit -m "log"
        * 注意：此时commit一定不能带有具体的文件名

## Git的基本原理

### 哈希

哈希是一个系列的加密算法，各个不同的哈希算法虽然强度不同，但是有几个共同特点：

* 不管输入数据的数据量有多大，输入同一个哈希算法，得到的加密结果长度的固定。
* 哈希算法确定，输入数据确定，输出数据就能够保证不变
* 哈希算法确定，输入数据有变化，输出数据一定不一致。
* 哈希算法不可逆

git底层就是采用的SHA-1算法。用来保证数据的完整性。

### git文件管理的机制

1. 每个版本通过快照的形式保存
2. 每个快照之间形成一个父子关系，最终形成一个父子链。

## git与远程库的操作

1. 添加别名

   ~~~shell
   git remote -v   查看当前所有远程库别名
   git remote add [别名][远程地址]
   ~~~

2. 推送

   ~~~shell
   git push [别名][分支名称]
   ~~~

   

3. 克隆

   * 命令

     ~~~shell
     git clone [远程地址]
     ~~~

     

   * 效果
     * 完整的把远程库下载到本地
     * 创建origin远程地址别名
     * 初始化本地库

4. 拉取
   * 命令：pull = fetch + merge
   * git fetch [远程库地址别名] [远程分支名]
   * git merge [远程库地址别名/远程分支名]

5. 冲突问题
   * 如果不是基于Github远程库的最新版所做的修改，不能推送，必须先拉取。
   * 拉取下来后如果进入冲突状态，则按照“分支冲突解决”操作解决即可。