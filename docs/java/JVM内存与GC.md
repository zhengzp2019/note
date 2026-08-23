# JVM 内存与 GC

## JVM 运行时内存区域

| 区域              | 存什么                     | 是否线程共享 | GC 是否管理     |
| ----------------- | -------------------------- | ------------ | --------------- |
| 堆(Heap)          | 对象实例、数组             | 共享         | 是（GC 主战场） |
| 元空间(Metaspace) | 类的元数据                 | 共享         | 是（类卸载时）  |
| 虚拟机栈          | 栈帧（局部变量、操作数栈） | 线程私有     | 否              |
| 本地方法栈        | native 方法栈帧            | 线程私有     | 否              |
| 程序计数器        | 当前线程执行的字节码行号   | 线程私有     | 否              |

## 堆的分代结构

堆分为**年轻代(Young)**和**老年代(Old)**。

```
堆(Heap)
├── 年轻代 Young Generation
│   ├── Eden 区          ← 新对象在这里分配
│   ├── Survivor 0 (S0)  ← from
│   └── Survivor 1 (S1)  ← to
└── 老年代 Old Generation ← 长期存活对象 / 大对象
```

默认比例：Eden : S0 : S1 = 8 : 1 : 1（可用 `-XX:SurvivorRatio` 调整）。

### 对象在分代间的流转

1. 新对象优先在 **Eden** 分配。
2. Eden 满 → 触发 **Young GC**，存活对象复制到 Survivor，年龄 +1。
3. 每经历一次 Young GC 仍存活，年龄 +1；年龄达到阈值（默认 15，`-XX:MaxTenuringThreshold`）→ **晋升到老年代**。
4. 大对象（超过 `-XX:PretenureSizeThreshold`）直接进老年代，避免在 Survivor 间反复复制。
5. Survivor 区放不下（同龄对象总大小超过 Survivor 一半）→ 提前晋升。

## Young GC（Minor GC）

- **触发**：Eden 区满时。
- **范围**：只回收年轻代。
- **算法**：复制算法（Eden + 一个 Survivor 中的存活对象复制到另一个 Survivor）。
- **特点**：频繁、速度快（年轻代对象大多朝生夕死，存活率低）、STW 时间短。

## Full GC

- **触发**（常见原因）：
  - 老年代空间不足（对象晋升失败）。
  - 元空间(Metaspace)不足。
  - 显式调用 `System.gc()`。
  - CMS 的 promotion failed / concurrent mode failure。
- **范围**：整个堆（年轻代 + 老年代）+ 元空间。
- **特点**：慢、STW 时间长（可能几百 ms 到数秒）。**频繁 Full GC 是性能问题的重要信号**。

### Young GC 与 Full GC 对比

|          | Young GC | Full GC                           |
| -------- | -------- | --------------------------------- |
| 回收范围 | 年轻代   | 整个堆 + 元空间                   |
| 频率     | 高       | 低（正常情况）                    |
| STW 时长 | 短       | 长                                |
| 触发条件 | Eden 满  | 老年代/元空间不足、System.gc() 等 |

## 元空间(Metaspace)

JDK8 起用 **Metaspace 取代永久代(PermGen)**，位于**本地内存(native memory)**，不再受堆大小限制，默认上限受物理内存约束（可用 `-XX:MaxMetaspaceSize` 限定）。

### 元空间存什么

- **类的结构元数据(Klass)**：类名、父类、接口、修饰符、字段定义、方法信息（签名、字节码、异常表）。
- **运行时常量池**：字面量、符号引用。（注意：字符串常量池 JDK7 后已移到堆）
- **方法的字节码、JIT 编译相关数据**。
- 元数据的生命周期**跟随 ClassLoader**：只有 ClassLoader 被回收，它加载的所有类元数据才会卸载。

### 关键认知

- **对象再多不占元空间**（那是堆的事）。元空间只和**类的数量**有关。
- **类越多、加载越频繁，元空间涨得越快**。
- 元空间 OOM 几乎总是**类泄漏**：动态生成类（fastjson ASM、cglib、$Proxy）挂在长期存活的 ClassLoader 上永不卸载，只涨不降最终撑爆。

## 元空间参数：MetaspaceSize vs MaxMetaspaceSize

```
-XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m
```

容易被名字误导：`MetaspaceSize` **不是「初始大小」**，而是**首次触发 Metaspace GC(类卸载)的阈值**。

### MaxMetaspaceSize —— 硬上限

- 元空间能占用的**最大值**，天花板。
- 用量顶到该值且 GC 也卸载不掉类 → 抛 `OutOfMemoryError: Metaspace`。
- 不设默认几乎无限(受本地内存约束)，容器里危险，应显式限定。

### MetaspaceSize —— GC 触发阈值(高水位线)

- **不是**启动时就分配这么多，而是用量**第一次涨到该值时触发一次 Metaspace GC**，尝试卸载不再使用的类。
- 这次 GC 后 JVM 会根据「释放了多少」**动态调整水位线**：
  - 释放得多 → 水位线**下调**。
  - 释放得少(类大多还在用) → 水位线**上调**(≤ Max)，避免频繁 GC。
- 更像「开始关注并回收类的起点」，而非固定尺寸。

### 类比与行为

| 参数                  | 类比                                   |
| --------------------- | -------------------------------------- |
| MetaspaceSize=128m    | 水位涨到 128m 就**拉警报做清理**       |
| MaxMetaspaceSize=256m | 水池**最大只能装 256m**，溢出就爆(OOM) |

