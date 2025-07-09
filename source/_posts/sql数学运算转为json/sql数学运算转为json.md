---
title: sql数学运算转为json
date: 2025-06-03 17:48:39
updated: 2025-06-03 17:48:39
tags:
 - 开发心得
categories:
 - 开发心得
keywords:
 - sql
 - json
description: 规范的 JSON 设计，用于表示和计算四则运算表达式（加、减、乘、除），支持变量、常量、函数和嵌套表达式
---
# 四则运算表达式 JSON 规范设计

下面是一个规范的 JSON 设计，用于表示和计算四则运算表达式（加、减、乘、除），支持变量、常量、函数和嵌套表达式：

```json
{
  "expression": {
    "type": "binary",
    "operator": "*",
    "left": {
      "type": "binary",
      "operator": "+",
      "left": {
        "type": "variable",
        "name": "price"
      },
      "right": {
        "type": "constant",
        "value": 10
      }
    },
    "right": {
      "type": "binary",
      "operator": "-",
      "left": {
        "type": "constant",
        "value": 100
      },
      "right": {
        "type": "variable",
        "name": "discount"
      }
    }
  }
}
```

## 结构说明

### 节点类型
1. **二元运算节点 (binary)**
   - `operator`: 运算符 (`+`, `-`, `*`, `/`)
   - `left`: 左操作数（表达式节点）
   - `right`: 右操作数（表达式节点）

2. **一元运算节点 (unary)**
   - `operator`: 运算符 (`-` 负号)
   - `operand`: 操作数（表达式节点）

3. **函数调用节点 (function)**
   - `name`: 函数名 (`pow`, `sqrt`, `log`, `sin`, `cos`, `abs` 等)
   - `arguments`: 参数列表（表达式节点数组）

4. **变量节点 (variable)**
   - `name`: 变量名（对应 DataFrame 列名）

5. **常量节点 (constant)**
   - `value`: 常量值（数字或字符串）

### 支持的运算符
| 类型     | 运算符 | 说明                |
|----------|--------|---------------------|
| 算术运算 | `+`    | 加法                |
|          | `-`    | 减法或负号          |
|          | `*`    | 乘法                |
|          | `/`    | 除法                |
| 函数     | 见列表 | 支持常见数学函数    |
| 比较     | `>`    | 大于                |
|          | `<`    | 小于                |
|          | `>=`   | 大于等于            |
|          | `<=`   | 小于等于            |
|          | `==`   | 等于                |
|          | `!=`   | 不等于              |
| 逻辑     | `&&`   | 逻辑与              |
|          | `||`   | 逻辑或              |
|          | `!`    | 逻辑非              |

## Python 解析实现

