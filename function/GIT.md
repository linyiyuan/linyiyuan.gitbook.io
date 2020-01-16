1. 配置昵称:
    > git config --global user.name "你的昵称"  #
    
2. 配置邮箱:
    > git config --global user.email "你的"
    
3. 生成SSH密钥:
    > ssh-keygen -t rsa -C "全局配置的邮箱"

4. 克隆项目:
    > git clone 项目地址
    
5. 克隆下来指定的单一分支:
     > git clone -b <分支名称> --single-branch 项目地址 （clone下来指定的单一分支）

6. 把工作时的所有变化提交到暂存区:
      > git add .

7. 更新已经提交到暂存区的文件（git add --update的缩写）:
    > git add -u

8. 是上面两个功能的合集（git add --all的缩写）:
    > git add -A

9. 提交本地仓库:
    > git commit -m ‘描述’

10. 提交远程指定仓库:
    > git push origin master

11. 从远程指定仓库拉取代码:
    > git pull origin master 

12. 显示改动日志:
    > git log

13. 重设第一个commit (也就是把所有的改动都重新放回工作区，并清空所有的commit，这样就可以重新提交第一个commit了):
    > git update-ref -d HEAD

14. 回到某一个commit的状态，并重新增加一个commit:
    > git revert <commit-id>

15. 回到某个commit的状态，并删除后面的commit 和revert的区别是：reset命令会抹去某个commit id之后的所有commit:
    > $ git reset <commit-id>  #默认就是-mixed参数。

16. 回退至上个版本，它将重置HEAD到另外一个commit,并且重置暂存区以便和HEAD相匹配，但是也到此为止。工作区不会被更改:
    > git reset –mixed HEAD^

17. 回退至三个版本之前，只回退了commit的信息，暂存区和工作区与回退之前保持一致。如果还要提交，直接commit即可:
    > git reset –soft HEAD~3
    
18. 彻底回退到指定commit-id的状态，暂存区和工作区也会变为指定commit-id版本的内容:
    > git reset –hard <commit-id>

19. 修改上一个commit的描述:
    > git commit --amend

20. commit历史中显示Branch1有的，但是Branch2没有commit:
    > git log Branch1 ^Branch2

21. 展示简化的commit历史:
    > git log --pretty=oneline --graph --decorate --all