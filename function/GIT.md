1. 配置昵称：
    > git config --global user.name "你的昵称"  #
    
2. 配置邮箱
    git config --global user.email "你的"
    
3. 生成SSH密钥
    ssh-keygen -t rsa -C "全局配置的邮箱"

4. 克隆项目
    git clone 项目地址
    
5. 克隆下来指定的单一分支
     git clone -b <分支名称> --single-branch 项目地址 （clone下来指定的单一分支）

6. 把工作时的所有变化提交到暂存区
      git add .

7. 更新已经提交到暂存区的文件（git add --update的缩写）
    git add -u

8. 是上面两个功能的合集（git add --all的缩写）
    git add -A