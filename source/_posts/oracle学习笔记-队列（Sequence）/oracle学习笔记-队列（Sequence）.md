---
title: oracle学习笔记-队列（Sequence）
date: 2025-04-13 16:56:42
updated: 2025-04-13 16:56:42
tags:
 - oracle
categories:
 - 技术
 - oracle
keywords:
 - oracle
 - Sequence
 - 队列
description:
---
Oracle 中的 **Sequence（序列）** 是一个数据库对象，用于生成唯一的、递增（或递减）的数值序列。它通常用于为表的主键字段自动生成唯一值（如自增ID），避免手动管理主键的复杂性。

---

### **Sequence 的核心特点**
1. **唯一性**：保证生成的数值全局唯一。
2. **高性能**：通过缓存机制减少磁盘I/O，提升生成效率。
3. **独立性**：不依赖表，多个表可共享同一个序列。
4. **可配置性**：可定义起始值、步长、循环等规则。

---

### **Sequence 的创建语法**
```sql
CREATE SEQUENCE sequence_name
  [START WITH n]         -- 起始值（默认 1）
  [INCREMENT BY n]       -- 步长（默认 1，可为负数）
  [MINVALUE n | NOMINVALUE]  -- 最小值
  [MAXVALUE n | NOMAXVALUE]  -- 最大值
  [CACHE n | NOCACHE]    -- 缓存值数量（默认 20，提升性能）
  [CYCLE | NOCYCLE]      -- 是否循环（达到极值后是否重置）
  [ORDER | NOORDER];     -- 是否保证顺序（多实例环境下）
```

---

### **示例：创建一个简单序列**
```sql
CREATE SEQUENCE employee_id_seq
  START WITH 1000
  INCREMENT BY 1
  NOCACHE
  NOCYCLE;
```

---

### **Sequence 的常用操作**
#### 1. **获取下一个值**
使用 `NEXTVAL` 生成下一个值：
```sql
SELECT employee_id_seq.NEXTVAL FROM dual; -- 返回 1000, 1001, 1002...
```

#### 2. **获取当前值**
使用 `CURRVAL` 查看当前值（需在当前会话中已调用过 `NEXTVAL`）：
```sql
SELECT employee_id_seq.CURRVAL FROM dual;
```

#### 3. **在插入语句中使用**
```sql
INSERT INTO employees (id, name) 
VALUES (employee_id_seq.NEXTVAL, 'John Doe');
```

#### 4. **修改序列**
使用 `ALTER SEQUENCE`（不能修改起始值，需删除重建）：
```sql
ALTER SEQUENCE employee_id_seq 
  INCREMENT BY 2
  MAXVALUE 9999;
```

#### 5. **删除序列**
```sql
DROP SEQUENCE employee_id_seq;
```

---

### **关键注意事项**
1. **缓存机制（CACHE）**  
   - 使用缓存（如 `CACHE 20`）可提升性能，但数据库异常关闭可能导致缓存值丢失，导致序列出现间隙。
   - `NOCACHE` 保证无间隙，但性能较低。

2. **并发安全**  
   - 多个会话同时调用 `NEXTVAL` 时，每个会话会获取唯一值，无冲突。

3. **循环（CYCLE）**  
   - 当序列达到极值（如 `MAXVALUE`）后，若启用 `CYCLE`，会重置到 `MINVALUE`。

4. **与表的关联**  
   - 序列独立于表，需手动绑定到字段（如通过触发器或插入语句）。

---

### **实际应用场景**
#### 1. **自增主键**
```sql
-- 创建表
CREATE TABLE employees (
  id NUMBER PRIMARY KEY,
  name VARCHAR2(50)
);

-- 插入数据时使用序列
INSERT INTO employees (id, name) 
VALUES (employee_id_seq.NEXTVAL, 'Jane Smith');
```

#### 2. **替代触发器（Oracle 12c+）**
Oracle 12c 支持 `IDENTITY` 列自动生成序列，简化操作：
```sql
CREATE TABLE employees (
  id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name VARCHAR2(50)
);
```

---

### **总结**
- **适用场景**：主键生成、唯一编号生成（如订单号）。
- **优点**：高效、简单、并发安全。
- **缺点**：可能产生间隙（如事务回滚、缓存丢失时）。

合理配置 `CACHE`、`CYCLE` 等参数，结合业务需求选择是否使用序列。