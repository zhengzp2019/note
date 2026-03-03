# Kubernetes 学习

## 常用命令

### 1. 查看命名空间下的 Pods

命令：
`kubectl get pods -n <namespace>`

参数说明：

- `get`：查询资源列表。
- `pods`：资源类型，表示 Pod。
- `-n`：`--namespace` 的简写，指定命名空间。
- `<namespace>`：命名空间名称，例如 `default`、`kube-system`。

示例：
`kubectl get pods -n default`

### 2. 查看 Pod 详细信息

命令：
`kubectl describe pod <pod-name> -n <namespace>`

参数说明：

- `describe`：查看资源的详细描述（事件、状态、容器信息等）。
- `pod`：资源类型，表示单个 Pod。
- `<pod-name>`：Pod 名称。
- `-n`：`--namespace` 的简写，指定命名空间。
- `<namespace>`：命名空间名称。

示例：
`kubectl describe pod nginx-6f7d7cbdc8-abcde -n default`

### 3. 删除 Pod

命令：
`kubectl delete pod <pod-name> -n <namespace>`

参数说明：

- `delete`：删除指定资源。
- `pod`：资源类型，表示单个 Pod。
- `<pod-name>`：要删除的 Pod 名称。
- `-n`：`--namespace` 的简写，指定命名空间。
- `<namespace>`：命名空间名称。

示例：
`kubectl delete pod nginx-6f7d7cbdc8-abcde -n default`

### 4. 实时查看 Pod 日志

命令：
`kubectl logs -f <pod-name> -n <namespace>`

参数说明：

- `logs`：查看 Pod 日志。
- `-f`：`--follow` 的简写，持续输出日志（类似 `tail -f`）。
- `<pod-name>`：Pod 名称。
- `-n`：`--namespace` 的简写，指定命名空间。
- `<namespace>`：命名空间名称。

示例：
`kubectl logs -f nginx-6f7d7cbdc8-abcde -n default`
