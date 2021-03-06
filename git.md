# **git**

## **git 基本概念**

### git介绍
git是当前流行的代码版本管理工具，分布式结构，分为本地和远端，其整体结构大概长这样

```
               ┌───────────────────────────┐                       
               │                           │                       
               │    (remote repository)    │                       
               │                           │                       
               └───────────────────────────┘                       
                          ▲    │                                   
                  git push│    │git pull                           
                          │    ▼                                   
               ┌───────────────────────────┐                       
               │                           │──────────│            
      ────────▶│    (local repository)     │          │            
      │        │                           ├────│     │            
      │        └───────────────────────────┘    │     │            
      │                   ▲    │                │     │            
      │         git commit│    │git reset --soft│     │            
      │                   │    ▼                │     │            
      │        ┌───────────────────────────┐    │     │            
git commit -a  │                           │    │     │            
      │        │      (Staging area)       │git reset │            
      │        │                           │    │     │            
      │        └───────────────────────────┘    │     │            
      │                   ▲    │                │ git reset --hard 
      │            git add│    │git reset       │     │            
      │                   │    ▼                │     │            
      │               ┌────────────┐            │     │            
      │───────────────┤   (edit)   │◀────────────     │            
                      └────────────┘                  │            
                          ▲    │                      │            
                     edit │    │git checkout --file   │            
                          │    ▼                      │            
                ┌────────────────────────┐            │            
                │                        │            │            
                │  (working directory)   │◀────────────            
                │                        │                         
                └────────────────────────┘                                                                
```
常见使用master分支模式进行迭代开发,远端master分支受到保护，只有Owner和commiter有权限合入
developer在本地开发后需要提交合并请求

### git 开发常见注意事项：
1.本地仓库只要不推送到远端可以随便搞。

2.开发人数较多或者其他开发者提交频繁时要经常rebase远端的分支，尽量提交合并时发生冲突。（注：为什么要进行rebase,如果不进行rebase的话提交合并后的git graph就会像麻花一样）

3.提交前进行代码风格格式化，和仓库格式保持统一，一般会用clang-format的工具比较方便，关于代码风格统一会再加一篇文章

4.切换不同分支时子包可能会切换，及时更新子包

5.开发时一次commit涉及的文件，修改，内容尽量合适（不要过多），目的有两个，一个是开发时条理清晰，另一个是出现问题后方便用cherry-pick将合适正确的commit提取出来。

## 常用git命令

1.git clone

有https和ssh两种，常用http方便一点

```shell
git clone https://github.com/Oceanflame/Squirtle_work_note.git .
```
其实上面的git clone是两个参数，前面是要clone的远端项目的地址，后面的.其实也是个参数，代表你要clone到本地的路径，.代表将项目clone到当前文件夹（打开命令行所在的文件夹）内。
如果不加后面的参数，它会帮你创建一个文件夹。

一般会包含一些其他的参数，比如常见的 --recursive 用来循环拉取项目内的子包（submodule）。


2.git pull

git pull 会自动拉取该本地分支所关联的远端分支全部更新
注：设置远端关联的上游分支命令为（括号内的不属于命令）
```shell
git branch --set-upstream-to=origin/master(远端) master(本地)
```
git pull会更新本地 git fetch --all 会拉取远端的更新，但是不更新本地分支

3.git branch

与分支有关的操作,列举一些常见的，其余的可以输入 git branch --help 查看
```shell
git branch -a  //查看所有的分支（本地+远端），如果不加 -a 则只显示本地的内容
git branch -D master  //用于删除本地某一个分支，后面要跟分支名
git branch --set-upstream-to=  //用于为本地分支设置上游分支
```

4.git log

查看本地分支commit记录，可以看到每个commit的哈希码
一般与修改commit记录的一些操作配合使用

5.git status

查看分支状态，如果关联了远端的分支那么会对比和远端分支的状态（虽然也是本地记录的远端分支状态）
有不同会提示你的哪些文件进行了修改

有commit记录不同会提示你领先或者落后了多少commit 

6.git checkout 

切换分支，后面接你要切换的分支名
```shell
git checkout master
```
如果加了-b参数，会直接从当前分支迁出一个新分支，如果分支名已存在会创建失败

7.git add

将内容添加到本地的暂存区，在没有git commit 前，修改的内容都会留在暂存区内

```shell
git add ./dev/  //add 后面跟着的是修改的文件路径，如果是文件夹就是文件夹内的全部文件
```

8.git commit 

将本地暂存区内容统一在本地分支上算作一次提交
每一次的commit都有为唯一标识的哈斯值作为记录，用来进行其他处理
```shell
git commit -m "update master"  //-m表示对这次提交的描述，后面的引号为描述内容
```

9.git rebase 

rebase 表示变基操作，用法较多

1>设定两个分支，master和dev，其中dev是由master迁出来的，之后dev又进行了其他的更新，这时候如果master也进行更新,但是dev分支没有记录，就需要用到rebase 来进行变基操作，让dev分支变成以新的master为基础迁出的分支（详细的图解可以通过git rebase --help来查看，看图好理解一点）。其目的是为了防止git graph不要太乱，方便进行版本管理

2>将多个commit 合并为一个commit ,也可以用rebase 命令进行操作，加入-i参数用进行交互式操作，可以重新修改commit 信息
```shell
git rebase -i HEAD~3  //表示合并前3个commit为一个
```
其余参见git rebase --help命令

10.git cherry-pick

用来将某一个到几个commit挑选出来移动到别的分支上，一定程度上可以和rebase完成一样的功能，后面接commit的哈希值。

