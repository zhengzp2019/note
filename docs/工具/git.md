# git

```bash
git rebase --onto HEAD~3 HEAD~2 feature_access_dayu
```

<https://chat.qwen.ai/s/634e2064-74b6-48df-b367-ea0deeced80b?fev=0.1.30>

## 从索引数中删除文件

```bash
git rm
```

## 定位每行最后修改人（git blame）

```bash
# 查看文件每一行最后一次是谁改的（含提交、作者、时间）
git blame <file>

# 只看某个行区间
git blame -L 20,80 <file>

# 忽略空白变化（缩进/空格）
git blame -w <file>

# 显示作者邮箱
git blame -e <file>
```

### 常见排查组合

```bash
# 先定位行，再看对应提交详情
git blame -L 120,150 src/main/java/com/example/App.java
git show <commit_id>
```

### 注意

1. `git blame` 显示的是“该行最后一次修改者”，不一定是最初作者。
2. 若经历过大规模格式化，归属可能被刷新，可配合 `-w` 降低干扰。
