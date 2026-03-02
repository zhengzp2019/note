# java8 新特性

lambda 简化写法：类名：：方法名（只适用于一个或零个参数的方法）

```java
// 1. 遍历场景
list.forEach(item -> System.out.println(item));

// 简化写法
list.forEach(System.out::println);

// 2. 转换场景
List<String> strings = Arrays.asList("1", "2", "3");

// 转换为 Integer 列表
List<Integer> numbers = strings.stream()
    .map(Integer::parseInt)
    .collect(Collectors.toList());
    
// 3. 排序
List<String> list = Arrays.asList("banana", "apple", "cherry");

// 按长度排序
list.sort(String::compareTo);

// 或
list.stream().sorted(String::compareToIgnoreCase).collect(Collectors.toList());

```
