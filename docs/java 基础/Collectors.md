# Collectors

 ## 介绍
一般用于流处理中的最后一步。Stream.collect() 是java 8 stream api中的终止方法。它实现对流实例中保存的数据元素执行可变的折叠操作。此操作的策略由Collector接口实现提供。

## 常见的 Collectors静态方法


| 方法名       | 功能                                           |
| --------- | -------------------------------------------- |
| toList()  | 将 stream 中的元素收集到 list 实例。该方法不能作用于 List 的实现类。 |
| toSet()   | 将 stream 中的元素收集到 set 实例。同上。                  |
| toMap()   | 将 stream 中的元素收集到 map 实例。同上。                  |
| mapping() | 把流中的元素映射为另一个类型，然后用下游收集器收集。                   |