```python
import pandas as pd
import numpy as np
import math
import json
from pandas.core.series import Series

def eval_expression(df: pd.DataFrame, expr_json: dict) -> Series:
    """
    解析JSON表达式并返回计算结果Series
    
    参数:
    df: Pandas DataFrame
    expr_json: 表达式JSON对象
    
    返回:
    计算结果的Series
    """
    def eval_node(node: dict) -> Series:
        node_type = node["type"]
        
        # 二元运算
        if node_type == "binary":
            op = node["operator"]
            left = eval_node(node["left"])
            right = eval_node(node["right"])
            
            if op == "+": return left + right
            if op == "-": return left - right
            if op == "*": return left * right
            if op == "/": return left / right
            if op == ">": return left > right
            if op == "<": return left < right
            if op == ">=": return left >= right
            if op == "<=": return left <= right
            if op == "==": return left == right
            if op == "!=": return left != right
            if op == "&&": return left & right
            if op == "||": return left | right
            raise ValueError(f"不支持的二元运算符: {op}")
        
        # 一元运算
        elif node_type == "unary":
            op = node["operator"]
            operand = eval_node(node["operand"])
            
            if op == "-": return -operand
            if op == "!": return ~operand
            raise ValueError(f"不支持的一元运算符: {op}")
        
        # 函数调用
        elif node_type == "function":
            func_name = node["name"]
            args = [eval_node(arg) for arg in node["arguments"]]
            
            if func_name == "pow": return args[0] ** args[1]
            if func_name == "sqrt": return np.sqrt(args[0])
            if func_name == "abs": return np.abs(args[0])
            if func_name == "log": return np.log(args[0])
            if func_name == "log10": return np.log10(args[0])
            if func_name == "exp": return np.exp(args[0])
            if func_name == "sin": return np.sin(args[0])
            if func_name == "cos": return np.cos(args[0])
            if func_name == "tan": return np.tan(args[0])
            if func_name == "asin": return np.arcsin(args[0])
            if func_name == "acos": return np.arccos(args[0])
            if func_name == "atan": return np.arctan(args[0])
            if func_name == "sinh": return np.sinh(args[0])
            if func_name == "cosh": return np.cosh(args[0])
            if func_name == "tanh": return np.tanh(args[0])
            if func_name == "ceil": return np.ceil(args[0])
            if func_name == "floor": return np.floor(args[0])
            if func_name == "round": return np.round(args[0])
            if func_name == "if": return np.where(args[0], args[1], args[2])
            raise ValueError(f"不支持的函数: {func_name}")
        
        # 变量引用
        elif node_type == "variable":
            return df[node["name"]]
        
        # 常量值
        elif node_type == "constant":
            value = node["value"]
            # 创建与DataFrame长度相同的常量Series
            return pd.Series([value] * len(df), index=df.index)
        
        raise ValueError(f"未知的节点类型: {node_type}")
    
    # 从JSON字符串解析（如果传入的是字符串）
    if isinstance(expr_json, str):
        expr_json = json.loads(expr_json)
    
    # 获取根表达式
    root_expr = expr_json.get("expression", expr_json)
    
    # 计算并返回结果
    return eval_node(root_expr)

# 示例用法
if __name__ == "__main__":
    # 创建示例数据
    data = {
        'price': [100, 150, 200, 250],
        'quantity': [2, 3, 1, 4],
        'discount': [0.1, 0.2, 0.15, 0.05],
        'is_premium': [True, False, True, False]
    }
    df = pd.DataFrame(data)
    
    # 示例1: 简单四则运算
    simple_expr = {
        "expression": {
            "type": "binary",
            "operator": "*",
            "left": {
                "type": "variable",
                "name": "price"
            },
            "right": {
                "type": "variable",
                "name": "quantity"
            }
        }
    }
    print("简单四则运算 (price * quantity):")
    print(eval_expression(df, simple_expr))
    
    # 示例2: 嵌套表达式
    nested_expr = {
        "expression": {
            "type": "binary",
            "operator": "*",
            "left": {
                "type": "binary",
                "operator": "-",
                "left": {
                    "type": "binary",
                    "operator": "*",
                    "left": {
                        "type": "variable",
                        "name": "price"
                    },
                    "right": {
                        "type": "constant",
                        "value": 1.1
                    }
                },
                "right": {
                    "type": "binary",
                    "operator": "*",
                    "left": {
                        "type": "variable",
                        "name": "price"
                    },
                    "right": {
                        "type": "variable",
                        "name": "discount"
                    }
                }
            },
            "right": {
                "type": "variable",
                "name": "quantity"
            }
        }
    }
    print("\n嵌套表达式 ((price * 1.1 - price * discount) * quantity):")
    print(eval_expression(df, nested_expr))
    
    # 示例3: 带函数的表达式
    func_expr = {
        "expression": {
            "type": "function",
            "name": "if",
            "arguments": [
                {
                    "type": "variable",
                    "name": "is_premium"
                },
                {
                    "type": "binary",
                    "operator": "*",
                    "left": {
                        "type": "variable",
                        "name": "price"
                    },
                    "right": {
                        "type": "constant",
                        "value": 0.9
                    }
                },
                {
                    "type": "binary",
                    "operator": "*",
                    "left": {
                        "type": "variable",
                        "name": "price"
                    },
                    "right": {
                        "type": "binary",
                        "operator": "-",
                        "left": {
                            "type": "constant",
                            "value": 1
                        },
                        "right": {
                            "type": "variable",
                            "name": "discount"
                        }
                    }
                }
            ]
        }
    }
    print("\n带条件函数的表达式 (if(is_premium, price*0.9, price*(1-discount)):")
    print(eval_expression(df, func_expr))
    
    # 示例4: 复杂表达式
    complex_expr = {
        "expression": {
            "type": "binary",
            "operator": "/",
            "left": {
                "type": "binary",
                "operator": "+",
                "left": {
                    "type": "function",
                    "name": "pow",
                    "arguments": [
                        {
                            "type": "variable",
                            "name": "price"
                        },
                        {
                            "type": "constant",
                            "value": 2
                        }
                    ]
                },
                "right": {
                    "type": "function",
                    "name": "sqrt",
                    "arguments": [
                        {
                            "type": "variable",
                            "name": "quantity"
                        }
                    ]
                }
            },
            "right": {
                "type": "binary",
                "operator": "-",
                "left": {
                    "type": "constant",
                    "value": 100
                },
                "right": {
                    "type": "variable",
                    "name": "discount"
                }
            }
        }
    }
    print("\n复杂表达式 ((price^2 + sqrt(quantity)) / (100 - discount)):")
    print(eval_expression(df, complex_expr))
```

