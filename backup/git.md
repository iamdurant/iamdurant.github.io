### 概念图
![image](https://github.com/iamdurant/iamdurant.github.io/assets/107034526/54916701-a7d3-4c9a-b465-b69d0be73426)

![image](https://github.com/iamdurant/iamdurant.github.io/assets/107034526/a25ae0ec-eea1-4aa8-b86c-b9bae5f8d9af)

### 基础

#### 设置用户信息
`git config --global user.email "123@bbq.com"`
`git config --global user.name "kevin"`

#### git log
`alias gl='git log --pretty=oneline --all --graph --abbrev-commit'`
`git reflog`

#### 初始化仓库
`git init`

#### staged
`git add <file>...`

#### 查看状态
`git status`

#### 提交
`git commit -m "提交描述信息"`

#### 版本回退
`git reset --hard commitID`

#### .gitignore
```text
.gitignore
*.class
*.iml
*.idea
```

### 分支相关

#### 查看分支
`git branch`

#### 创建分支
`git branch <branch_name>`

#### 删除分支
`git branch -d <branch_name>`
`git branch -D <branch_name>` *强制删除*

#### 切换分支
```text
git checkout <branch_name>
git switch <branch_name>
```

#### 创建并切换分支
`git checkout -b <branch_name>`

#### 合并分支
`git merge <be_merge_branch_name>`

#### 通用分支使用图
![image](https://github.com/iamdurant/iamdurant.github.io/assets/107034526/f0a1c210-5152-4ac9-a989-1e42fa39a0f8)

### 远程仓库相关

#### 关联仓库
`git remote add <name> <git_url>`

#### 关联分支
```text
git branch --set-upstream=<remote_repository_name>/<remote_branch_name> <current_branch_name>
```

#### 查看关联仓库
`git remote`

#### 关联分支
`git push --set-upstream`

#### 推送代码
`git push <remote_repository_name> <local_branch>:<remote_branch>`

#### 拉代码
```text
git fetch <repository_name> <branch_name>     抓取分支，不合并，若不指定branch，则抓取所有branch
git pull <repository_name> <branch_name>       拉取分支，合并，若不指定branch，则拉取所有分支
```
