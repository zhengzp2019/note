# git

## 合并指定提交（cherry-pick）

`git cherry-pick` 用于将一个或多个指定提交的改动，按顺序应用到当前分支。其核心价值在于“选择提交”，而非“合并整个分支”。在多分支协作中，若仅需引入某一项修复、某一段功能提交，或需要将补丁同步至其他分支，`cherry-pick` 通常比 `merge` 更精确，也比 `rebase` 更直接。

### 常见用法

```bash
# 将单个提交应用到当前分支
git cherry-pick <commit_id>

# 一次应用多个提交
git cherry-pick <commit_id_1> <commit_id_2> <commit_id_3>

# 应用一段连续提交
git cherry-pick <start_commit>^..<end_commit>
```

### 其他用法

```bash
# 应用并提交，但允许编辑提交说明
git cherry-pick --edit <commit_id>

# 将改动应用到工作区和暂存区，不立即生成提交
git cherry-pick --no-commit <commit_id>
```

其中，`--no-commit` 适用于以下场景：

1. 需要将多个提交整理为一个新的提交。
2. 需要在应用改动后，进一步删减、补充或调整代码内容。
3. 不希望保留原有提交边界，仅希望保留最终结果。

### 冲突处理

当目标分支与待拣选提交修改了相同代码区域时，`cherry-pick` 可能产生冲突。此时可按以下流程处理：

```bash
# 解决冲突后，标记为已处理
git add .

# 继续执行 cherry-pick
git cherry-pick --continue

# 放弃本次 cherry-pick
git cherry-pick --abort
```

### 使用建议

1. `cherry-pick` 适用于“选择少量提交”的场景，不适用于整体分支合并。
2. 若某个提交依赖其前序提交，应一并拣选，否则可能出现编译错误或行为不完整。
3. `cherry-pick` 会在当前分支生成新的提交，提交内容可以相同，但提交 ID 不会与原分支一致。

## cherry-pick、merge 与 rebase 的区别

`cherry-pick`、`merge` 与 `rebase` 都可用于整合代码，但处理对象、历史表现以及适用场景并不相同。

### 1. cherry-pick

作用：选择指定提交，并将其改动复制到当前分支。  
特点：精确、局部、可控，不会直接引入整个分支历史。  
适用场景：补丁迁移、热修复同步、跨分支选择性引入功能。

### 2. merge

作用：将一个分支的整体变更合并到当前分支。  
特点：保留原始分支拓扑，通常会形成一次合并提交，历史关系最完整。  
适用场景：功能分支开发完成后，整体并入主干或集成分支。

常见命令如下：

```bash
git merge <branch_name>
```

### 3. rebase（换基）

作用：将当前分支的一组提交，改为基于另一个分支重新播放。  
特点：会改写提交历史，使历史更线性，但提交 ID 会发生变化。  
适用场景：在合并前整理分支历史，或使当前分支基于最新主干继续开发。

常见命令如下：

```bash
git rebase <branch_name>
```

```bash
git rebase --onto HEAD~3 HEAD~2 feature_access_dayu
```

<https://chat.qwen.ai/s/634e2064-74b6-48df-b367-ea0deeced80b?fev=0.1.30>

### 核心差异总结

1. 若需要“只拿部分提交”，优先考虑 `cherry-pick`。
2. 若需要“整合整个分支”，优先考虑 `merge`。
3. 若需要“整理提交历史并重新建立基底”，优先考虑 `rebase`。
4. `merge` 侧重保留真实历史，`rebase` 侧重形成线性历史，`cherry-pick` 侧重精确提取改动。
5. `rebase` 与 `cherry-pick` 都会生成新的提交，因此提交 ID 会变化；`merge` 通常不会重写原有提交。

## 恢复工作区或索引中的文件（git restore）

`git restore` 用于将文件恢复到某个已知状态。其主要用途包括：撤销工作区中的未提交修改、将暂存区中的文件恢复为未暂存状态，以及从指定分支或指定提交中提取某个文件版本。与 `cherry-pick` 复制“提交”不同，`git restore` 处理的是“文件内容”。

### 常见用法

```bash
# 撤销工作区中某个文件的修改
git restore <file>

# 一次恢复多个文件
git restore <file_1> <file_2>

# 撤销当前目录下全部未提交修改
git restore .
```

上述命令主要影响工作区，适用于“尚未提交，但希望放弃当前修改”的场景。

### 撤销暂存

```bash
# 将文件从暂存区恢复到未暂存状态
git restore --staged <file>

# 同时撤销暂存与工作区修改
git restore --staged --worktree <file>
```

其中：

1. `--staged` 用于恢复索引区。
2. `--worktree` 用于恢复工作区。
3. 二者同时使用时，相当于同时撤销暂存与本地修改。

### 从其他分支或提交恢复文件

```bash
# 将指定分支中的文件恢复到当前分支工作区
git restore --source <branch_name> -- <file>

# 将指定提交中的文件恢复到当前分支工作区
git restore --source <commit_id> -- <file>
```

这一用法常用于以下场景：

1. 需要将另一个分支中的单个文件复制到当前分支。
2. 需要取回某次历史提交中的文件版本。
3. 不希望合并整条分支历史，仅希望恢复某个文件的内容。

### 使用建议

1. `git restore` 更适合文件级恢复，不适合跨分支迁移一组提交。
2. 若目标是引入某次提交的完整改动，应优先考虑 `cherry-pick`，而非 `restore`。
3. 在执行 `git restore .` 之前，应确认工作区中不存在仍需保留的未提交修改。

## 从索引树中删除文件

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
