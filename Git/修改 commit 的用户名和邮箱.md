在 Git 上工作的时候，你也许会由于某种原因想要修订你的提交历史。Git 的一个卓越之处就是它允许你在最后可能的时刻再作决定。你可以在你即将提交暂存区时决定什么文件归入哪一次提交，你可以使用 stash 命令来决定你暂时搁置的工作，你可以重写已经发生的提交以使它们看起来是另外一种样子。这个包括改变提交的次序、改变说明或者修改提交中包含的文件，将提交归并、拆分或者完全删除——这一切在你尚未开始将你的工作和别人共享前都是可以的。

## filter-branch
如果你想用脚本的方式修改大量的提交，还有一个重写历史的选项可以用——例如，全局性地修改电子邮件地址或者将一个文件从所有提交中删除。这个命令是filter-branch，这个会大面积地修改你的历史，所以你很有可能不该去用它，除非你的项目尚未公开，没有其他人在你准备修改的提交的基础上工作。尽管如此，这个可以非常有用。

我们创建了一个脚本，它将更改以前在author或committer字段中拥有旧电子邮件地址的所有提交，以使用正确的名称和电子邮件地址。
注意:运行此脚本将重写所有Git库协作者的历史记录。完成这些步骤后，任何使用fork或clone的人都必须获取重写的历史记录，并将任何本地更改重新建立到重写的历史记录中。

1.打开终端

2.创建一个你的repo的全新裸clone（repo.git替换为你的项目）

```bash
git clone --bare https://github.com/user/repo.git
cd repo.git
```

3.复制粘贴脚本，并根据你的信息修改以下变量：
* OLD_EMAIL
* CORRECT_NAME
* CORRECT_EMAIL

```bash
#!/bin/sh

git filter-branch --env-filter '

OLD_EMAIL="your-old-email@example.com"
CORRECT_NAME="Your Correct Name"
CORRECT_EMAIL="your-correct-email@example.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

4.执行脚本

5.查看新 Git 历史有没有错误

6.把正确历史 push 到 Github

```bash
git push --force --tags origin 'refs/heads/*'
```

7.清楚临时clone

```bash
cd ..
rm -rf repo.git
```

参考自[官方教程](https://help.github.com/en/articles/changing-author-info)

## 修改git的配置文件

### 全局修改

对所有的 git 项目配置

```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

或者直接在全局配置文件中修改

```bash
目录: ~/.gitconfig
[user]
    name = John Doe
    email = johndoe@example.com
```

### 只对当前项目修改

```bash
git config user.name "John Doe"
git config user.email johndoe@example.com
```

### 查看配置信息

要检查已有的配置信息，可以使用 git config --list 命令：

```bash
$ git config --list
user.name=Scott Chacon
user.email=schacon@gmail.com
...
```