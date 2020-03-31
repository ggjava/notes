##  本地修改了一堆文件，没有git add到暂存区

### 单个文件/文件夹

```bash
$ git checkout -- filename
```

### 所有文件/文件夹

```bash
$ git checkout .
```

### 本地新增了一堆文件，没有git add到暂存区

### 单个文件/文件夹

```bash
$ rm filename / rm dir -rf
```

### 所有文件/文件夹

```bash
$ git clean -xdf
```

> 删除新增的文件，如果文件已经已经git add到暂存区，并不会删除！

## 本地修改/新增了一堆文件，已经git add到暂存区

### 单个文件/文件夹

```bash
$ git reset HEAD filename
```

### 所有文件/文件夹

```bash
$ git reset HEAD .
```

## 撤销commit

```bash
$ git reset commit_id
```

> 撤销之后，你所做的已经commit的修改还在工作区！

```bash
$ git reset --hard commit_id
```

commit_id是你想要回到的那个节点，可以通过git log查看，可以只选前6位

> 撤销之后，你所做的已经commit的修改将会清除，仍在工作区/暂存区的代码也将会清除！
