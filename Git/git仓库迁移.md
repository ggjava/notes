
最近迁移`git`项目到新的仓库，想保留原有分支和提交，网上搜索了好多步骤都很繁琐。最后发现了最优的解决方案。

### 先克隆老项目的镜像

```bash
git clone --mirror old.git  // old.git 为老项目的git地址
```

### 进入老项目的目录

```bash
cd old.git
```

### 移除老项目的地址替换成新项目

```bash
git remote set-url --push origin  new.git  // new.git 为新项目的git地址
```

### 将镜像推到远程

```bash
git push --mirror  
```

> 这一步需要输入新的git的账号和密码

四步就搞定了
