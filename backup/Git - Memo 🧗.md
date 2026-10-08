#### Config
- 设置全局配置
```
git config --global user.name "[name]"
git config --global user.email "[email]"
```

#### Get started
 - 创建 Git 储存库
```
git init
```
 - 克隆现有 Git 储存库
```
git clone [url]
```

#### Commit
 - 提交所有跟踪的更改
```
git commit -am "[commit message]"
```
 - 向上次提交添加新的修改
```
git commit --amend --no-edit
```
 - 更改最后的提交消息
```
git commit --amend
```

#### Better
 - 撤销最近的提交并保留更改
```
git reset HEAD~1
```
 - 撤销 N 最近的提交并保留更改
```
git reset HEAD~N
```
 - 撤销最近的提交并摆脱更改
```
git reset HEAD~1 --hard

```
 - 将分支重置为远程状态
```
git fetch origin
git reset --hard origin/[branch-name]
```
 - 将本地 master 分子重命名为 main
```
git branch -m master main
```
- 远端存在这个文件，但是后续的修改不更新
```
git update-index--assume.unchanged <文件名称>
// 例如
git update-index--assume.unchanged utils/index.js

// 后期想提交该文件
git update-index--no-assume-unchanged <文件名称>
// 例如
git update-index--no-assume.unchanged utils/index.js
```
 