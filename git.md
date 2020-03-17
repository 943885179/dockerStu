
git clone [git地址] //下载git项目到本地
git branch //查看本地分支
git branch -a //本地+远程所有分支
git branch [分支名字] //添加分支
git checkout -b xxxxxx 切换到指定分支
git add * 将本地文件添加到本地git
git commit '描述' 提交本地缓存
git push 上传远程
git pull 远程下载
git init 初始化
-- git回滚到特点commit
git reset --hard gitId --本地回滚到特点分支
git push orgin dev --force --强制推送本地到远程orgin dev分支 --force等同于-f
git cherry-pick  在当前分支添加其他分支的固定请求
git log 查看日志
git reflog 查看详细日志
git show gitId --查看某个commit的内容

 git branch -d 分支名称 --删除本地分支
 git branch -D 分支名称 --强制删除本地分支
 git push origin --delete bug_xzx --删除远程的bug_xzx分支
