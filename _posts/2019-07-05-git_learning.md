---
layout: post
title:  "Git学习之路"
date:   2019-07-05 15:20:00 +0000
categories: jekyll update
---
<font size="5" color="#00dddd"> __git学习笔记__ :</font><br />

<font size=3 color=green>__Tips__</font> : 
> 1. git支持多种协议，默认点git://使用ssh，也可以使用https等其他协议。不过https速度慢，且每次推送必须输入口令。  
> 2. [.gitignore](https://github.com/github/gitignore)文件用于填写忽略文件，git将会自动无视这些文件。.gitignore文件本身要提交版本库里(add+commit)，并且可以对.gitignore做版本管理！

<font size="3" color=red>__.git__</font> : 
> 表示repository(版本库)，其中有stage，branch等等。  

<font size=3 color=red>__HEAD__</font> : 
> 指向分支的指针，其所指位置表示当前所在的版本。 

<font size=3 color=red>__分支策略__</font> : 
> master分支保持稳定，用于发布新版本，不可用于工作。多人合作时应当在dev分支上创建自己的分支进行工作，并时不时地往dev分支上进行合并。在发布版本时，再将dev分支合并到master分支上。

<font size=3 color=pink>__注意__</font> : 
> 1. git追踪记录的是文件的修改内容而非文件。因此，当一次修改add至stage后又进行了修改，需要再次进行add，最后commit。  
> 2. 在进行分支合并时，一旦出现冲突，应当先将冲突的文件进行手动编辑进行解决。冲突解决后，再提交，合并完成。(Git用<<<<<<<，=======，>>>>>>>标记出不同分支的内容。)  
> 3. 使用git merge 进行Fast-forward模式进行合并时，若文件内容产生冲突，当前分支版本状态不会改变，工作区中的文件内容被修改为冲突的情况。需要进行修改后并git add，git commit进行提交后完成合并。若两个分支存在不同的文件，则会提示这种合并的情况，然后会生成新的提交点，该版本包含这两个文件，完成合并。  
> 4. 添加至各分支的不同修改内容是彼此不可见的，包括新建的文件。  
> 5. 当工作区新产生的修改时，包括文件内容和文件删增，若未提交至分支中(即使修改情况被添加至stage)，则在所有分支中都可见。(这种可见在切换分支后会覆盖相同文件的内容，即产生修改)


`git add` : 将工作区中修改的文件添加至stage(暂存区)。  
`git commit (-m)` : 将stage中的修改提交至branch(分支)，-m用于注释。  
`git status` : 显示目前的状态，包括工作区，stage和branch等。
`git diff` : 展示工作区中文件内容修改的现状(与上一次add/commit相比)。  
`git diff HEAD -- file` : 查看当前版本与工作区中文件内容的不同状况。  
`git log (--pretty=oneline)` : 日志，可按一行展示，最前面的id为commit(快照)id，即版本号。  
`git reset --hard id` : 将版本切换至id版本，id可以为版本号，HEAD，或者HEAD^/～num(^表示HEAD上一个，～num表示HEAD上num个)。  
`git relog` : 显示所有的操作记录。  
`git checkout -- file` : 将工作区中修改的文件回退至上一次git add / git commit时的内容。  
`git reset HEAD file` : 将stage中的文件移除。  
`git rm file` : 删除文件，并从版本库中移除，之后需要commit。当使用rm删除文件时也需要此命令来从版本库中移除该文件，或git checkout --来恢复该文件。(reset来切换版本号也可以恢复删除的文件)  
`git remote add origin git@server-name:path/repo-name.git` : 关联远程仓库。  
`git push -u origin master` : 第一次推送master分支内容，此后推送无需-u参数。  
`git clone git@server-name:path/repo-name.git` : 可以从远程仓库克隆至本地仓库。(同时会于远程仓库进行关联，并默认将master分支进行链接)  
`git branch` : 查看所有分支.  
`git branch name` : 创建分支。  
`git branch -d name` : 删除分支。  
`git checkout name` : 切换分支。  
`git checkout -b name` : 创建并切换分支。  
`git merge name` : 合并分支。(默认尝试Fast-forward模式合并，合并完成后便可删除分支)  
`git log --graph --pretty=oneline --abbrev-commit` : 查看分支合并情况。  
`git merge --no-ff -m "comment" name` : 不使用Fast-forword模式合并，在合并完成后生成一个commit。(这种方式，在删除分支后，不会丢弃分支信息，便于从合并后的历史分支中观察出曾经的合并)  
`git stash` : 将现场的工作“储藏”起来，包括stage中的内容以及其对应的文件。(常用于修改bug时将现有的工作现场进行保存，以避免提交和修改未完成的文件内容，待完成后继续；workdirectory的修改情况和stage的内容是所有分支共享的)  
`git stash list` : 显示stash中的所有暂存的工作现场。 
`git stash apply stash@{i}` : 将stash@{i}进行恢复。(恢复后不会删除stash中的内容，需要额外命令进行删除)  
`git stash drop stash@{i}` : 将stash@{i}从stash中删除。  
`git stash pop stash@{i}` : 将stash@{i}进行恢复并从stash中删除。(stash类似栈，0表示栈顶；以上若不使用stash@{i}参数则表示将栈顶进行操作)  
`git branch -D name` : 强制删除未合并的分支。  
`git remote` : 查看远程仓库名称。  
`git remote -v` : 查看远程库的信息，包括抓取和推送的地址。   
`git checkout -b branch-nanme origin/branch-name` : 创建并切换分支，同时将该分支远程库的分支建立链。  
`git push origin branch-name` : 向远程分支(若不提供后面两个参数，则将与本地当前分支相链接的远程分支进行操作)进行推送。  
`git pull origin branch-name` : 将远程分支(不提供后两个参数同上一条)抓取下来，并与本地分支进行合并。(需要在本地解决冲突后再向远程进行推送；若远程提前于本地，则会将本地更新至远程的状态)  
`git fetch origin branch-name` : 将远程库的分支抓取下来，但是不与本地分支进行merge。(<font color=yellow>fetch+merge=pull</font>)  
`git branch --set-upstream-to=origin/branch-name branch-name` : 将远程分支与本地分支进行链接。(在抓取远程分支内容提示未建立链接时使用，然后再git pull抓取；或者不建立链接，通过 git pull remote-name branch-name进行抓取)  
`git rebase` : 将本地的未push的分叉整理成直线。  
`git tag tag-name` : 在当前分支的最新提交点上建立一个标签。(即创建一个指向该提交点的指针)  
`git tag tag-name commit-id` : 对id提交点建立标签。  
`git tag` : 查看所有标签。  
`git show tag-name` : 显示标签的具体信息。  
`git tag -a tag-name -m "..." (commit-id)` : 建立标签并注释。  
`git push origin tag-name` : 将标签推送上去。  
`git push origin --tags` : 推送本地全部标签。  
`git tag -d tag-name` : 删除本地标签。  
`git push origin :refs/tags/tag-name` : 删除远程的一个标签。  

