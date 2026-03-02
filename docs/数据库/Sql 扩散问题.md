# Sql 扩散问题

## 1、什么是Left Join扩散问题

## 2、扩散问题的常见场景

| 场景                  | 说明                                                           |
| --------------------- | -------------------------------------------------------------- |
| 1：N关联 + 聚合       | 如：用户left join订单 -> 一用户多订单，用户信息重复            |
| N：M（中间表）        | 如：文章Left Join 标签关系表 -> 一篇文章多个标签，文章内容重复 |
| 多层Left Join级联     | A Left Join B (1:N) Left Join C(1:N) -> 行数 = A x B x C       |
| 用Left join代替子查询 |                                                                |

## 3、扩散问题 vs Left Join vs Inner Join

|          | Left Join 扩散                                                          | Inner Join 扩散           |
| -------- | ----------------------------------------------------------------------- | ------------------------- |
| 行数变化 | 左表行数 <= 结果行数                                                    | 结果行数 = 匹配的笛卡尔积 |
| 空置风险 | 左表无匹配是补NULL                                                      | 无匹配则整行消失          |
| 常见错误 | 统计左表字段是重复计算                                                  | 误认为1:1，实际上1:N      |
| 检查方法 | `SELECT COUNT(*) FROM left_table` vs `SELECT COUNT(*) FROM join_result` | 同左                      |

## 4、解法

先聚合右表，在join。

```sql
SELECT
    o.order_id,
    o.amount,
    COALESCE(oi.item_count, 0) AS item_count,
    COALESCE(oi.total_quantity, 0) AS total_quantity
FROM orders o
LEFT JOIN (
    SELECT
        order_id,
        COUNT(*) AS item_count,
        SUM(quantity) AS total_quantity
    FROM order_items
    GROUP BY order_id
) oi ON o.order_id = oi.order_id;
```