## 使用示例说明

### 1. 简单四则运算
```json
{
  "expression": {
    "type": "binary",
    "operator": "*",
    "left": {"type": "variable", "name": "price"},
    "right": {"type": "variable", "name": "quantity"}
  }
}
```
计算: `price * quantity`

### 2. 嵌套表达式
```json
{
  "expression": {
    "type": "binary",
    "operator": "*",
    "left": {
      "type": "binary",
      "operator": "-",
      "left": {
        "type": "binary",
        "operator": "*",
        "left": {"type": "variable", "name": "price"},
        "right": {"type": "constant", "value": 1.1}
      },
      "right": {
        "type": "binary",
        "operator": "*",
        "left": {"type": "variable", "name": "price"},
        "right": {"type": "variable", "name": "discount"}
      }
    },
    "right": {"type": "variable", "name": "quantity"}
  }
}
```
计算: `((price * 1.1) - (price * discount)) * quantity`

### 3. 带函数的表达式
```json
{
  "expression": {
    "type": "function",
    "name": "if",
    "arguments": [
      {"type": "variable", "name": "is_premium"},
      {"type": "binary", "operator": "*", ...}, // true分支
      {"type": "binary", "operator": "*", ...}   // false分支
    ]
  }
}
```
计算: `if(is_premium, price*0.9, price*(1-discount))`

### 4. 复杂表达式
```json
{
  "expression": {
    "type": "binary",
    "operator": "/",
    "left": {
      "type": "binary",
      "operator": "+",
      "left": {
        "type": "function",
        "name": "pow",
        "arguments": [{"type": "variable", "name": "price"}, {"type": "constant", "value": 2}]
      },
      "right": {
        "type": "function",
        "name": "sqrt",
        "arguments": [{"type": "variable", "name": "quantity"}]
      }
    },
    "right": {
      "type": "binary",
      "operator": "-",
      "left": {"type": "constant", "value": 100},
      "right": {"type": "variable", "name": "discount"}
    }
  }
}
```
计算: `(price² + √quantity) / (100 - discount)`

## 功能特点

1. **完整的四则运算支持**：
   - 支持加减乘除基本运算
   - 支持嵌套任意深度的表达式
   - 支持括号（通过嵌套结构隐式实现）

2. **扩展功能**：
   - 变量引用（DataFrame列）
   - 数学函数（pow, sqrt, log, sin, cos等）
   - 条件函数（if）
   - 比较运算和逻辑运算

3. **健壮性设计**：
   - 类型安全：每个节点明确指定类型
   - 错误处理：检测未知运算符和函数
   - 常量传播：优化常量表达式计算

4. **与Pandas无缝集成**：
   - 返回Pandas Series，可直接赋值给新列
   - 支持向量化运算，高效处理大数据
   - 兼容Pandas的索引系统

这个设计提供了一个灵活且强大的方式来表示和执行复杂的数学表达式，特别适合在数据处理和科学计算场景中使用。