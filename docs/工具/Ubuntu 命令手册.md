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

## 进入插入模式
i

## 保存并退出
:wq

## 不保存退出
:q!

## 搜索关键字（向下/向上）
/keyword
?keyword

## 替换（全文）
:%s/old/new/g

## 显示行号
:set number

## 复制/粘贴/删除当前行
yy
p
dd
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
