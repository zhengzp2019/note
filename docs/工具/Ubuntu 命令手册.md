# Linux 学习

## 包管理

```shell
apt
yum
```

## 远程连接与 SSH

```shell
## 查看 ssh 服务是否启动
sudo systemctl is-active ssh

## 重启 ssh 服务
/etc/init.d/ssh restart

## 生成 ssh 密钥
ssh-keygen -t rsa -b 4096
```

## 网络排查与下载

```shell
netstat
ping
curl
telnet
wget
```

## Shell 与基础统计

```shell
## 统计目录下文件数目
ls -l | wc -l
```

## 文本搜索（grep）

```shell
## 在文件中搜索关键字
grep "error" app.log

## 忽略大小写搜索
grep -i "warning" app.log

## 显示行号
grep -n "port" nginx.conf

## 递归搜索目录下所有文件
grep -rn "TODO" .

## 只显示匹配到的文件名
grep -rl "spring.datasource" .

## 反向匹配（排除包含关键字的行）
grep -v "^#" /etc/ssh/sshd_config

## 显示匹配行前后文
grep -n -C 2 "Exception" app.log

## 正则匹配
grep -E "^(?!#).*" /etc/ssh/sshd_config
```


### rg 与 grep 的区别（推荐优先用 rg）

1. 相同点：都能按关键字搜索文本，也都支持正则。
2. 不同点：
   - `rg`（ripgrep）默认递归搜索、速度更快、默认遵守 `.gitignore`。
   - `grep` 需要加 `-r/-R` 才递归，默认不遵守 `.gitignore`，兼容性更强（系统基本自带）。
3. 选择建议：
   - 代码仓库检索优先 `rg`（快、噪声少）。
   - 服务器最小环境或脚本兼容性优先 `grep`。

```shell
# rg：默认递归（当前目录）
rg "ERROR"

# grep：递归搜索需加 -r，并建议加 -n 看行号
grep -rn "ERROR" .

# rg 只看文件名（等价 grep -rl）
rg -l "spring.datasource"

# grep 只看文件名
grep -rl "spring.datasource" .
```
## 文本处理（awk）

```shell
## 按空格分隔，打印第 1 和第 4 列
awk '{print $1, $4}' access.log

## 指定分隔符（例如 :）
awk -F ":" '{print $1}' /etc/passwd

## 统计文件行数
awk 'END {print NR}' app.log

## 统计某列求和（第 3 列）
awk '{sum += $3} END {print sum}' data.txt

## 条件过滤（第 9 列状态码为 500）
awk '$9 == 500 {print $0}' access.log

## 打印文件最后一行
awk 'END {print}' app.log
```

## 文本编辑（vim）

```shell
## 打开文件
vim test.txt

## 模式切换（普通模式按 Esc 返回）
i
a            # 在光标后插入
o            # 在下一行插入

## 光标移动
h j k l      # 左 下 上 右
w / b        # 下一个词 / 上一个词
0 / $        # 行首 / 行尾
gg / G       # 文件开头 / 文件末尾

## 常用编辑
yy           # 复制当前行
p            # 粘贴到下一行
dd           # 删除当前行
u            # 撤销
Ctrl+r       # 重做

## 搜索与替换
/keyword     # 向下搜索
?keyword     # 向上搜索
n / N        # 下一个 / 上一个匹配
:%s/old/new/g      # 全文替换
:%s/old/new/gc     # 全文替换并逐个确认

## 显示与辅助
:set number          # 显示行号
:set relativenumber  # 显示相对行号

## 保存与退出
:w            # 保存
:q            # 退出
:wq           # 保存并退出
:q!           # 不保存强制退出
```

## 日志查看（tail）

```shell
## 查看文件最后 10 行（默认）
tail app.log

## 查看最后 50 行
tail -n 50 app.log

## 实时跟踪日志
tail -f app.log

## 实时跟踪并显示最近 100 行
tail -n 100 -f app.log

## 从第 200 行开始向后显示
tail -n +200 app.log

## 配合 grep 过滤关键字
tail -f app.log | grep "ERROR"
```

## 文本搜索（grep）- Kubernetes 排查学习版

### 1. 先记住这 6 个高频参数

```shell
-n   # 显示行号，便于回看上下文
-i   # 忽略大小写
-E   # 使用扩展正则（支持 | 分组）
-v   # 反向匹配（排除）
-C 2 # 显示匹配行前后 2 行
-r   # 递归搜索目录
```

### 2. 本次实战中的 grep 用法拆解

```shell
# 1) 在 kubectl 输出中筛选目标 Pod
kubectl get pods -n feature-worker | grep zhengzhanpeng.1029-ops-swift.default--broker
```

用途：从大量 Pod 中快速锁定异常对象。

```shell
# 2) 一次匹配多个关键词（扩展正则）
kubectl get pods -n feature-worker -o wide | grep -E "john-ops-swift.default--broker|zhengzhanpeng.1029-ops-swift.default--broker"
```

用途：并排比较“正常实例 vs 异常实例”。

```shell
# 3) 在应用日志里抓 ERROR 链路
grep -nEi "delete table|drop table|drop build|delete topic|Failed to .*table|ERROR" tianji-ops-controller/logs/tianji-ops-dev.log
```

用途：从整份日志提取删除/建表相关失败。

```shell
# 4) 抽取建表流程关键阶段
grep -nE "Start to create table|operation: create table|operation: get build Request|operation: get build Response|Failed to generate build task" tianji-ops-controller/logs/tianji-ops-dev.log
```

用途：把流程日志串成一条时序线。

```shell
# 5) 只看某个 broker 相关日志
grep -n "broker_2_0" ./logs/swift/swift.log
```

用途：按实例维度缩小排查范围。