```
0 ──涨──> 128m ─┐
               │ 触发 Metaspace GC，卸载无用类
               ├─ 释放多 → 水位线下调
               └─ 释放少 → 水位线上调(≤256m)
继续增长 ... 最终顶到 256m 仍卸不掉 → OOM: Metaspace
```

### 调优建议

保持 `MetaspaceSize` 略高于常态基线(减少早期无谓的类卸载)、`Max` 留足余量。若基线 MU ≈ 214m：

```
-XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m
```

### 参数优先级（排查易踩的坑）

JVM 读参数有优先级，后读的覆盖先读的：

```
JAVA_TOOL_OPTIONS   ← 最低优先级
命令行 -XX 参数       ← 覆盖上面
_JAVA_OPTIONS       ← 最高
```

- 启动日志里 `Picked up JAVA_TOOL_OPTIONS: ...` 只是打印**该环境变量的内容**，不代表最终生效值——命令行 `-XX` 会覆盖它。
- 曾遇到 `JAVA_TOOL_OPTIONS` 里被基础镜像注入 `MaxMetaspaceSize=1m`，但实际生效的是启动脚本命令行里的 `256m`。判断真实生效值应以命令行参数为准，或用 `jcmd <pid> VM.flags` 确认。

## 元空间问题排查

背景：`java.lang.OutOfMemoryError: Metaspace`，常因元空间上限偏低或类泄漏。以下命令 `<pid>` 换成实际进程号。

### 1. jstat —— 看实时使用量和 GC（首选）

```bash
# 单次看 Metaspace 容量和使用（单位 KB）
jstat -gcmetacapacity <pid>

# 每 1 秒一次，共 10 次，盯 MC / MU
jstat -gc <pid> 1000 10
```

| 列          | 含义                                            |
| ----------- | ----------------------------------------------- |
| MC          | Metaspace 当前容量(committed, KB)               |
| MU          | Metaspace 已使用(KB)                            |
| MCMN / MCMX | 最小 / 最大容量(MCMX 对应 -XX:MaxMetaspaceSize) |
| CCSC / CCSU | Compressed Class Space 容量 / 使用              |
| FGC / FGCT  | Full GC 次数 / 耗时                             |

判断：

- MU 贴着 MCMX 不降 + FGC 频繁 → 元空间满了，正在疯狂 Full GC。
- MU 随时间只涨不降 → 疑似类泄漏。

### 2. jstat -class —— 判断是否类泄漏（铁证）

```bash
jstat -class <pid> 2000 20
```

| 列       | 含义           |
| -------- | -------------- |
| Loaded   | 累计加载的类数 |
| Unloaded | 累计卸载的类数 |

判断：**Loaded 持续上涨、Unloaded 几乎不动** → 有东西不停产类却不卸载，确认泄漏。

### 3. jcmd —— 定位是哪些类 / 哪个 ClassLoader

```bash
# 类直方图：哪些类占多少（能看到重复的动态类名）
jcmd <pid> GC.class_histogram | head -50

# 各 ClassLoader 加载了多少类、占多少元空间
jcmd <pid> VM.classloader_stats

# 看 Metaspace 相关启动参数
jcmd <pid> VM.flags | tr ' ' '\n' | grep -i meta
```

**泄漏铁证**：class_histogram 里出现大量**同名前缀带序号**的类：

- `com.alibaba.fastjson.serializer.ASMSerializer_*` / `ASMDeserializer_*`
- `$Proxy1234`、`GeneratedMethodAccessor*`
- `...$$EnhancerByCGLIB$$*`

说明有代码在反复动态生成类。

### 4. NativeMemoryTracking —— 最精确（需重启加参数）

```bash
# 启动参数加：-XX:NativeMemoryTracking=summary
jcmd <pid> VM.native_memory summary | grep -A5 -i "Class\|Metaspace"
```

### 排查顺序

1. `jstat -gc <pid> 1000 10` 看 MU/FGC → 判断现在是否满了。
2. `jstat -class <pid>` 看 Loaded/Unloaded → 判断是否泄漏。
3. 若泄漏：`jcmd GC.class_histogram` + `VM.classloader_stats` → 定位是哪个库 / ClassLoader 在产类。
4. 若只是上限低：调大 `-XX:MaxMetaspaceSize`（如 256m → 512m）。

### 常见类泄漏根因

- **fastjson** 用 ASM 为反序列化类型生成类；若每次都 `new TypeReference<>(){}` 构造匿名类型 → 持续产类。
- **cglib / 动态代理**：反复创建代理类。
- **反复 new ClassLoader** 但不释放。
- **Groovy / 脚本引擎**：每次编译脚本生成新类。

## 元空间满导致的连锁故障（真实案例）

一次生产事故的因果链：

1. 元空间(256m)耗尽 → `OutOfMemoryError: Metaspace`。
2. JVM 加载新类失败 / 频繁 Full GC → 某个周期任务实例**卡死收不了尾**。
3. 调度框架(EasyJob)仍认为该实例「运行中」，后续每次触发的任务因配置 `concurrency=false` 被 **non-concurrency 中止**（`abort because of non-concurrency`）。
4. worker 线程池 10 个线程集体 **WAITING** 空转，机器**假死**。

排查提示：

- 调度任务大面积 `abort because of non-concurrency` → 先找那个「一直运行中不结束」的实例，它是元凶。
- 线程监控里全 WAITING 且「正在执行任务」为空，可能是卡死的业务任务不在该 worker 池，而在业务自己的异步线程池。
- 应急：先重启 / 摘除不健康实例恢复调度，再定位元空间泄漏根因。
