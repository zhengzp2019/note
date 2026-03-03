# Maven 常用编译命令

## 说明

Maven 以模块（module）为最小构建单元，不支持直接只编译某一个 `.java` 文件。  
因此“编译一个类”通常是指“在目标模块内执行编译”。

## 1. 编译指定模块（可选同时编译其依赖模块）

```bash
mvn -pl <module-name> -DskipTests compile
```

```bash
mvn -pl <module-name> -am -DskipTests compile
```

适用场景：只编译某个模块；如果该模块依赖同工程内其他模块，则加 `-am` 一并构建。

参数说明：

- `-pl <module-name>`：仅构建指定模块（`--projects` 的缩写）。
- `-am`：同时构建目标模块依赖的上游模块（`--also-make` 的缩写），可选。
- `-DskipTests`：跳过测试执行，缩短构建时间。
- `compile`：执行 `compile` 生命周期阶段，编译主代码（`src/main/java`）。

## 2. 编译整个项目

```bash
mvn -DskipTests compile
```

适用场景：希望一次性检查整个仓库所有模块的编译状态。

参数说明：

- `-DskipTests`：跳过测试执行。
- `compile`：编译所有模块的主代码。

## 常见补充

- `-DskipTests` 只跳过测试运行，不影响测试代码是否被编译（取决于执行到的生命周期阶段）。
- 如果需要完整构建产物，使用 `package` 或 `install`，例如：`mvn -DskipTests package`。
