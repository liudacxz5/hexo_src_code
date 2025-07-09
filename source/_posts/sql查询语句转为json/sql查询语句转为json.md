---
title: sql查询语句转为json
date: 2025-06-03 16:48:55
updated: 2025-06-03 16:48:55
tags:
 - 开发心得
categories:
 - 开发心得
keywords:
 - sql
 - json
 - pandas
 - dataframe
description: 一个规范的JSON设计，支持多层嵌套的 WHERE 条件查询，转化为pandas操作
---
## 1. 支持多层嵌套的 WHERE 条件查询的 JSON 设计
下面是一个规范的 JSON 设计，支持多层嵌套的 WHERE 条件查询，符合数据库查询条件的设计模式。该结构支持逻辑运算符（AND/OR/NOT）和比较运算符（=, >, in, like 等），并允许无限级嵌套：

```json
{
  "condition": {
    "operator": "AND",
    "conditions": [
      {
        "field": "age",
        "operator": ">=",
        "value": 18
      },
      {
        "operator": "OR",
        "conditions": [
          {
            "field": "city",
            "operator": "=",
            "value": "New York"
          },
          {
            "field": "city",
            "operator": "=",
            "value": "London"
          }
        ]
      },
      {
        "operator": "NOT",
        "condition": {
          "field": "status",
          "operator": "in",
          "value": ["inactive", "pending"]
        }
      }
    ]
  }
}
```

### 结构说明：
1. **顶级对象**  
   - `condition` (对象)：包含整个查询条件的根节点

2. **逻辑条件节点** (可嵌套)  
   ```typescript
   {
     "operator": "AND" | "OR" | "NOT",  // 逻辑运算符
     "conditions"?: [ ... ],            // 仅当 operator=AND/OR 时存在（子条件数组）
     "condition"?: { ... }              // 仅当 operator=NOT 时存在（单个子条件）
   }
   ```

3. **原子条件节点** (叶子节点)  
   ```typescript
   {
     "field": "字段名",       // 数据库字段名
     "operator": "操作符",    // 比较运算符
     "value": "值"            // 支持类型：string/number/boolean/array
   }
   ```

### 支持的运算符：
| 类型         | 运算符                  | 说明                              | 值类型               |
|--------------|-------------------------|-----------------------------------|----------------------|
| **比较运算符** | `=`, `!=`, `>`, `<`, `>=`, `<=` | 基础比较          | string/number       |
|              | `in`, `not_in`          | 集合判断                          | array               |
|              | `like`, `not_like`      | 模糊匹配（需用 % 通配符）          | string              |
|              | `is_null`, `is_not_null`| 空值检查                          | 忽略 value 字段      |
| **逻辑运算符** | `AND`, `OR`             | 组合多个条件（需配 conditions 数组）| 嵌套对象数组        |
|              | `NOT`                   | 取反（需配 condition 对象）        | 单个嵌套对象        |

### 使用示例：
```json
{
  "condition": {
    "operator": "AND",
    "conditions": [
      {
        "field": "salary",
        "operator": ">",
        "value": 50000
      },
      {
        "operator": "OR",
        "conditions": [
          {
            "field": "department",
            "operator": "=",
            "value": "Engineering"
          },
          {
            "operator": "NOT",
            "condition": {
              "field": "project",
              "operator": "in",
              "value": ["Legacy", "Deprecated"]
            }
          }
        ]
      },
      {
        "field": "name",
        "operator": "like",
        "value": "%John%"
      }
    ]
  }
}
```



## 2. 将上述规范的JSON条件解析为Pandas的DataFrame查询操作
以下是一个Python方法，可以将上述规范的JSON条件解析为Pandas的DataFrame查询操作。该方法支持多层嵌套条件，并处理了所有定义的运算符：

