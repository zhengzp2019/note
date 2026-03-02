![image](<../../resources/image/img-20251221163556.png>)

# LangChain（大模型应用开发框架）

## 1.1 什么是LangChain

提供了一套高度模块化、可扩展的工具集，让开发人员能够快速、灵活地构建功能丰富的大模型应用，无需关注底层复杂的细节（避免开发人员重复造轮子）

## 1.2 LangChain的优势

![image](<../../resources/image/img-20251221164406.png>)

## 1.3 宏观架构设计

![image](<../../resources/image/img-20251221164424.png>)
上图展示了LangChain（V3.0版本）生态系统的主要组件及其分类，分为三个层次：架构（Architecture）、组件（Components）和部署（deployment）。

## 1.4 核心组件

![image](<../../resources/image/img-20251221165048.png>)

### 1.4.1 model I/O

### 1.4.2 Clains

### 1.4.3 Memory

![image](<../../resources/image/img-20251224095703.png>)

### 1.4.4 Tools

![image](<../../resources/image/img-20251224095756.png>)
使用流程
![image](<../../resources/image/img-20251224095914.png>)

### 1.4.5 Agent

![image](<../../resources/image/img-20251224100228.png>)
**工作模式**

- Function Calling：方法调用模式，可以利用外部提供的一些工具（例如：数据库、文本编辑器等）辅助LLM完成任务。
- ReAct模式（思考与行动）
-

### 1.4.6 Retrieval（RAG）

> 从大量的文档或者数据源中查找相关信息的模块

RAG

**Retrieval的过程**

## 2 实战

## 参考

- [LangChain](https://mp.weixin.qq.com/s/PsUuWLDTYS0O5Ug5285PaA)
