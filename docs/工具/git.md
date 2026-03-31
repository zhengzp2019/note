# git常用指令

## Git 三区

理解 Git 命令之前，先区分 Git 中最常见的三个区域：工作区、暂存区与本地仓库。许多看似相近的命令，本质上是在不同区域之间移动或恢复内容。

### 工作区

工作区是当前实际可见、可编辑的项目目录，即开发者正在直接修改文件的区域。凡是尚未执行 `git add` 的改动，均只存在于工作区。

### 暂存区

暂存区也称为索引区（index），用于保存“准备纳入下一次提交”的文件状态。执行 `git add <file>` 后，Git 会将该文件当前内容写入暂存区，但此时尚未形成正式提交。

### 本地仓库

本地仓库存放于 `.git` 目录中，用于保存提交历史。执行 `git commit` 后，暂存区中的内容会被写入本地仓库，形成一个新的提交记录。

### 三者关系

可用下列流程理解三者之间的关系：

```text
工作区 --git add--> 暂存区 --git commit--> 本地仓库 --git push--> 远程仓库
```

其中，远程仓库不属于本地三区的一部分，但在日常协作中通常会与本地仓库一并讨论。

### 一个简单例子

假设你修改了 `App.java` 与 `UserService.java`，但只执行了：

```bash
git add App.java
```

此时：

1. `App.java` 已进入暂存区。
2. `UserService.java` 仍只存在于工作区。
3. 本地仓库依然保持上一次提交的状态。

若随后执行：

```bash
git commit -m "fix: update app logic"
```

则本次提交只会包含 `App.java` 的改动，而 `UserService.java` 仍未被提交。

## merge

`git merge` 用于将某个分支的整体变更并入当前分支。执行 `merge` 时，Git 会以两个分支的共同祖先为基础，计算差异并完成合并。若当前分支与目标分支均有新增提交，Git 通常会生成一次新的合并提交，以记录两条历史链在此处汇合。

### 常见用法

```bash
# 将指定分支合并到当前分支
git merge <branch_name>

# 创建合并提交，即使可以快进
git merge --no-ff <branch_name>
```

### 特点说明

1. `merge` 的核心在于保留分支关系，适合表达“该功能由独立分支完成后再并入主线”这一事实。
2. 若当前分支没有额外提交，Git 可能直接执行快进合并，此时不会生成新的合并提交。
3. 若两个分支修改了相同代码区域，也可能产生冲突，处理方式与其他 Git 合并操作一致。

### 使用建议

1. 需要完整保留开发轨迹时，应优先考虑 `merge`。
2. 团队协作分支、发布分支或主干分支，通常更适合使用 `merge`，以减少历史改写带来的同步成本。
3. 若希望在历史中明确保留“功能分支合入”这一事件，可使用 `--no-ff`。

## rebase（换基）

`git rebase` 用于将当前分支的一组提交，改为基于另一个提交或分支重新播放。其结果并非简单合并，而是以新的基底重新生成一条提交链，因此提交顺序可能保持不变，但提交 ID 会发生变化。

### 常见用法

```bash
# 将当前分支变基到指定分支之上
git rebase <branch_name>

# 指定新的基底，重放某一段提交
git rebase --onto <new_base> <old_base> <branch_name>
```

例如：

```bash
git rebase --onto HEAD~3 HEAD~2 feature_access_dayu
```

### 特点说明

1. `rebase` 适合在功能分支中整理提交历史，使提交链更加线性、整洁。
2. 由于其本质是“重新生成提交”，因此会改写历史，不应轻易用于已经被多人共享的公共分支。
3. 在代码审阅或问题追踪场景中，线性历史通常更易阅读，但也意味着原始分支汇合关系可能不再直观保留。

### 使用建议

1. 在个人功能分支上同步主干最新变更时，`rebase` 往往比 `merge` 更适合。
2. 在提交合入主干前，若需要压缩、重排或重写提交说明，通常也会配合交互式 `rebase` 使用。
3. 若分支已经推送并被他人基于其继续开发，应谨慎执行 `rebase`，否则后续同步成本会显著增加。

### rebase 交互

交互式 `rebase` 常用于整理提交历史，其典型能力包括压缩提交、调整提交顺序以及重写提交说明。假设当前分支最近三个提交如下：

```text
a1b2c3d feat: add user dto
b2c3d4e fix: rename field
c3d4e5f docs: update api example
```

若希望对这三个提交进行整理，可执行：

```bash
git rebase -i HEAD~3
```

Git 会打开一个待编辑列表，例如：

```text
pick a1b2c3d feat: add user dto
pick b2c3d4e fix: rename field
pick c3d4e5f docs: update api example
```

#### 压缩提交

若第二个提交只是对第一个提交的补充修正，希望将二者合并为一个提交，可改为：

```text
pick a1b2c3d feat: add user dto
squash b2c3d4e fix: rename field
pick c3d4e5f docs: update api example
```