```shell
# 6) 过滤错误级别
grep -nE "ERROR|FATAL|fail|Failed" ./logs/swift/swift.log
```

用途：优先看失败点，再回头补看上下文。

### 3. 可直接复用的 grep 模板

```shell
# 模板 A：多关键词匹配（最常用）
grep -nE "kw1|kw2|kw3" <file>

# 模板 B：忽略大小写 + 行号
grep -nEi "error|failed|exception" <file>

# 模板 C：先筛再看上下文
grep -nE "<pattern>" <file>
grep -n -C 3 "<exact_line_part>" <file>

# 模板 D：和 kubectl 组合
kubectl get pods -n <ns> | grep <keyword>
kubectl logs -n <ns> <pod> | grep -nE "ERROR|FATAL|fail|Failed"
```

### 4. 常见误区

1. 忘了 `-E`，导致 `|` 被当普通字符。
2. 关键词太宽（如只搜 `error`），结果噪声太大。
3. 只搜到一行就下结论，没有回看上下文（建议配合 `-C`）。
4. 不加 `-n`，后续定位源日志困难。

### 5. 一个实战排查套路（建议背下来）

1. `kubectl get ... | grep ...` 先定位对象。
2. `kubectl logs ... | grep -nE "ERROR|FATAL|fail|Failed"` 找错误行。
3. 回原日志用 `grep -n -C 3` 看前后文。
4. 再按业务关键词（如 `create table`/`delete table`）串流程。

## grep 实战过程（结合本次排查）

### 目标

在大量日志中快速回答三个问题：

1. 哪个对象在报错？
2. 报错发生在流程哪一步？
3. 是配置问题还是运行时偶发问题？

### 实战步骤

1. 先从对象名过滤。
   - `kubectl get pods -n feature-worker | grep zhengzhanpeng.1029-ops-swift.default--broker`
   - 作用：先缩小到异常实例。

2. 再抓错误级别关键词。
   - `grep -nE "ERROR|FATAL|fail|Failed" ./logs/swift/swift.log`
   - 作用：快速拿到候选失败点。

3. 用业务关键词串时序。
   - `grep -nE "Start to create table|operation: create table|operation: get build Request|operation: get build Response|Failed to generate build task" tianji-ops-controller/logs/tianji-ops-dev.log`
   - 作用：判断失败前后发生了什么。

4. 针对删除链路单独筛。
   - `grep -nEi "delete table|drop table|drop build|delete topic|Failed to .*table|ERROR" tianji-ops-controller/logs/tianji-ops-dev.log`
   - 作用：验证 delete table 的失败点是否可重复。

5. 精准到单实例关键词。
   - `grep -n "broker_2_0" ./logs/swift/swift.log`
   - 作用：避免被其他 broker 日志干扰。

### 实战结论

- `grep` 的核心不是“多写几个命令”，而是先定关键词集合，再按对象名和阶段词逐步收敛。
- 最有效组合：`对象过滤 -> 错误过滤 -> 业务阶段过滤 -> 上下文复核`。

## 文本流编辑（sed）

### 1. 基本语法与常用参数

```shell
sed [选项] '脚本' 文件

-n   # 关闭默认输出，配合 p 只打印命中内容
-E   # 启用扩展正则（支持 +、|、()）
-i   # 原地修改文件（建议先备份）
```

### 2. 替换（s 命令）

```shell
# 每行只替换第一个 old
sed 's/old/new/' app.log

# 每行全部替换
sed 's/old/new/g' app.log

# 只替换每行第 2 个匹配
sed 's/old/new/2' app.log

# 只替换第 3 行
sed '3s/old/new/' app.log

# 只替换第 10~30 行
sed '10,30s/old/new/g' app.log
```

### 3. 按条件匹配后再操作

```shell
# 仅对包含 ERROR 的行做替换
sed '/ERROR/s/timeout/retry/g' app.log

# 打印从 start 到 end 的区间
sed -n '/start/,/end/p' app.log

# 仅打印匹配 create table 的行
sed -n '/create table/p' app.log
```

### 4. 删除与提取

```shell
# 删除第 5 行
sed '5d' app.log

# 删除第 10~20 行
sed '10,20d' app.log

# 删除空行
sed '/^$/d' app.log

# 只看前 20 行
sed -n '1,20p' app.log

# 只看最后 20 行（和 tail 等价思路不同，需先拿总行数）
# 更推荐：tail -n 20 app.log
```

### 5. 插入与追加

```shell
# 在第 3 行前插入
sed '3i\\# inserted line' conf.txt

# 在第 3 行后追加
sed '3a\\# appended line' conf.txt
```

### 6. 原地修改（谨慎）

```shell
# 直接改文件
sed -i 's/127.0.0.1/0.0.0.0/g' config.ini

# 先备份再改（推荐）
sed -i.bak 's/127.0.0.1/0.0.0.0/g' config.ini
```

### 7. 与日志排查组合（实用模板）

```shell
# 提取 ERROR 附近 3 行（grep 更强，这里演示 sed 范围能力）
sed -n '/ERROR/,+3p' app.log

# 只看某时间段日志（按固定前缀）
sed -n '/2026-03-04 10:00/,/2026-03-04 10:30/p' app.log

# 先 grep 再 sed 精简字段
grep -n "ERROR" app.log | sed -E 's/^([0-9]+):/line=\\1 /'
```

### 8. 常见坑

1. `sed -i` 会直接改原文件，生产环境先 `-i.bak`。
2. 正则分组与 `+` 需要 `-E`，否则匹配结果可能不符合预期。
3. 路径或替换文本含 `/` 时，建议改分隔符：`sed 's#/old/path#/new/path#g'`。
4. `grep -n` 的行号是原文件行号；如果是 `tail | grep -n`，行号是管道结果内的相对行号。
