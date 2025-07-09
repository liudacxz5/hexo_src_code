---
title: oracle学习笔记-12c后的新特性cdb和pdb
date: 2025-04-12 20:29:34
updated: 2025-04-12 20:29:34
tags:
 - oracle
categories:
 - 技术
 - oracle
keywords:
 - oracle
 - cdb
 - pdb
 - 新特性
description: 描述了oracle12c后新增的cdb和pdb的特性
---
Oracle 12c 引入了 **CDB（Container Database，容器数据库）** 和 **PDB（Pluggable Database，可插拔数据库）** 的新特性，通过多租户架构（Multitenant Environment）实现了更高效的资源管理和数据库部署。以下是其核心概念与特性的详细说明：

---

### **1. CDB 与 PDB 的核心概念**
#### **CDB（容器数据库）**
- **定义**：CDB 是一个物理数据库容器，可以承载多个逻辑上独立的 PDB。所有 PDB 共享 CDB 的硬件资源（如内存、CPU）和后台进程（如 SGA、PGA、Redo Log）。
- **组成**：
  - **CDB$ROOT**：根容器，存储 Oracle 提供的元数据（如 PL/SQL 包源代码）和公共用户（Common User），用于全局管理所有 PDB。
  - **PDB$SEED**：种子容器，作为创建新 PDB 的模板，不可修改。
  - **PDB**：用户创建的可插拔数据库，每个 PDB 独立存储业务数据，对外表现为一个完整的数据库。

#### **PDB（可插拔数据库）**
- **定义**：PDB 是逻辑上独立的数据库，可以像传统数据库一样运行，但物理上依附于 CDB。PDB 支持“热插拔”，即可以在线迁移到其他 CDB 中。
- **特性**：
  - **向后兼容**：PDB 的操作方式与传统数据库完全一致，支持常规的 DDL、DML 操作。
  - **独立性**：每个 PDB 可以单独设置字符集、时区、表空间等参数。

---

### **2. CDB 与 PDB 的架构优势**
1. **资源高效利用**  
   - 多个 PDB 共享 CDB 的 SGA、PGA、Redo Log 等资源，减少硬件开销。
   - 通过资源管理器（Resource Manager）为每个 PDB 分配 CPU 和内存的最低使用份额。

2. **简化运维**  
   - **快速克隆**：通过 `CREATE PLUGGABLE DATABASE ... FROM` 命令从现有 PDB 或种子模板快速创建新 PDB。
   - **无缝迁移**：PDB 可导出为 XML 文件并迁移到其他 CDB，无需停机。

3. **高可用性与隔离性**  
   - PDB 可以独立启动、关闭或备份，故障隔离性更强。
   - 支持在 CDB 级别实现统一的高可用性策略（如 Data Guard）。

---

### **3. 关键操作与命令**
#### **（1）查看与切换容器**
- **查看当前容器**：
  ```sql
  SHOW CON_NAME;  -- 当前容器名称
  SELECT sys_context('userenv', 'con_name') FROM dual; 
  ```
- **切换容器**：
  ```sql
  ALTER SESSION SET CONTAINER = pdb_name;  -- 切换到指定 PDB
  ALTER SESSION SET CONTAINER = CDB$ROOT;  -- 切换回根容器
  ```

#### **（2）管理 PDB**
- **创建 PDB**：
  ```sql
  CREATE PLUGGABLE DATABASE pdb1 
    ADMIN USER admin IDENTIFIED BY password
    FILE_NAME_CONVERT = ('/pdbseed/', '/pdb1/'); 
  ```
- **启动/关闭 PDB**：
  ```sql
  ALTER PLUGGABLE DATABASE pdb1 OPEN;  -- 启动
  ALTER PLUGGABLE DATABASE pdb1 CLOSE; -- 关闭
  ```
- **自动启动 PDB**：
  ```sql
  -- 创建触发器实现 CDB 启动时自动打开所有 PDB
  CREATE OR REPLACE TRIGGER open_pdbs 
  AFTER STARTUP ON DATABASE 
  BEGIN 
    EXECUTE IMMEDIATE 'ALTER PLUGGABLE DATABASE ALL OPEN'; 
  END; 
  ```

---

### **4. 与传统架构的对比**
| **特性**         | **传统数据库（非 CDB）**        | **CDB-PDB 架构**                |
|------------------|--------------------------------|---------------------------------|
| **实例与数据库关系** | 一对一或多对一（RAC）          | 一对多（一个实例管理多个 PDB）  |
| **资源隔离**       | 独立分配资源，冗余开销大       | 共享资源，按需分配              |
| **运维复杂度**     | 每个数据库需单独维护           | 集中管理，批量操作              |
| **迁移灵活性**     | 需导出/导入完整数据库          | 支持 PDB 热插拔迁移             |

---

### **5. 应用场景**
1. **多租户 SaaS 平台**：每个租户对应一个 PDB，实现数据隔离与资源共享。
2. **开发与测试环境**：快速克隆生产环境的 PDB 用于测试，减少数据准备时间。
3. **混合工作负载**：为 OLTP 和 OLAP 分配不同的 PDB，通过资源管理器优化性能。

---

### **6. 注意事项**
- **权限管理**：公共用户（Common User）以 `C##` 开头，可在所有容器中操作；本地用户（Local User）仅限特定 PDB。
- **版本兼容性**：CDB-PDB 特性需 Oracle 12c 及以上版本，且需要企业版许可。

通过 CDB-PDB 架构，Oracle 12c 实现了数据库资源的灵活管理与高效利用，尤其适合需要快速扩展和简化运维的企业场景。更多操作细节可参考 Oracle 官方文档或相关技术博客。