此时 Git 会将第二个提交压缩到第一个提交中，并提示合并提交说明。整理完成后，原先两个零散提交会变成一个新的提交。

#### 重排提交顺序

若希望先提交功能说明文档，再提交字段修正，便可直接调整行顺序：

```text
pick a1b2c3d feat: add user dto
pick c3d4e5f docs: update api example
pick b2c3d4e fix: rename field
```

保存后，Git 会按照新的顺序重新播放这些提交。若提交之间存在依赖关系，重排过程中可能产生冲突，因此仅适用于可安全调整顺序的提交。

#### 重写提交说明

若仅希望修改某个提交的 message，而不改变其内容，可将对应行改为：

```text
reword a1b2c3d feat: add user dto
pick b2c3d4e fix: rename field
pick c3d4e5f docs: update api example
```

保存后，Git 会在重放到该提交时打开提交说明编辑界面，供你将其改写为更准确的描述，例如：

```text
feat: add user dto and validation constraints
```

#### 解决冲突

若在交互式 `rebase` 过程中出现冲突，应先完成冲突解决，再执行：

```bash
git add .
git rebase --continue
```

若希望放弃本次整理过程，可执行：

```bash
git rebase --abort
```

## cherry-pick

`cherry-pick` 用于将一个或多个指定提交的改动，按顺序应用到当前分支。其核心价值在于“选择提交”，而非“合并整个分支”。在多分支协作中，若仅需引入某一项修复、某一段功能提交，或需要将补丁同步至其他分支，`cherry-pick` 通常比 `merge` 更精确，也比 `rebase` 更直接。

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

| 维度             | cherry-pick                          | merge                                   | rebase                                     |
| ---------------- | ------------------------------------ | --------------------------------------- | ------------------------------------------ |
| 处理对象         | 指定的一个或多个提交                 | 整个分支的变更集合                      | 当前分支的一组提交                         |
| 核心作用         | 将选定提交复制到当前分支             | 将一个分支整体并入当前分支              | 将当前分支提交改为基于新的基底重新播放     |
| 历史表现         | 生成新的提交，保留局部改动           | 保留原始分支拓扑，通常形成合并提交      | 改写提交历史，使提交链更线性               |
| 提交 ID 是否变化 | 会变化                               | 原有提交通常不变，可能新增 merge commit | 会变化                                     |
| 适用场景         | 补丁迁移、热修复同步、选择性引入功能 | 功能分支整体合并、集成分支收敛          | 合并前整理历史、同步最新主干、调整提交基底 |
| 主要优势         | 精确、局部、可控                     | 历史信息完整，分支关系清晰              | 历史线性，阅读与追踪更简洁                 |
| 主要风险         | 若遗漏依赖提交，可能导致功能不完整   | 历史可能出现较多合并节点                | 会改写历史，不宜随意用于已共享分支         |

若需要“只拿部分提交”，应优先考虑 `cherry-pick`；若需要“整合整个分支”，应优先考虑 `merge`；若需要“整理历史并重新建立基底”，则应优先考虑 `rebase`。

## restore

`restore` 用于将文件恢复到某个已知状态。其主要用途包括：撤销工作区中的未提交修改、将暂存区中的文件恢复为未暂存状态，以及从指定分支或指定提交中提取某个文件版本。与 `cherry-pick` 复制“提交”不同，`git restore` 处理的是“文件内容”。

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

## rm（从索引树中删除文件）

`git rm` 用于删除已被 Git 跟踪的文件。与直接执行系统层面的 `rm` 不同，`git rm` 不仅会删除工作区中的文件，还会将“该文件已被删除”这一变化写入暂存区，以便在下一次提交中生效。

### 常见用法

```bash
# 删除单个文件，并将删除操作加入暂存区
git rm <file>

# 递归删除目录
git rm -r <dir>

# 仅从 Git 跟踪中移除，但保留本地文件
git rm --cached <file>
```

### 典型场景

1. 需要正式删除某个已纳入版本管理的文件或目录。
2. 某个文件不应继续被 Git 跟踪，但希望保留本地副本，例如将配置文件加入 `.gitignore` 之前先取消跟踪。
3. 需要在一次提交中明确记录“删除文件”这一变更。

### 使用说明

1. `git rm <file>` 会同时影响工作区与暂存区，即本地文件会被删除，且删除状态会进入暂存区。
2. `git rm --cached <file>` 只会将文件从 Git 跟踪中移除，不会删除工作区中的实际文件。
3. 删除目录时通常需要配合 `-r`，否则 Git 不会递归处理其中的文件。

### 注意事项

1. 若文件内容已有修改，Git 可能拒绝直接执行 `git rm`，以避免误删未提交内容；此时应先确认是否需要保留改动。
2. 若确实需要强制删除，可使用 `git rm -f <file>`，但这一操作应谨慎执行。
3. 若只是希望删除工作区中的未跟踪文件，`git rm` 并不适用，这类场景通常应使用 `git clean`。

## blame（定位每行最后修改人）

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