```python
import pandas as pd
import re
import json
from pandas.core.series import Series

def json_to_pandas(df: pd.DataFrame, condition_json: dict) -> pd.DataFrame:
    """
    将JSON条件解析为Pandas DataFrame查询
    
    参数:
    df: 要查询的Pandas DataFrame
    condition_json: 符合规范的JSON条件对象
    
    返回:
    过滤后的DataFrame
    """
    def parse_condition(cond: dict) -> Series:
        # 原子条件（叶子节点）
        if 'field' in cond:
            field = cond['field']
            op = cond['operator']
            value = cond.get('value')
            series = df[field]
            
            # 处理NULL检查
            if op == 'is_null':
                return series.isnull()
            elif op == 'is_not_null':
                return series.notnull()
            
            # 处理比较运算符
            if op == '=':
                return series == value
            elif op == '!=':
                return series != value
            elif op == '>':
                return series > value
            elif op == '>=':
                return series >= value
            elif op == '<':
                return series < value
            elif op == '<=':
                return series <= value
            elif op == 'in':
                return series.isin(value)
            elif op == 'not_in':
                return ~series.isin(value)
            elif op == 'like':
                # 处理不同通配符模式
                if value.startswith('%') and value.endswith('%'):
                    pattern = value[1:-1]
                    return series.str.contains(pattern, regex=False)
                elif value.startswith('%'):
                    pattern = value[1:]
                    return series.str.endswith(pattern)
                elif value.endswith('%'):
                    pattern = value[:-1]
                    return series.str.startswith(pattern)
                else:
                    return series == value
            elif op == 'not_like':
                if value.startswith('%') and value.endswith('%'):
                    pattern = value[1:-1]
                    return ~series.str.contains(pattern, regex=False)
                elif value.startswith('%'):
                    pattern = value[1:]
                    return ~series.str.endswith(pattern)
                elif value.endswith('%'):
                    pattern = value[:-1]
                    return ~series.str.startswith(pattern)
                else:
                    return series != value
        
        # 逻辑条件（分支节点）
        op = cond['operator']
        
        if op == 'AND':
            conditions = [parse_condition(c) for c in cond['conditions']]
            # 逐步合并条件
            result = conditions[0]
            for c in conditions[1:]:
                result = result & c
            return result
        
        elif op == 'OR':
            conditions = [parse_condition(c) for c in cond['conditions']]
            result = conditions[0]
            for c in conditions[1:]:
                result = result | c
            return result
        
        elif op == 'NOT':
            return ~parse_condition(cond['condition'])
        
        else:
            raise ValueError(f"不支持的运算符: {op}")
    
    # 从JSON字符串解析（如果传入的是字符串）
    if isinstance(condition_json, str):
        condition_json = json.loads(condition_json)
    
    # 获取根条件
    root_condition = condition_json.get('condition', condition_json)
    
    # 解析并应用条件
    mask = parse_condition(root_condition)
    return df[mask]

# 示例用法
if __name__ == "__main__":
    # 示例数据
    data = {
        'name': ['John', 'Alice', 'Bob', 'Charlie'],
        'age': [25, 30, 18, 35],
        'city': ['New York', 'London', 'Paris', 'London'],
        'salary': [60000, 75000, 45000, 90000],
        'department': ['Engineering', 'HR', 'Engineering', 'Marketing'],
        'status': ['active', 'inactive', 'pending', 'active']
    }
    df = pd.DataFrame(data)
    
    # 示例JSON条件
    condition = {
        "condition": {
            "operator": "AND",
            "conditions": [
                {
                    "field": "age",
                    "operator": ">=",
                    "value": 18
                },
                {
                    "operator": "OR",
                    "conditions": [
                        {
                            "field": "city",
                            "operator": "=",
                            "value": "London"
                        },
                        {
                            "operator": "NOT",
                            "condition": {
                                "field": "department",
                                "operator": "in",
                                "value": ["HR", "Marketing"]
                            }
                        }
                    ]
                },
                {
                    "field": "name",
                    "operator": "like",
                    "value": "%a%"
                }
            ]
        }
    }
    
    # 应用条件
    result = json_to_pandas(df, condition)
    print("过滤后的数据:")
    print(result)
```

### 功能说明：

1. **支持所有运算符**：
   - 比较运算符：`=`, `!=`, `>`, `<`, `>=`, `<=`
   - 集合运算符：`in`, `not_in`
   - 模糊匹配：`like`, `not_like`
   - 空值检查：`is_null`, `is_not_null`
   - 逻辑运算符：`AND`, `OR`, `NOT`

2. **特殊处理`like`运算符**：
   - `%value%`：包含模式（使用`str.contains`）
   - `%value`：以结尾模式（使用`str.endswith`）
   - `value%`：以开头模式（使用`str.startswith`）
   - 无通配符：精确匹配（使用`==`）

3. **嵌套条件处理**：
   - 递归解析多层嵌套条件
   - 正确处理`AND`/`OR`的组合逻辑
   - 处理`NOT`运算符的单条件反转

4. **输入灵活性**：
   - 可直接传入JSON对象或JSON字符串
   - 自动检测根节点（支持带`condition`键或不带）

### 使用示例：

```python
# 简单条件
simple_cond = {
    "field": "age",
    "operator": ">",
    "value": 30
}

# 嵌套条件
nested_cond = {
    "operator": "AND",
    "conditions": [
        {"field": "salary", "operator": ">=", "value": 50000},
        {"operator": "NOT", "condition": {
            "field": "status", "operator": "in", "value": ["inactive"]
        }}
    ]
}

# 从文件加载JSON条件
with open('conditions.json') as f:
    file_cond = json.load(f)

# 应用条件
result1 = json_to_pandas(df, simple_cond)
result2 = json_to_pandas(df, nested_cond)
result3 = json_to_pandas(df, file_cond)
```

### 注意事项：

1. **性能考虑**：
   - 对于大型数据集，避免深度嵌套的复杂条件
   - `like`操作使用字符串方法比正则表达式更快

2. **错误处理**：
   - 当字段不存在或类型不匹配时会抛出异常
   - 无效运算符会引发`ValueError`

3. **类型安全**：
   - 确保比较操作的数据类型一致（如数值字段不要用`like`）
   - `in`/`not_in`操作的值必须是列表

此实现提供了灵活的条件解析，可以将复杂的嵌套JSON条件转换为高效的Pandas查询操作。